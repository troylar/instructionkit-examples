# Security OWASP Checklist

OWASP Top 10 security checklist for developers with specific code patterns to prevent common vulnerabilities. These guidelines help AI assistants generate secure code by following industry-standard security practices.

## Core Guidelines

### 1. Prevent Injection Attacks

Always use parameterized queries, prepared statements, or ORMs. Never concatenate user input into SQL, commands, or code. Validate and sanitize all inputs.

**Example**:
```python
# Good (parameterized query)
def get_user(email: str) -> User:
    query = "SELECT * FROM users WHERE email = ?"
    return db.execute(query, (email,)).fetchone()

# Good (ORM)
def get_user(email: str) -> User:
    return User.query.filter_by(email=email).first()

# Avoid (SQL injection vulnerability!)
def get_user(email: str) -> User:
    query = f"SELECT * FROM users WHERE email = '{email}'"
    return db.execute(query).fetchone()
    # Attacker: "' OR '1'='1'; DROP TABLE users; --"
```

### 2. Implement Proper Authentication

Use strong authentication mechanisms. Never store passwords in plain text. Use bcrypt, Argon2, or similar for password hashing. Implement multi-factor authentication where possible.

**Example**:
```python
# Good (bcrypt hashing)
from passlib.context import CryptContext

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

def create_user(email: str, password: str) -> User:
    hashed = pwd_context.hash(password)
    return User.create(email=email, password_hash=hashed)

def verify_password(plain: str, hashed: str) -> bool:
    return pwd_context.verify(plain, hashed)

# Avoid (plain text - NEVER do this!)
def create_user(email: str, password: str) -> User:
    return User.create(email=email, password=password)
```

### 3. Protect Sensitive Data

Encrypt sensitive data at rest and in transit. Use HTTPS for all communications. Never log passwords, tokens, or PII. Store secrets in environment variables, not code.

**Example**:
```python
# Good (environment variables)
import os
from cryptography.fernet import Fernet

SECRET_KEY = os.environ['SECRET_KEY']
DATABASE_URL = os.environ['DATABASE_URL']
cipher = Fernet(SECRET_KEY.encode())

def encrypt_ssn(ssn: str) -> str:
    return cipher.encrypt(ssn.encode()).decode()

def decrypt_ssn(encrypted: str) -> str:
    return cipher.decrypt(encrypted.encode()).decode()

# Avoid (hardcoded secrets)
SECRET_KEY = "my-secret-key-123"  # Exposed in Git!
DATABASE_URL = "postgresql://user:password@localhost/db"
```

### 4. Implement Access Control

Verify authorization for every protected resource. Check permissions at the code level, not just UI. Use principle of least privilege.

**Example**:
```python
# Good (explicit authorization checks)
def delete_post(post_id: int, current_user: User) -> None:
    post = Post.get(post_id)
    if not post:
        raise NotFoundError("Post not found")

    # Check ownership
    if post.author_id != current_user.id and not current_user.is_admin:
        raise ForbiddenError("Not authorized to delete this post")

    post.delete()

# Avoid (no authorization check)
def delete_post(post_id: int) -> None:
    post = Post.get(post_id)
    post.delete()  # Anyone can delete any post!
```

### 5. Validate and Sanitize All Input

Validate input on server side, never trust client validation. Use allowlists, not denylists. Reject invalid input, don't try to "fix" it.

**Example**:
```python
# Good (strict validation)
from pydantic import BaseModel, EmailStr, validator

class UserCreate(BaseModel):
    email: EmailStr
    age: int
    username: str

    @validator('age')
    def validate_age(cls, v):
        if not 0 <= v <= 150:
            raise ValueError('Age must be 0-150')
        return v

    @validator('username')
    def validate_username(cls, v):
        if not v.isalnum() or len(v) < 3 or len(v) > 20:
            raise ValueError('Username must be 3-20 alphanumeric chars')
        return v

# Avoid (weak or no validation)
def create_user(data: dict) -> User:
    return User.create(**data)  # Accepts anything!
```

### 6. Escape Output to Prevent XSS

Always escape user-generated content before rendering in HTML. Use templating engines with auto-escaping. Set Content-Security-Policy headers.

**Example**:
```python
# Good (Jinja2 auto-escaping)
from jinja2 import Environment, select_autoescape

env = Environment(autoescape=select_autoescape(['html', 'xml']))

def render_comment(comment_text: str) -> str:
    template = env.from_string('<div class="comment">{{ text }}</div>')
    return template.render(text=comment_text)

# Good (explicit escaping)
import html

def render_comment(comment_text: str) -> str:
    safe_text = html.escape(comment_text)
    return f'<div class="comment">{safe_text}</div>'

# Avoid (XSS vulnerability!)
def render_comment(comment_text: str) -> str:
    return f'<div class="comment">{comment_text}</div>'
    # Attacker: "<script>steal_cookies()</script>"
```

### 7. Implement Security Logging and Monitoring

Log security-relevant events (failed logins, authorization failures, input validation failures). Monitor logs for suspicious patterns. Never log sensitive data.

**Example**:
```python
# Good (security logging)
import logging

security_logger = logging.getLogger('security')

def login(email: str, password: str) -> User:
    user = User.get_by_email(email)

    if not user or not verify_password(password, user.password_hash):
        security_logger.warning(
            f"Failed login attempt for email: {email} from IP: {request.remote_addr}"
        )
        raise AuthenticationError("Invalid credentials")

    security_logger.info(f"Successful login for user: {user.id}")
    return user

# Avoid (no security logging)
def login(email: str, password: str) -> User:
    user = User.get_by_email(email)
    if not user or not verify_password(password, user.password_hash):
        raise AuthenticationError("Invalid credentials")
    return user
```

## Quick Reference (OWASP Top 10 Checklist)

- [ ] **A01 - Broken Access Control**: Authorization checks on all protected resources
- [ ] **A02 - Cryptographic Failures**: Sensitive data encrypted, HTTPS enforced
- [ ] **A03 - Injection**: Parameterized queries, input validation
- [ ] **A04 - Insecure Design**: Security requirements defined upfront
- [ ] **A05 - Security Misconfiguration**: Secure defaults, unnecessary features disabled
- [ ] **A06 - Vulnerable Components**: Dependencies updated, vulnerabilities scanned
- [ ] **A07 - Authentication Failures**: Strong password hashing, MFA available
- [ ] **A08 - Data Integrity Failures**: Signature verification, secure deserialization
- [ ] **A09 - Security Logging Failures**: Security events logged, monitoring configured
- [ ] **A10 - Server-Side Request Forgery**: URL validation, allowlist destinations

## Additional Security Practices

- [ ] Use secure session management (httpOnly, secure, sameSite cookies)
- [ ] Implement rate limiting on authentication endpoints
- [ ] Set proper CORS headers
- [ ] Configure Content-Security-Policy headers
- [ ] Use prepared statements for all database queries
- [ ] Validate file uploads (type, size, content)
- [ ] Implement CSRF protection for state-changing operations
