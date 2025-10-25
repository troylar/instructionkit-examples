# Documentation Standards

Code documentation guidelines including docstrings, inline comments, README structure, and API documentation. These standards help AI assistants generate well-documented code that's maintainable and easy to understand.

## Core Guidelines

### 1. Write Docstrings for All Public Functions

Document all public functions, classes, and modules with docstrings. Include purpose, parameters, return values, and exceptions. Use consistent format (Google, NumPy, or Sphinx style).

**Example**:
```python
# Good (Google-style docstring)
def fetch_user_orders(user_id: int, status: str = "active") -> list[Order]:
    """Fetch all orders for a specific user.

    Retrieves orders from the database filtered by user ID and
    optional status. Results are ordered by creation date descending.

    Args:
        user_id: Unique identifier for the user
        status: Order status filter (default: "active").
                Options: "active", "completed", "cancelled"

    Returns:
        List of Order objects matching the criteria. Empty list
        if no orders found.

    Raises:
        ValueError: If status is not a valid option
        DatabaseError: If database connection fails
    """
    if status not in ["active", "completed", "cancelled"]:
        raise ValueError(f"Invalid status: {status}")

    return Order.query.filter_by(
        user_id=user_id,
        status=status
    ).order_by(Order.created_at.desc()).all()

# Avoid (no documentation)
def fetch_user_orders(user_id, status="active"):
    return Order.query.filter_by(
        user_id=user_id, status=status
    ).order_by(Order.created_at.desc()).all()
```

### 2. Use Inline Comments for Complex Logic

Add comments to explain "why", not "what". Comment complex algorithms, non-obvious decisions, and workarounds. Keep comments up-to-date with code changes.

**Example**:
```python
# Good (explains why)
def calculate_discount(price: float, user: User) -> float:
    # Apply 20% loyalty discount for users with 10+ orders
    # Business requirement from Q2 2024 strategy
    if user.order_count >= 10:
        return price * 0.8

    # New users get 10% first-purchase discount
    # Marketing campaign: expires 2024-12-31
    if user.is_new_customer:
        return price * 0.9

    return price

# Avoid (comments the obvious)
def calculate_discount(price, user):
    # Check if order count is 10 or more
    if user.order_count >= 10:
        # Multiply price by 0.8
        return price * 0.8
    # Return the price
    return price
```

### 3. Structure README Files Clearly

Include purpose, installation, quick start, usage examples, and contribution guidelines. Use clear sections with headings. Provide code examples that users can copy-paste.

**Example**:
```markdown
# Good (comprehensive README)
# Project Name

Brief description of what this project does and why it exists.

## Features

- Feature 1
- Feature 2
- Feature 3

## Installation

\`\`\`bash
pip install project-name
\`\`\`

## Quick Start

\`\`\`python
from project import MainClass

# Simple usage example
client = MainClass(api_key="your-key")
result = client.fetch_data()
print(result)
\`\`\`

## Usage

### Basic Example
[More detailed example]

### Advanced Usage
[Complex scenarios]

## Configuration

| Option | Description | Default |
|--------|-------------|---------|
| api_key | Your API key | None |
| timeout | Request timeout | 30 |

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md)

## License

MIT License - see [LICENSE](LICENSE)

# Avoid (minimal README)
# Project Name

This is a project.

Install: pip install project-name
```

### 4. Document API Endpoints Thoroughly

For APIs, document all endpoints with method, path, parameters, request body, responses, and error codes. Include example requests and responses.

**Example**:
```python
# Good (comprehensive API documentation)
@app.post("/users", status_code=201)
async def create_user(user: UserCreate) -> UserResponse:
    """Create a new user account.

    Creates a user with the provided information and sends
    a verification email to the provided address.

    Request Body:
        {
            "email": "user@example.com",
            "name": "John Doe",
            "password": "secure-password"
        }

    Response (201 Created):
        {
            "id": 123,
            "email": "user@example.com",
            "name": "John Doe",
            "created_at": "2024-01-01T00:00:00Z"
        }

    Error Responses:
        400: Invalid input (missing required fields)
        409: User with this email already exists
        500: Internal server error

    Example:
        curl -X POST http://api.example.com/users \\
             -H "Content-Type: application/json" \\
             -d '{"email": "user@example.com", "name": "John", "password": "pass"}'
    """
    # Implementation...
```

