# API Design Principles

RESTful API design guidelines covering resource naming, HTTP methods, status codes, and response patterns. These principles help AI assistants generate consistent, intuitive APIs that follow REST conventions.

## Core Guidelines

### 1. Use Nouns for Resource Names, Not Verbs

Resource URLs should represent entities (nouns), not actions (verbs). Use plural nouns for collections. HTTP methods express the action.

**Example**:
```
# Good
GET    /users              # Get all users
POST   /users              # Create user
GET    /users/123          # Get user 123
PUT    /users/123          # Update user 123
DELETE /users/123          # Delete user 123

# Avoid
GET    /getUsers
POST   /createUser
GET    /user/get?id=123
POST   /deleteUser/123
```

### 2. Use Proper HTTP Methods

Use HTTP methods according to their semantic meaning. GET for reading, POST for creating, PUT for full updates, PATCH for partial updates, DELETE for removing.

**Example**:
```python
# Good
@app.get("/users/{user_id}")
async def get_user(user_id: int) -> User:
    return fetch_user(user_id)

@app.post("/users")
async def create_user(user: UserCreate) -> User:
    return save_user(user)

@app.patch("/users/{user_id}")
async def update_user(user_id: int, updates: UserUpdate) -> User:
    return patch_user(user_id, updates)

# Avoid (wrong methods)
@app.get("/users/delete/{user_id}")  # DELETE operation with GET
async def delete_user(user_id: int):
    remove_user(user_id)

@app.post("/users/get")  # GET operation with POST
async def get_user(request: GetUserRequest):
    return fetch_user(request.user_id)
```

### 3. Return Appropriate Status Codes

Use HTTP status codes to indicate outcome. 2xx for success, 4xx for client errors, 5xx for server errors. Be specific within these ranges.

**Example**:
```python
# Good
@app.post("/users", status_code=201)
async def create_user(user: UserCreate) -> User:
    return save_user(user)

@app.delete("/users/{user_id}", status_code=204)
async def delete_user(user_id: int):
    remove_user(user_id)
    return None  # 204 No Content

@app.get("/users/{user_id}")
async def get_user(user_id: int) -> User:
    user = fetch_user(user_id)
    if not user:
        raise HTTPException(status_code=404, detail="User not found")
    return user

# Avoid (always 200)
@app.post("/users")
async def create_user(user: UserCreate):
    save_user(user)
    return {"status": "created"}  # Should be 201, not 200
```

### 4. Use Nested Resources for Relationships

Express resource relationships through URL hierarchy. Keep nesting to 2-3 levels maximum for clarity.

**Example**:
```
# Good
GET    /users/123/posts           # Get posts by user 123
POST   /users/123/posts           # Create post for user 123
GET    /users/123/posts/456       # Get post 456 by user 123
DELETE /posts/456/comments/789    # Delete comment 789 on post 456

# Avoid (too deep or unclear relationships)
GET    /users/123/posts/456/comments/789/likes/012
GET    /user_posts?user_id=123    # Should use nested resource
```

### 5. Implement Pagination for Collections

Always paginate collection endpoints. Support limit/offset or cursor-based pagination. Include pagination metadata in response.

**Example**:
```python
# Good
@app.get("/users")
async def list_users(
    limit: int = 20,
    offset: int = 0
) -> UsersResponse:
    users = fetch_users(limit=limit, offset=offset)
    total = count_users()
    return {
        "items": users,
        "total": total,
        "limit": limit,
        "offset": offset
    }

# Avoid (no pagination - could return millions of records)
@app.get("/users")
async def list_users() -> list[User]:
    return fetch_all_users()  # Could be huge!
```

### 6. Use Query Parameters for Filtering and Sorting

Accept query parameters for filtering, sorting, and searching collections. Document supported parameters clearly.

**Example**:
```python
# Good
@app.get("/users")
async def list_users(
    status: str | None = None,
    role: str | None = None,
    sort_by: str = "created_at",
    order: str = "desc"
) -> list[User]:
    return fetch_users(
        filters={"status": status, "role": role},
        sort_by=sort_by,
        order=order
    )

# Usage:
# GET /users?status=active&role=admin&sort_by=name&order=asc

# Avoid (filtering in request body for GET)
@app.get("/users")
async def list_users(filters: FilterRequest):
    return fetch_users(filters)
```

### 7. Return Consistent Error Responses

Use consistent error response format across all endpoints. Include error code, message, and optional details.

**Example**:
```python
# Good
class ErrorResponse(BaseModel):
    error: str
    message: str
    details: dict | None = None

@app.exception_handler(ValidationError)
async def validation_error_handler(request, exc):
    return JSONResponse(
        status_code=422,
        content={
            "error": "VALIDATION_ERROR",
            "message": "Invalid request data",
            "details": exc.errors()
        }
    )

# Response:
# {
#   "error": "VALIDATION_ERROR",
#   "message": "Invalid request data",
#   "details": {"email": "Invalid email format"}
# }

# Avoid (inconsistent error formats)
# Sometimes: {"error": "bad request"}
# Other times: {"message": "Something went wrong", "code": 400}
```

## Quick Reference

- [ ] Resource names are plural nouns (users, posts), not verbs (getUsers)
- [ ] HTTP methods used correctly (GET/POST/PUT/PATCH/DELETE)
- [ ] Status codes are specific (201 for created, 204 for no content, 404 for not found)
- [ ] Related resources use nested URLs (max 2-3 levels)
- [ ] Collections paginated with limit/offset or cursor
- [ ] Filtering and sorting via query parameters
- [ ] Error responses have consistent format with error code and message
