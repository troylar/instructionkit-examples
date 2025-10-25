# Security Guidelines

Security coding patterns for preventing common vulnerabilities including input validation, authentication, secrets management, and protection against SQL injection and XSS. These guidelines help AI assistants generate secure code by default.

## Core Guidelines

### 1. Always Validate and Sanitize Input

Validate all user input against expected format, type, and range. Sanitize input before using in queries, commands, or rendering. Never trust client data.

**Example**:
```python
# Good
from pydantic import BaseModel, EmailStr, validator

class UserCreate(BaseModel):
    email: EmailStr
    age: int

    @validator('age')
    def age_must_be_valid(cls, v):
        if v < 0 or v > 150:
            raise ValueError('Age must be between 0 and 150')
        return v

def create_user(data: UserCreate):
    # Input already validated by Pydantic
    return save_user(data)

# Avoid (no validation)
def create_user(email, age):
    # What if age is -5 or "DROP TABLE"?
    return save_user(email, age)
```

### 2. Use Parameterized Queries, Never String Concatenation

Always use parameterized queries or ORM methods for database operations. Never concatenate user input into SQL strings.

**Example**:
```python
# Good (parameterized query)
def get_user_by_email(email: str) -> User:
    query = "SELECT * FROM users WHERE email = $1"
    return db.execute(query, email)

# Good (ORM)
def get_user_by_email(email: str) -> User:
    return db.query(User).filter(User.email == email).first()

# Avoid (SQL injection vulnerability!)
def get_user_by_email(email: str) -> User:
    query = f"SELECT * FROM users WHERE email = '{email}'"
    return db.execute(query)
    # Attacker input: "'; DROP TABLE users; --"
```

### 3. Store Secrets in Environment Variables

Never hardcode API keys, passwords, or secrets in source code. Use environment variables or secret management services. Add secret files to .gitignore.

**Example**:
```python
# Good
import os
from pathlib import Path

API_KEY = os.environ.get('API_KEY')
if not API_KEY:
    raise ValueError("API_KEY environment variable required")

DATABASE_URL = os.environ['DATABASE_URL']

# Good (.env file with python-dotenv)
from dotenv import load_dotenv
load_dotenv()
SECRET_KEY = os.getenv('SECRET_KEY')

# Avoid (hardcoded secrets)
API_KEY = "sk_live_abc123xyz789"  # Exposed in git!
DATABASE_URL = "postgresql://user:password@localhost/db"
```

### 4. Hash Passwords with Modern Algorithms

Use bcrypt, Argon2, or scrypt for password hashing. Never store passwords in plain text or use weak hashing (MD5, SHA1). Include salt automatically.

**Example**:
```python
# Good
from passlib.context import CryptContext

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

def create_user(email: str, password: str):
    hashed_password = pwd_context.hash(password)
    return save_user(email, hashed_password)

def verify_password(plain_password: str, hashed_password: str) -> bool:
    return pwd_context.verify(plain_password, hashed_password)

# Avoid (plain text!)
def create_user(email: str, password: str):
    return save_user(email, password)

# Avoid (weak hashing)
import hashlib
def hash_password(password: str) -> str:
    return hashlib.md5(password.encode()).hexdigest()
```

### 5. Escape Output to Prevent XSS

Escape user-generated content before rendering in HTML. Use templating engines with auto-escaping enabled. Never insert raw user input into HTML.

**Example**:
```python
# Good (Jinja2 with auto-escaping)
from jinja2 import Template, Environment, select_autoescape

env = Environment(autoescape=select_autoescape(['html', 'xml']))
template = env.from_string('<h1>{{ user_name }}</h1>')
html = template.render(user_name=user_input)

# Good (explicit escaping)
import html
safe_comment = html.escape(user_comment)
output = f'<p>{safe_comment}</p>'

# Avoid (XSS vulnerability!)
def render_comment(comment: str) -> str:
    return f'<p>{comment}</p>'
    # Attacker input: "<script>alert('XSS')</script>"
```

### 6. Implement Authentication and Authorization

Authenticate users before allowing access. Verify authorization (permissions) for each protected resource. Use established libraries, don't roll your own.

**Example**:
```python
# Good
from fastapi import Depends, HTTPException, status
from fastapi.security import HTTPBearer

security = HTTPBearer()

async def get_current_user(token: str = Depends(security)) -> User:
    user = verify_token(token.credentials)
    if not user:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Invalid credentials"
        )
    return user

@app.delete("/posts/{post_id}")
async def delete_post(
    post_id: int,
    current_user: User = Depends(get_current_user)
):
    post = get_post(post_id)
    if post.author_id != current_user.id:
        raise HTTPException(status_code=403, detail="Not authorized")
    delete_post(post_id)

# Avoid (no authentication or authorization)
@app.delete("/posts/{post_id}")
async def delete_post(post_id: int):
    delete_post(post_id)  # Anyone can delete any post!
```

### 7. Use HTTPS for All Sensitive Data

Transmit sensitive data (passwords, tokens, personal info) only over HTTPS. Set secure cookie flags. Configure HSTS headers.

**Example**:
```python
# Good (FastAPI with secure settings)
from fastapi.middleware.httpsredirect import HTTPSRedirectMiddleware
from fastapi.middleware.trustedhost import TrustedHostMiddleware

app.add_middleware(HTTPSRedirectMiddleware)
app.add_middleware(
    TrustedHostMiddleware,
    allowed_hosts=["example.com", "*.example.com"]
)

# Good (secure cookie)
response.set_cookie(
    key="session_id",
    value=session_token,
    httponly=True,
    secure=True,  # HTTPS only
    samesite="strict"
)

# Avoid (HTTP allowed for sensitive data)
@app.post("/login")
async def login(username: str, password: str):
    # Password sent over HTTP - can be intercepted!
    user = authenticate(username, password)
    return {"token": user.token}
```

## Quick Reference

- [ ] All user input validated against expected format/type/range
- [ ] Database queries use parameterization, never string concatenation
- [ ] Secrets loaded from environment variables, not hardcoded
- [ ] Passwords hashed with bcrypt/Argon2, never plain text
- [ ] HTML output escaped to prevent XSS attacks
- [ ] Authentication required for protected endpoints
- [ ] Authorization checked before allowing resource access
- [ ] HTTPS enforced for all sensitive data transmission