### 5. Keep Documentation Close to Code

Place documentation near the code it describes. Use docstrings for functions, inline comments for implementation details. Avoid separate doc files that drift out of sync.

**Example**:
```python
# Good (documentation with code)
class UserRepository:
    """Repository for user data access operations.

    Provides CRUD operations for User entities with caching
    and automatic retry logic for database failures.
    """

    def __init__(self, db: Database, cache: Cache):
        """Initialize repository with database and cache.

        Args:
            db: Database connection instance
            cache: Cache instance for query results
        """
        self.db = db
        self.cache = cache

    def find_by_email(self, email: str) -> User | None:
        """Find user by email address.

        Checks cache first, then database. Caches result
        for 5 minutes on successful database fetch.

        Args:
            email: User's email address

        Returns:
            User object if found, None otherwise
        """
        # Implementation with inline comments for complex logic
```

### 6. Document Configuration and Environment Variables

List all configuration options and environment variables. Include descriptions, default values, and examples. Specify which are required vs optional.

**Example**:
```python
# Good (documented configuration)
"""
Application Configuration

Required Environment Variables:
    DATABASE_URL: PostgreSQL connection string
        Example: postgresql://user:pass@localhost:5432/dbname

    SECRET_KEY: Secret key for session encryption (min 32 chars)
        Generate: python -c "import secrets; print(secrets.token_hex(32))"

Optional Environment Variables:
    DEBUG: Enable debug mode (default: False)
        Values: true, false

    LOG_LEVEL: Logging verbosity (default: INFO)
        Values: DEBUG, INFO, WARNING, ERROR

    CACHE_TTL: Cache time-to-live in seconds (default: 300)
        Example: 600 for 10 minutes
"""

import os

class Config:
    # Required
    DATABASE_URL: str = os.environ["DATABASE_URL"]
    SECRET_KEY: str = os.environ["SECRET_KEY"]

    # Optional with defaults
    DEBUG: bool = os.getenv("DEBUG", "false").lower() == "true"
    LOG_LEVEL: str = os.getenv("LOG_LEVEL", "INFO")
    CACHE_TTL: int = int(os.getenv("CACHE_TTL", "300"))
```

### 7. Include Examples for Common Use Cases

Provide working code examples for typical scenarios. Examples should be complete, runnable, and cover the most common use cases.

**Example**:
```python
# Good (practical examples in docstring)
class EmailClient:
    """Client for sending emails via SMTP.

    Examples:
        Basic email:
            >>> client = EmailClient("smtp.example.com")
            >>> client.send(
            ...     to="user@example.com",
            ...     subject="Hello",
            ...     body="Email content"
            ... )

        Email with attachments:
            >>> client = EmailClient("smtp.example.com")
            >>> client.send(
            ...     to="user@example.com",
            ...     subject="Report",
            ...     body="See attached",
            ...     attachments=["report.pdf"]
            ... )

        HTML email:
            >>> html = "<h1>Hello</h1><p>Content</p>"
            >>> client.send(
            ...     to="user@example.com",
            ...     subject="Newsletter",
            ...     body_html=html
            ... )
    """
```

## Quick Reference

- [ ] All public functions have docstrings with Args, Returns, Raises
- [ ] Inline comments explain "why" not "what"
- [ ] README includes installation, quick start, usage examples
- [ ] API endpoints documented with request/response examples
- [ ] Documentation stays close to code (not separate files)
- [ ] Configuration variables documented with examples
- [ ] Common use cases have working code examples
- [ ] Documentation updated when code changes
