# Python Async/Await Patterns

Modern asynchronous programming patterns for Python using async/await syntax. These guidelines help AI assistants generate efficient async code for I/O-bound operations, web frameworks like FastAPI, and concurrent task execution.

## Core Guidelines

### 1. Use async/await for I/O Operations

Mark functions as async when they perform I/O operations (network requests, file access, database queries). Use await for async function calls. Never mix blocking and async code.

**Example**:
```python
# Good
async def fetch_user(user_id: int) -> User:
    async with httpx.AsyncClient() as client:
        response = await client.get(f"/api/users/{user_id}")
        return User(**response.json())

# Avoid (blocking in async context)
async def fetch_user(user_id: int) -> User:
    response = requests.get(f"/api/users/{user_id}")  # Blocks event loop!
    return User(**response.json())
```

### 2. Gather Concurrent Tasks with asyncio.gather

When running multiple independent async operations, use `asyncio.gather()` to execute them concurrently. This dramatically improves performance over sequential await calls.

**Example**:
```python
# Good
async def fetch_user_data(user_id: int) -> UserData:
    user, posts, comments = await asyncio.gather(
        fetch_user(user_id),
        fetch_posts(user_id),
        fetch_comments(user_id)
    )
    return UserData(user=user, posts=posts, comments=comments)

# Avoid (sequential - much slower)
async def fetch_user_data(user_id: int) -> UserData:
    user = await fetch_user(user_id)
    posts = await fetch_posts(user_id)
    comments = await fetch_comments(user_id)
    return UserData(user=user, posts=posts, comments=comments)
```

### 3. Handle Errors in Async Context

Use try-except blocks around await calls. For `asyncio.gather()`, use `return_exceptions=True` to handle individual task failures without canceling all tasks.

**Example**:
```python
# Good
async def fetch_multiple_users(user_ids: list[int]) -> list[User | Exception]:
    tasks = [fetch_user(uid) for uid in user_ids]
    results = await asyncio.gather(*tasks, return_exceptions=True)

    for i, result in enumerate(results):
        if isinstance(result, Exception):
            logger.error(f"Failed to fetch user {user_ids[i]}: {result}")

    return results

# Avoid (one failure cancels all)
async def fetch_multiple_users(user_ids: list[int]) -> list[User]:
    tasks = [fetch_user(uid) for uid in user_ids]
    return await asyncio.gather(*tasks)
```

### 4. Use Async Context Managers

For resources that need cleanup (database connections, HTTP clients, file handles), use async context managers (`async with`). This ensures proper resource cleanup.

**Example**:
```python
# Good
async def save_user(user: User) -> None:
    async with get_db_connection() as conn:
        await conn.execute(
            "INSERT INTO users (name, email) VALUES ($1, $2)",
            user.name, user.email
        )

# Avoid (manual cleanup, error-prone)
async def save_user(user: User) -> None:
    conn = await get_db_connection()
    await conn.execute(
        "INSERT INTO users (name, email) VALUES ($1, $2)",
        user.name, user.email
    )
    await conn.close()
```

### 5. Type Hint Async Functions Correctly

Use `async def` for async functions. Return type hints work the same as sync functions (no need for `Awaitable` or `Coroutine` in most cases).

**Example**:
```python
# Good
async def fetch_user(user_id: int) -> User:
    """Fetch user by ID."""
    async with httpx.AsyncClient() as client:
        response = await client.get(f"/api/users/{user_id}")
        return User(**response.json())

# Avoid (unnecessary type complexity)
from typing import Awaitable

async def fetch_user(user_id: int) -> Awaitable[User]:
    async with httpx.AsyncClient() as client:
        response = await client.get(f"/api/users/{user_id}")
        return User(**response.json())
```

### 6. Use Async Comprehensions for Collections

When processing collections with async operations, use async list/dict comprehensions or `asyncio.gather()`. Don't use sync comprehensions with await inside.

**Example**:
```python
# Good
async def fetch_all_users(user_ids: list[int]) -> list[User]:
    tasks = [fetch_user(uid) for uid in user_ids]
    return await asyncio.gather(*tasks)

# Also Good (async comprehension for small lists)
async def process_items(items: list[Item]) -> list[Result]:
    return [await process_item(item) async for item in items]

# Avoid (runs sequentially, defeats async purpose)
async def fetch_all_users(user_ids: list[int]) -> list[User]:
    return [await fetch_user(uid) for uid in user_ids]
```

## Quick Reference

- [ ] All I/O operations use async/await, not blocking calls
- [ ] Multiple independent async operations use `asyncio.gather()`
- [ ] Error handling uses try-except with `return_exceptions=True` for gather
- [ ] Resources use async context managers (`async with`)
- [ ] Async functions have simple return type hints (no Awaitable wrapper)
- [ ] Collections processed with async comprehensions or gather
- [ ] Never mix blocking (requests, time.sleep) with async code
