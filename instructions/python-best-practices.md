# Python Best Practices (3.9+)

## Purpose
Comprehensive Python best practices covering core principles and version-specific features for Python 3.9 through 3.14. Features are tagged by version, and the instruction auto-detects the Python version to apply appropriate features.

## Automatic Version Detection

**CRITICAL: ALWAYS detect the Python version before applying version-specific features.**

**DO NOT use features from newer Python versions without checking version compatibility.**

```python
import sys

# Get Python version - ALWAYS DO THIS FIRST
PYTHON_VERSION = sys.version_info

# Check for specific versions
if PYTHON_VERSION >= (3, 14):
    # Use Python 3.14+ features
    pass
elif PYTHON_VERSION >= (3, 13):
    # Use Python 3.13+ features
    pass
elif PYTHON_VERSION >= (3, 12):
    # Use Python 3.12+ features
    pass
elif PYTHON_VERSION >= (3, 11):
    # Use Python 3.11+ features
    pass
elif PYTHON_VERSION >= (3, 10):
    # Use Python 3.10+ features
    pass
elif PYTHON_VERSION >= (3, 9):
    # Use Python 3.9+ features
    pass
else:
    # Python < 3.9 not supported
    raise RuntimeError(f"Python 3.9+ required, got {PYTHON_VERSION.major}.{PYTHON_VERSION.minor}")

# Display version in error messages
print(f"Python {PYTHON_VERSION.major}.{PYTHON_VERSION.minor}.{PYTHON_VERSION.micro}")
```

**Version Detection Examples:**

```python
# Example 1: Check before using match/case (Python 3.10+)
if sys.version_info >= (3, 10):
    # Use pattern matching
    match value:
        case "create": return create_resource()
        case "delete": return delete_resource()
else:
    # Fallback to if/elif
    if value == "create":
        return create_resource()
    elif value == "delete":
        return delete_resource()

# Example 2: Check before using ExceptionGroup (Python 3.11+)
if sys.version_info >= (3, 11):
    try:
        process_tasks()
    except* ValueError as eg:
        handle_value_errors(eg.exceptions)
else:
    # Fallback: catch exceptions individually
    try:
        process_tasks()
    except ValueError as e:
        handle_value_errors([e])

# Example 3: Check before using type parameters (Python 3.12+)
if sys.version_info >= (3, 12):
    def first[T](items: list[T]) -> T | None:
        return items[0] if items else None
else:
    from typing import TypeVar
    T = TypeVar('T')
    def first(items: list[T]) -> T | None:
        return items[0] if items else None
```

---

# Core Best Practices (All Versions 3.9+)

These practices apply to all Python versions 3.9 and above.

## 1. Always Use Type Hints

Include type hints for all function parameters, return values, and class attributes.

```python
# Good: Clear type hints
from typing import Optional, Any

def fetch_user(user_id: int) -> dict[str, Any]:
    """Fetch user data by ID."""
    return {"id": user_id, "name": "John", "email": "john@example.com"}

def process_data(data: list[str], default: Optional[str] = None) -> str:
    """Process list of strings and return result."""
    if not data:
        return default or ""
    return ", ".join(data)

# Bad: No type hints
def fetch_user(user_id):
    return {"id": user_id, "name": "John"}
```

## 2. Write Descriptive Names

Follow PEP 8 naming conventions:
- `snake_case` for functions, variables, modules
- `PascalCase` for classes
- `UPPER_SNAKE_CASE` for constants
- `_private` for internal use
- `__name_mangled` for class-private attributes

```python
# Good: Descriptive naming
class UserRepository:
    """Repository for user data operations."""

    MAX_RETRY_ATTEMPTS = 3

    def __init__(self, database_url: str):
        self.database_url = database_url
        self._connection_pool = None

    def calculate_total_price(self, items: list[dict]) -> float:
        """Calculate total price from list of items."""
        return sum(item.get("price", 0.0) for item in items)

# Bad: Unclear naming
class usrRep:
    def calc(self, itms):
        return sum(i.get("p", 0) for i in itms)
```

## 3. Include Comprehensive Docstrings

Use Google-style docstrings for all public functions, classes, and modules.

```python
# Good: Comprehensive docstrings
def validate_email(email: str) -> bool:
    """Validate email address format.

    Checks for basic email format requirements including @ symbol
    and domain extension.

    Args:
        email: Email address to validate

    Returns:
        True if email format is valid, False otherwise

    Raises:
        ValueError: If email is None or empty string
        TypeError: If email is not a string

    Example:
        >>> validate_email("user@example.com")
        True
        >>> validate_email("invalid-email")
        False
    """
    if email is None or email == "":
        raise ValueError("Email cannot be None or empty")
    if not isinstance(email, str):
        raise TypeError(f"Email must be string, got {type(email)}")

    return "@" in email and "." in email.split("@")[-1]
```

## 4. Handle Errors Explicitly

Use specific exception handling with meaningful error messages.

```python
# Good: Specific error handling
import logging
from pathlib import Path
import json

logger = logging.getLogger(__name__)

def read_config(config_path: Path) -> dict:
    """Read and parse JSON configuration file."""
    try:
        content = config_path.read_text(encoding="utf-8")
    except FileNotFoundError:
        logger.error(f"Configuration file not found: {config_path}")
        raise
    except PermissionError as e:
        logger.error(f"Permission denied reading config: {config_path}")
        raise ValueError(f"Cannot read config file: {e}") from e

    try:
        return json.loads(content)
    except json.JSONDecodeError as e:
        logger.error(f"Invalid JSON in config {config_path}: {e}")
        raise ValueError(f"Config has invalid JSON at line {e.lineno}") from e

# Bad: Catching all exceptions
def read_config(path):
    try:
        return json.loads(open(path).read())
    except:
        return {}
```

## 5. Use Context Managers for Resources

Always use `with` statements for file operations and resource management.

```python
# Good: Context managers
from pathlib import Path
from contextlib import contextmanager

def process_file(file_path: Path) -> list[str]:
    """Read and process file contents."""
    with file_path.open("r", encoding="utf-8") as f:
        return [line.strip() for line in f if line.strip()]

@contextmanager
def temporary_setting(key: str, value: str):
    """Temporarily change a setting."""
    old_value = settings.get(key)
    settings.set(key, value)
    try:
        yield
    finally:
        settings.set(key, old_value)

# Bad: Manual resource management
def process_file(path):
    f = open(path)
    data = f.read()
    f.close()
    return data
```

## 6. Use Pathlib for File Operations

Use `pathlib.Path` instead of `os.path`.

```python
# Good: Using pathlib
from pathlib import Path

def process_directory(dir_path: Path) -> list[Path]:
    """Process all Python files in directory."""
    if not dir_path.exists():
        raise FileNotFoundError(f"Directory not found: {dir_path}")

    # Find all Python files recursively
    python_files = list(dir_path.glob("**/*.py"))
    return [f for f in python_files if f.stem.startswith("test_")]

# Create paths
config_dir = Path.home() / ".config" / "myapp"
config_file = config_dir / "settings.json"

# Check and create
config_dir.mkdir(parents=True, exist_ok=True)
config_file.write_text('{"debug": true}', encoding="utf-8")
```

## 7. Use Logging Instead of Print

Use the logging module for application messages.

```python
# Good: Using logging
import logging

logger = logging.getLogger(__name__)

def process_batch(items: list[dict]) -> int:
    """Process batch of items."""
    logger.info(f"Starting batch processing of {len(items)} items")

    processed = 0
    for item in items:
        try:
            process_item(item)
            processed += 1
        except ValueError as e:
            logger.warning(f"Skipping invalid item {item.get('id')}: {e}")
        except Exception as e:
            logger.error(f"Error processing item {item.get('id')}: {e}", exc_info=True)

    logger.info(f"Completed: {processed}/{len(items)} successful")
    return processed
```

## 8. Use Dataclasses for Data Structures

Use `@dataclass` for data-holding classes, Pydantic for validation.

```python
# Good: Using dataclasses
from dataclasses import dataclass, field
from datetime import datetime
from typing import ClassVar

@dataclass
class User:
    """User data model."""
    id: int
    username: str
    email: str
    created_at: datetime = field(default_factory=datetime.now)
    is_active: bool = True
    metadata: dict = field(default_factory=dict)

    # Class variable (not instance variable)
    _registry: ClassVar[dict] = {}

    def __post_init__(self):
        """Validate user data."""
        if not self.email or "@" not in self.email:
            raise ValueError(f"Invalid email: {self.email}")
        self._registry[self.id] = self

# Good: Using Pydantic for validation and serialization
from pydantic import BaseModel, EmailStr, Field, field_validator, model_validator

class UserCreate(BaseModel):
    """User creation with validation."""
    username: str = Field(..., min_length=3, max_length=50, pattern=r"^[a-zA-Z0-9_]+$")
    email: EmailStr
    password: str = Field(..., min_length=12)
    age: int | None = Field(None, ge=0, le=150)

    @field_validator("username")
    @classmethod
    def username_alphanumeric(cls, v: str) -> str:
        """Validate username is alphanumeric."""
        if not v.replace("_", "").isalnum():
            raise ValueError("Username must be alphanumeric with underscores")
        return v.lower()

    @field_validator("password")
    @classmethod
    def password_strong(cls, v: str) -> str:
        """Validate password strength."""
        if not any(c.isupper() for c in v):
            raise ValueError("Password must contain uppercase letter")
        if not any(c.isdigit() for c in v):
            raise ValueError("Password must contain digit")
        return v

    @model_validator(mode="after")
    def check_email_username_match(self) -> "UserCreate":
        """Validate email and username consistency."""
        if self.username in self.email:
            raise ValueError("Username should not be in email")
        return self

    model_config = {
        "json_schema_extra": {
            "example": {
                "username": "johndoe",
                "email": "john@example.com",
                "password": "SecurePass123",
                "age": 30
            }
        }
    }
```

## 9. Use Enums for Fixed Sets of Values

Use `Enum` for constants and fixed value sets.

```python
# Good: Using Enums
from enum import Enum, auto

class Status(str, Enum):
    """Order status values."""
    PENDING = "pending"
    PROCESSING = "processing"
    COMPLETED = "completed"
    CANCELLED = "cancelled"

class Priority(int, Enum):
    """Task priority levels."""
    LOW = 1
    MEDIUM = 2
    HIGH = 3
    CRITICAL = 4

def process_order(status: Status) -> str:
    """Process order based on status."""
    match status:
        case Status.PENDING:
            return "Order awaiting processing"
        case Status.PROCESSING:
            return "Order in progress"
        case Status.COMPLETED:
            return "Order complete"
        case Status.CANCELLED:
            return "Order cancelled"

# Usage
current_status = Status.PENDING
if current_status == Status.PENDING:
    start_processing()

# Bad: Using strings
def process_order(status: str) -> str:
    if status == "pending":  # Typo-prone, no autocomplete
        return "Order awaiting processing"
```

## 10. Use Protocol for Structural Subtyping

Use `Protocol` for duck typing with type safety.

```python
# Good: Using Protocol
from typing import Protocol, runtime_checkable

@runtime_checkable
class Drawable(Protocol):
    """Protocol for drawable objects."""

    def draw(self) -> None:
        """Draw the object."""
        ...

    def get_bounds(self) -> tuple[int, int, int, int]:
        """Get bounding box (x, y, width, height)."""
        ...

class Circle:
    """Circle implementation."""

    def draw(self) -> None:
        print("Drawing circle")

    def get_bounds(self) -> tuple[int, int, int, int]:
        return (0, 0, 10, 10)

class Rectangle:
    """Rectangle implementation."""

    def draw(self) -> None:
        print("Drawing rectangle")

    def get_bounds(self) -> tuple[int, int, int, int]:
        return (0, 0, 20, 15)

def render_shape(shape: Drawable) -> None:
    """Render any drawable shape."""
    shape.draw()
    bounds = shape.get_bounds()
    print(f"Bounds: {bounds}")

# Works without inheritance
circle = Circle()
render_shape(circle)  # Type-safe!

# Runtime checking
assert isinstance(circle, Drawable)
```

## 11. Use Type Aliases and NewType

Use type aliases for complex types and NewType for distinct types.

```python
# Good: Type aliases for readability
from typing import TypeAlias, NewType

# Type alias - just an alias, not a distinct type
UserId: TypeAlias = int
UserName: TypeAlias = str
JsonDict: TypeAlias = dict[str, any]
Headers: TypeAlias = dict[str, str]

def get_user(user_id: UserId) -> JsonDict:
    """Get user by ID."""
    return {"id": user_id, "name": "John"}

# NewType - creates a distinct type for type checking
UserId = NewType("UserId", int)
OrderId = NewType("OrderId", int)

def get_user(user_id: UserId) -> dict:
    """Get user by ID - requires UserId, not plain int."""
    return {"id": user_id}

def get_order(order_id: OrderId) -> dict:
    """Get order by ID - requires OrderId, not plain int."""
    return {"id": order_id}

# Usage
user_id = UserId(123)  # Must explicitly create
order_id = OrderId(456)

get_user(user_id)  # OK
get_user(123)  # Type error - must use UserId(123)
get_user(order_id)  # Type error - OrderId is not UserId
```

## 12. Use Comprehensions and Generator Expressions

Use comprehensions for readability, generators for memory efficiency.

```python
# Good: List comprehensions for simple transformations
active_users = [user for user in users if user.is_active]
user_emails = [user.email for user in users if user.email]
squared_numbers = [x**2 for x in range(10)]

# Good: Dict comprehensions
user_map = {user.id: user for user in users}
email_to_name = {user.email: user.name for user in users if user.email}

# Good: Set comprehensions
unique_domains = {email.split("@")[1] for email in emails}

# Good: Generator expressions for large datasets
total = sum(item.price for item in large_item_list)  # Memory efficient
first_match = next((user for user in users if user.id == target_id), None)

# Generator function for lazy evaluation
def read_large_file(file_path: Path) -> Iterator[str]:
    """Read file line by line - memory efficient."""
    with file_path.open("r") as f:
        for line in f:
            if line.strip():
                yield line.strip()

# Bad: Loading everything into memory
def read_large_file(file_path: Path) -> list[str]:
    with file_path.open("r") as f:
        return [line.strip() for line in f if line.strip()]
```

## 13. Use Context Managers Extensively

Create custom context managers for resource management.

```python
# Good: Custom context managers
from contextlib import contextmanager, asynccontextmanager
import asyncio

@contextmanager
def database_transaction(db_connection):
    """Context manager for database transactions."""
    transaction = db_connection.begin()
    try:
        yield transaction
        transaction.commit()
    except Exception:
        transaction.rollback()
        raise

@contextmanager
def timer(name: str):
    """Context manager to time code blocks."""
    import time
    start = time.time()
    try:
        yield
    finally:
        elapsed = time.time() - start
        print(f"{name} took {elapsed:.2f}s")

# Usage
with timer("Data processing"):
    process_large_dataset()

# Good: Async context managers
@asynccontextmanager
async def async_database_connection(connection_string: str):
    """Async context manager for database connections."""
    conn = await create_connection(connection_string)
    try:
        yield conn
    finally:
        await conn.close()

# Usage
async def fetch_user(user_id: int):
    async with async_database_connection("postgresql://...") as conn:
        return await conn.fetch_one("SELECT * FROM users WHERE id = $1", user_id)
```

## 14. Use Proper Exception Handling

Create custom exceptions and handle errors appropriately.

```python
# Good: Custom exception hierarchy
class ApplicationError(Exception):
    """Base exception for application errors."""
    pass

class ValidationError(ApplicationError):
    """Validation failed."""
    pass

class DatabaseError(ApplicationError):
    """Database operation failed."""
    pass

class NotFoundError(ApplicationError):
    """Resource not found."""

    def __init__(self, resource_type: str, resource_id: any):
        self.resource_type = resource_type
        self.resource_id = resource_id
        super().__init__(f"{resource_type} with id {resource_id} not found")

# Good: Specific error handling with context
def get_user(user_id: int) -> User:
    """Get user by ID."""
    try:
        user_data = database.query("SELECT * FROM users WHERE id = ?", user_id)
    except DatabaseConnectionError as e:
        logger.error(f"Database connection failed: {e}")
        raise DatabaseError("Failed to connect to database") from e
    except QueryError as e:
        logger.error(f"Query failed for user {user_id}: {e}")
        raise DatabaseError(f"Failed to query user {user_id}") from e

    if not user_data:
        raise NotFoundError("User", user_id)

    try:
        return User(**user_data)
    except (TypeError, ValueError) as e:
        logger.error(f"Failed to parse user data: {e}")
        raise ValidationError(f"Invalid user data for id {user_id}") from e

# Usage
try:
    user = get_user(123)
except NotFoundError as e:
    return {"error": str(e)}, 404
except DatabaseError as e:
    return {"error": "Database unavailable"}, 503
except ValidationError as e:
    return {"error": str(e)}, 500
```

## 15. Use Modern Async/Await Patterns

Write proper async code with structured concurrency.

```python
# Good: Async/await with proper error handling
import asyncio
from typing import Any

async def fetch_user_data(user_id: int) -> dict[str, Any]:
    """Fetch user data asynchronously."""
    async with httpx.AsyncClient() as client:
        try:
            response = await client.get(f"/api/users/{user_id}", timeout=5.0)
            response.raise_for_status()
            return response.json()
        except httpx.TimeoutException:
            logger.error(f"Timeout fetching user {user_id}")
            raise
        except httpx.HTTPStatusError as e:
            logger.error(f"HTTP error {e.response.status_code} for user {user_id}")
            raise

# Good: Concurrent execution with gather
async def fetch_multiple_users(user_ids: list[int]) -> list[dict[str, Any]]:
    """Fetch multiple users concurrently."""
    tasks = [fetch_user_data(user_id) for user_id in user_ids]
    results = await asyncio.gather(*tasks, return_exceptions=True)

    # Handle partial failures
    successful_results = []
    for user_id, result in zip(user_ids, results):
        if isinstance(result, Exception):
            logger.error(f"Failed to fetch user {user_id}: {result}")
        else:
            successful_results.append(result)

    return successful_results

# Good: TaskGroup for structured concurrency (Python 3.11+)
async def process_with_taskgroup(items: list[str]) -> list[Any]:
    """Process items with TaskGroup - fails fast on error."""
    async with asyncio.TaskGroup() as group:
        tasks = [group.create_task(process_item(item)) for item in items]

    return [task.result() for task in tasks]

# Good: Async generators
async def fetch_paginated_data(endpoint: str) -> AsyncIterator[dict]:
    """Fetch paginated data lazily."""
    page = 1
    async with httpx.AsyncClient() as client:
        while True:
            response = await client.get(f"{endpoint}?page={page}")
            data = response.json()

            if not data["items"]:
                break

            for item in data["items"]:
                yield item

            page += 1

# Usage
async for user in fetch_paginated_data("/api/users"):
    await process_user(user)
```

---

# Python 3.9+ Features

**Available in:** Python 3.9, 3.10, 3.11, 3.12, 3.13, 3.14

## Dictionary Merge Operators `|` and `|=`

```python
# Python 3.9+: Dictionary merge
defaults = {"host": "localhost", "port": 8080, "debug": False}
user_config = {"port": 9000, "debug": True}

# Merge operator |
config = defaults | user_config
# Result: {"host": "localhost", "port": 9000, "debug": True}

# Update operator |=
settings = {"timeout": 30}
settings |= {"retry": 3, "timeout": 60}
# Result: {"timeout": 60, "retry": 3}

# Combining multiple configs
final_config = base_config | env_config | cli_args
```

## Built-in Generic Type Hints

```python
# Python 3.9+: Use built-in types directly
def process_items(items: list[str]) -> dict[str, int]:
    """Count frequency of items."""
    return {item: items.count(item) for item in set(items)}

def group_data(
    data: list[dict[str, any]],
    key: str
) -> dict[str, list[dict[str, any]]]:
    """Group dictionaries by a key."""
    result: dict[str, list[dict[str, any]]] = {}
    for item in data:
        group_key = item.get(key)
        if group_key:
            result.setdefault(group_key, []).append(item)
    return result

# All available: list[], dict[], set[], tuple[], frozenset[]
```

## String removeprefix() and removesuffix()

```python
# Python 3.9+: String prefix/suffix removal
filename = "test_user_model.py"

# Remove prefix
module_name = filename.removeprefix("test_")
# Result: "user_model.py"

# Remove suffix
name_without_ext = filename.removesuffix(".py")
# Result: "test_user_model"

# Chain operations
clean_name = filename.removeprefix("test_").removesuffix(".py")
# Result: "user_model"

# URL handling
def clean_url(url: str) -> str:
    """Remove protocol from URL."""
    url = url.removeprefix("https://")
    url = url.removeprefix("http://")
    return url.removesuffix("/")
```

## zoneinfo Module for Timezones

```python
# Python 3.9+: Standard library timezone support
from zoneinfo import ZoneInfo
from datetime import datetime

# Create timezone-aware datetime
now_utc = datetime.now(ZoneInfo("UTC"))
now_ny = datetime.now(ZoneInfo("America/New_York"))

# Convert between timezones
utc_time = datetime(2024, 1, 1, 12, 0, tzinfo=ZoneInfo("UTC"))
ny_time = utc_time.astimezone(ZoneInfo("America/New_York"))

# Schedule across timezones
def schedule_meeting(
    start_utc: datetime,
    attendee_timezones: list[str]
) -> dict[str, datetime]:
    """Show meeting time in each attendee's timezone."""
    return {
        tz: start_utc.astimezone(ZoneInfo(tz))
        for tz in attendee_timezones
    }
```

---

# Python 3.10+ Features

**Available in:** Python 3.10, 3.11, 3.12, 3.13, 3.14

## Structural Pattern Matching (match/case)

```python
# Python 3.10+: Pattern matching
def process_command(command: dict) -> str:
    """Process command using pattern matching."""
    match command:
        case {"action": "create", "resource": resource, "data": data}:
            return f"Creating {resource} with {data}"

        case {"action": "delete", "resource": resource, "id": item_id}:
            return f"Deleting {resource} with id {item_id}"

        case {"action": "update", "resource": resource, "id": item_id, "data": data}:
            return f"Updating {resource} {item_id} with {data}"

        case {"action": "list", "resource": resource}:
            return f"Listing all {resource}"

        case _:
            return "Unknown command"

# Pattern matching with types
def handle_response(response: int | dict | list) -> str:
    """Handle different response types."""
    match response:
        case int(x) if x >= 200 and x < 300:
            return f"Success: {x}"
        case int(x) if x >= 400:
            return f"Error: {x}"
        case {"error": message}:
            return f"Error: {message}"
        case {"data": items} if isinstance(items, list):
            return f"Got {len(items)} items"
        case _:
            return "Unknown response"

# Pattern matching with classes
from dataclasses import dataclass

@dataclass
class Point:
    x: int
    y: int

def describe_point(point: Point) -> str:
    """Describe point location."""
    match point:
        case Point(x=0, y=0):
            return "Origin"
        case Point(x=0, y=y):
            return f"On Y-axis at {y}"
        case Point(x=x, y=0):
            return f"On X-axis at {x}"
        case Point(x=x, y=y) if x == y:
            return f"On diagonal at {x}"
        case Point(x=x, y=y):
            return f"At ({x}, {y})"
```

## Union Type Operator `|`

```python
# Python 3.10+: Union types with |
def process_id(user_id: int | str) -> str:
    """Process user ID of either type."""
    return str(user_id)

def fetch_data(source: str | Path) -> bytes:
    """Fetch data from string URL or file path."""
    if isinstance(source, str):
        return httpx.get(source).content
    return source.read_bytes()

# Optional is now T | None
def get_user(user_id: int) -> dict[str, any] | None:
    """Get user or None if not found."""
    return database.get(user_id)

# Multiple types
def format_value(value: int | float | str | None) -> str:
    """Format value to string."""
    if value is None:
        return "N/A"
    return str(value)
```

## Better Error Messages

Python 3.10 provides more precise error messages with context:

```python
# Python 3.10+ shows exact location of errors
# Before: SyntaxError: invalid syntax
# After: SyntaxError: '(' was never closed

data = {
    "name": "John",
    "age": 30,
    # Missing closing brace - Python 3.10 tells you exactly where
```

## TypeAlias Annotation

```python
# Python 3.10+: Explicit type aliases
from typing import TypeAlias

UserId: TypeAlias = int
UserName: TypeAlias = str
UserData: TypeAlias = dict[str, any]

def create_user(user_id: UserId, name: UserName) -> UserData:
    """Create user with type aliases."""
    return {"id": user_id, "name": name}
```

---

# Python 3.11+ Features

**Available in:** Python 3.11, 3.12, 3.13, 3.14

## Exception Groups and except*

```python
# Python 3.11+: Handle multiple exceptions
def process_multiple_tasks(tasks: list[callable]) -> list[any]:
    """Process tasks and collect all errors."""
    results = []
    exceptions = []

    for task in tasks:
        try:
            results.append(task())
        except Exception as e:
            exceptions.append(e)

    if exceptions:
        raise ExceptionGroup("Task processing failed", exceptions)

    return results

# Catch exception groups
try:
    process_multiple_tasks(tasks)
except* ValueError as eg:
    # Handle all ValueErrors
    for exc in eg.exceptions:
        logger.error(f"Value error: {exc}")
except* TypeError as eg:
    # Handle all TypeErrors
    for exc in eg.exceptions:
        logger.error(f"Type error: {exc}")
```

## tomllib for TOML Files

```python
# Python 3.11+: Built-in TOML support
import tomllib
from pathlib import Path

def load_pyproject_config(project_dir: Path) -> dict:
    """Load pyproject.toml configuration."""
    pyproject_path = project_dir / "pyproject.toml"

    with pyproject_path.open("rb") as f:
        config = tomllib.load(f)

    return config

# Access configuration
config = load_pyproject_config(Path.cwd())
project_name = config["tool"]["poetry"]["name"]
dependencies = config["tool"]["poetry"]["dependencies"]
```

## Task and TaskGroup for asyncio

```python
# Python 3.11+: Structured concurrency
import asyncio

async def fetch_data(url: str) -> dict:
    """Fetch data from URL."""
    # Simulated fetch
    await asyncio.sleep(1)
    return {"url": url, "data": "..."}

async def process_multiple_urls(urls: list[str]) -> list[dict]:
    """Process multiple URLs concurrently."""
    async with asyncio.TaskGroup() as group:
        tasks = [group.create_task(fetch_data(url)) for url in urls]

    # All tasks completed or exception raised
    return [task.result() for task in tasks]

# If any task fails, all tasks are cancelled
try:
    results = await process_multiple_urls(urls)
except* Exception as eg:
    # Handle exception group from failed tasks
    pass
```

## Self Type for Returning Self

```python
# Python 3.11+: Self type annotation
from typing import Self

class Builder:
    """Fluent builder pattern."""

    def __init__(self):
        self.data = {}

    def set_name(self, name: str) -> Self:
        """Set name and return self."""
        self.data["name"] = name
        return self

    def set_age(self, age: int) -> Self:
        """Set age and return self."""
        self.data["age"] = age
        return self

    def build(self) -> dict:
        """Build final object."""
        return self.data.copy()

# Usage with chaining
user = Builder().set_name("John").set_age(30).build()
```

## Performance Improvements

Python 3.11 is 10-60% faster than 3.10 due to:
- Faster startup
- Faster runtime
- Zero-cost exceptions
- Inline caching

---

# Python 3.12+ Features

**Available in:** Python 3.12, 3.13, 3.14

## f-string Improvements

```python
# Python 3.12+: Enhanced f-strings

# Reuse quotes
name = "World"
greeting = f"Hello {name!r}"  # Works with nested quotes

# Multi-line expressions
result = f"""
    Processing data:
    {sum(
        item.value
        for item in items
        if item.is_valid
    )}
"""

# Complex expressions
data = {"values": [1, 2, 3, 4, 5]}
summary = f"Total: {sum(data['values'])}, Average: {sum(data['values']) / len(data['values']):.2f}"

# Debugging with =
x = 10
y = 20
print(f"{x + y = }")  # Prints: x + y = 30
```

## Type Parameter Syntax (PEP 695)

```python
# Python 3.12+: Generic function syntax
def first[T](items: list[T]) -> T | None:
    """Get first item from list."""
    return items[0] if items else None

def identity[T](value: T) -> T:
    """Return the same value."""
    return value

# Generic class syntax
class Stack[T]:
    """Generic stack implementation."""

    def __init__(self):
        self._items: list[T] = []

    def push(self, item: T) -> None:
        """Push item onto stack."""
        self._items.append(item)

    def pop(self) -> T | None:
        """Pop item from stack."""
        return self._items.pop() if self._items else None

# Generic type alias
type Point[T] = tuple[T, T]
type Matrix[T] = list[list[T]]

def distance[T: int | float](p1: Point[T], p2: Point[T]) -> float:
    """Calculate distance between two points."""
    return ((p2[0] - p1[0])**2 + (p2[1] - p1[1])**2)**0.5
```

## Improved Error Messages

```python
# Python 3.12+ provides even better error messages with suggestions

# Before:
# NameError: name 'usr_name' is not defined

# After:
# NameError: name 'usr_name' is not defined. Did you mean: 'user_name'?

# Attribute errors with suggestions
user.usrname  # AttributeError: 'User' object has no attribute 'usrname'. Did you mean: 'username'?
```

## Per-Interpreter GIL (Experimental)

```python
# Python 3.12+: Subinterpreters (experimental)
# True parallelism for CPU-bound tasks
import _xxsubinterpreters as subinterpreters

# Create subinterpreter
interp_id = subinterpreters.create()

# Run code in subinterpreter
code = """
def calculate(n):
    return sum(i*i for i in range(n))
result = calculate(1000000)
"""

subinterpreters.run_string(interp_id, code)
```

---

# Python 3.13+ Features

**Available in:** Python 3.13, 3.14

## Experimental JIT Compiler

```python
# Python 3.13+: Experimental Just-In-Time compiler
# Enable with: python -X jit or PYTHON_JIT=1

# JIT provides performance improvements for:
# - Numeric computations
# - Loops
# - Function calls

# No code changes needed - automatic optimization
def calculate_primes(limit: int) -> list[int]:
    """Calculate prime numbers up to limit."""
    primes = []
    for num in range(2, limit):
        is_prime = True
        for i in range(2, int(num ** 0.5) + 1):
            if num % i == 0:
                is_prime = False
                break
        if is_prime:
            primes.append(num)
    return primes

# This function benefits from JIT compilation automatically
```

## Improved Error Messages

```python
# Python 3.13+: Even more helpful error messages

# Better traceback formatting
# Clearer indication of error location
# More context in exception messages

# Type hint improvements
# Better error messages for type mismatches
```

## Enhanced Type System

```python
# Python 3.13+: TypedDict improvements
from typing import TypedDict, NotRequired, Required

class UserData(TypedDict):
    """User data with required and optional fields."""
    id: Required[int]  # Explicitly required
    username: Required[str]
    email: NotRequired[str]  # Explicitly optional
    age: NotRequired[int]

# ReadOnly type qualifier
from typing import ReadOnly

class Config(TypedDict):
    """Configuration with read-only fields."""
    api_key: ReadOnly[str]  # Cannot be modified
    timeout: int  # Can be modified
```

## Improved asyncio Performance

```python
# Python 3.13+: Faster asyncio operations
import asyncio

async def fetch_with_timeout(url: str, timeout: float) -> str:
    """Fetch URL with timeout - faster in 3.13."""
    async with asyncio.timeout(timeout):  # Faster implementation
        # Fetch operation
        await asyncio.sleep(0.1)
        return f"Data from {url}"

# TaskGroup is more efficient
async def process_batch_fast(items: list[str]) -> list[str]:
    """Process items with improved TaskGroup performance."""
    async with asyncio.TaskGroup() as group:
        tasks = [
            group.create_task(fetch_with_timeout(item, 5.0))
            for item in items
        ]
    return [task.result() for task in tasks]
```

---

# Python 3.14+ Features

**Available in:** Python 3.14

**Note:** Python 3.14 is in development. Features listed are tentative.

## Tentative Features

```python
# Python 3.14+: Tentative features (subject to change)

# Potential improvements to pattern matching
# Enhanced performance optimizations
# Further JIT compiler enhancements
# Additional typing system improvements

# Check official Python 3.14 release notes when available
```

---

# Advanced Best Practices

## Testing with Pytest

Write comprehensive tests with fixtures and parametrization.

```python
# Good: Comprehensive pytest tests
import pytest
from pathlib import Path
from unittest.mock import Mock, patch, AsyncMock

# Fixtures for reusable test setup
@pytest.fixture
def user_data():
    """Sample user data for tests."""
    return {
        "id": 1,
        "username": "testuser",
        "email": "test@example.com"
    }

@pytest.fixture
def database_session():
    """Database session for tests."""
    session = create_test_session()
    yield session
    session.rollback()
    session.close()

# Parametrized tests
@pytest.mark.parametrize("email,expected", [
    ("valid@example.com", True),
    ("invalid-email", False),
    ("@example.com", False),
    ("user@", False),
])
def test_email_validation(email: str, expected: bool):
    """Test email validation with multiple inputs."""
    assert validate_email(email) == expected

# Testing exceptions
def test_user_creation_with_invalid_email():
    """Test that invalid email raises ValueError."""
    with pytest.raises(ValueError, match="Invalid email"):
        create_user(username="test", email="invalid")

# Async tests
@pytest.mark.asyncio
async def test_async_fetch_user():
    """Test async user fetching."""
    user = await fetch_user_async(user_id=1)
    assert user["id"] == 1
    assert "username" in user

# Mocking
@patch("myapp.services.external_api_call")
def test_service_with_mocked_api(mock_api):
    """Test service with mocked external API."""
    mock_api.return_value = {"status": "success"}

    result = process_with_api_call(data="test")

    assert result["status"] == "success"
    mock_api.assert_called_once_with(data="test")

# Async mocking
@pytest.mark.asyncio
@patch("myapp.services.async_api_call", new_callable=AsyncMock)
async def test_async_service(mock_async_api):
    """Test async service with mocked API."""
    mock_async_api.return_value = {"data": "test"}

    result = await fetch_data_async()

    assert result["data"] == "test"
    mock_async_api.assert_awaited_once()

# Fixture with parameterization
@pytest.fixture(params=[
    ("postgresql", "localhost", 5432),
    ("mysql", "localhost", 3306),
])
def database_config(request):
    """Database configuration variants."""
    db_type, host, port = request.param
    return {"type": db_type, "host": host, "port": port}

def test_database_connection(database_config):
    """Test connection with different databases."""
    conn = connect_database(**database_config)
    assert conn.is_connected()
```

## Dependency Injection

Use dependency injection for testable, maintainable code.

```python
# Good: Dependency injection with protocols
from typing import Protocol
from abc import abstractmethod

class UserRepository(Protocol):
    """Protocol for user repository."""

    @abstractmethod
    def get_by_id(self, user_id: int) -> User | None:
        """Get user by ID."""
        ...

    @abstractmethod
    def save(self, user: User) -> None:
        """Save user."""
        ...

class DatabaseUserRepository:
    """Database implementation of user repository."""

    def __init__(self, db_connection):
        self.db = db_connection

    def get_by_id(self, user_id: int) -> User | None:
        """Get user by ID from database."""
        data = self.db.query("SELECT * FROM users WHERE id = ?", user_id)
        return User(**data) if data else None

    def save(self, user: User) -> None:
        """Save user to database."""
        self.db.execute(
            "INSERT INTO users (id, username, email) VALUES (?, ?, ?)",
            user.id, user.username, user.email
        )

class UserService:
    """User service with injected dependencies."""

    def __init__(self, repository: UserRepository):
        """Initialize with repository dependency."""
        self.repository = repository

    def get_user(self, user_id: int) -> User | None:
        """Get user by ID."""
        return self.repository.get_by_id(user_id)

    def create_user(self, username: str, email: str) -> User:
        """Create new user."""
        user = User(id=generate_id(), username=username, email=email)
        self.repository.save(user)
        return user

# Usage in production
db_connection = create_database_connection()
repository = DatabaseUserRepository(db_connection)
service = UserService(repository)

# Usage in tests with mock
mock_repository = Mock(spec=UserRepository)
mock_repository.get_by_id.return_value = User(id=1, username="test", email="test@example.com")
test_service = UserService(mock_repository)
```

## Caching and Memoization

Use caching for expensive operations.

```python
# Good: Using functools.lru_cache
from functools import lru_cache, cache
import time

@lru_cache(maxsize=128)
def expensive_computation(n: int) -> int:
    """Expensive computation with caching."""
    time.sleep(1)  # Simulate expensive operation
    return n ** 2

# Python 3.9+: @cache is equivalent to @lru_cache(maxsize=None)
@cache
def fibonacci(n: int) -> int:
    """Fibonacci with unlimited cache."""
    if n < 2:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)

# Custom caching with TTL
from datetime import datetime, timedelta
from typing import Any

class TTLCache:
    """Time-to-live cache implementation."""

    def __init__(self, ttl_seconds: int):
        self.ttl = timedelta(seconds=ttl_seconds)
        self._cache: dict[Any, tuple[Any, datetime]] = {}

    def get(self, key: Any) -> Any | None:
        """Get value from cache if not expired."""
        if key in self._cache:
            value, timestamp = self._cache[key]
            if datetime.now() - timestamp < self.ttl:
                return value
            del self._cache[key]
        return None

    def set(self, key: Any, value: Any) -> None:
        """Set value in cache with timestamp."""
        self._cache[key] = (value, datetime.now())

# Decorator with TTL cache
def ttl_cache(ttl_seconds: int):
    """Decorator for TTL caching."""
    cache = TTLCache(ttl_seconds)

    def decorator(func):
        def wrapper(*args, **kwargs):
            key = (args, tuple(sorted(kwargs.items())))
            result = cache.get(key)
            if result is None:
                result = func(*args, **kwargs)
                cache.set(key, result)
            return result
        return wrapper
    return decorator

@ttl_cache(ttl_seconds=300)  # 5 minute cache
def fetch_user_data(user_id: int) -> dict:
    """Fetch user data with 5-minute cache."""
    return api.get_user(user_id)
```

## Performance Optimization

Write performant Python code.

```python
# Good: Performance best practices

# 1. Use slots for classes with fixed attributes
class Point:
    """Memory-efficient point class."""
    __slots__ = ('x', 'y')

    def __init__(self, x: float, y: float):
        self.x = x
        self.y = y

# 2. Use generators for large datasets
def process_large_file(file_path: Path) -> Iterator[dict]:
    """Process large file line by line."""
    with file_path.open() as f:
        for line in f:
            yield json.loads(line)

# 3. Use itertools for efficient iteration
from itertools import islice, chain, groupby

# Take first 10 items efficiently
first_10 = list(islice(large_iterator, 10))

# Chain multiple iterators
combined = chain(iter1, iter2, iter3)

# Group by key
data = [("a", 1), ("a", 2), ("b", 3), ("b", 4)]
for key, group in groupby(data, key=lambda x: x[0]):
    print(f"{key}: {list(group)}")

# 4. Use set operations for membership testing
# Good: O(1) lookup
allowed_users = {"user1", "user2", "user3"}
if username in allowed_users:
    grant_access()

# Bad: O(n) lookup
allowed_users = ["user1", "user2", "user3"]
if username in allowed_users:
    grant_access()

# 5. Use dict.get() with default instead of key checking
# Good
value = data.get("key", default_value)

# Bad
if "key" in data:
    value = data["key"]
else:
    value = default_value

# 6. Use list/dict comprehensions over loops
# Good: Fast
squares = [x**2 for x in range(1000)]

# Slower
squares = []
for x in range(1000):
    squares.append(x**2)

# 7. Use local variables in loops
# Good: Lookup local variable (faster)
def process_items(items: list[str]) -> list[str]:
    upper = str.upper  # Cache method lookup
    return [upper(item) for item in items]

# Slower: Global/attribute lookup each iteration
def process_items(items: list[str]) -> list[str]:
    return [item.upper() for item in items]
```

## Security Best Practices

Write secure Python code.

```python
# Good: Security best practices

# 1. Never store passwords in plain text
from passlib.hash import bcrypt
import secrets

def hash_password(password: str) -> str:
    """Hash password securely."""
    return bcrypt.hash(password)

def verify_password(password: str, hashed: str) -> bool:
    """Verify password against hash."""
    return bcrypt.verify(password, hashed)

# 2. Use secrets module for tokens and keys
def generate_api_key() -> str:
    """Generate secure API key."""
    return secrets.token_urlsafe(32)

def generate_session_token() -> str:
    """Generate secure session token."""
    return secrets.token_hex(32)

# 3. Validate and sanitize user input
from html import escape
import re

def sanitize_html_input(user_input: str) -> str:
    """Sanitize HTML input to prevent XSS."""
    return escape(user_input)

def validate_username(username: str) -> str:
    """Validate username format."""
    if not re.match(r"^[a-zA-Z0-9_]{3,20}$", username):
        raise ValueError("Invalid username format")
    return username

# 4. Use parameterized queries for SQL
# Good: Parameterized query prevents SQL injection
def get_user_by_email(email: str) -> dict | None:
    """Get user by email securely."""
    query = "SELECT * FROM users WHERE email = ?"
    return database.fetch_one(query, email)

# Bad: String formatting allows SQL injection
def get_user_by_email(email: str) -> dict | None:
    query = f"SELECT * FROM users WHERE email = '{email}'"
    return database.fetch_one(query)

# 5. Handle secrets securely
import os
from pathlib import Path

def load_api_key() -> str:
    """Load API key from environment or file."""
    # Try environment variable first
    api_key = os.getenv("API_KEY")
    if api_key:
        return api_key

    # Fall back to secure file
    key_file = Path.home() / ".secrets" / "api_key"
    if key_file.exists():
        return key_file.read_text().strip()

    raise ValueError("API key not found")

# 6. Use constant-time comparison for secrets
import hmac

def verify_api_key(provided_key: str, stored_key: str) -> bool:
    """Verify API key using constant-time comparison."""
    return hmac.compare_digest(provided_key, stored_key)

# 7. Set secure cookie attributes
from http.cookies import SimpleCookie

def create_secure_cookie(name: str, value: str) -> str:
    """Create secure cookie."""
    cookie = SimpleCookie()
    cookie[name] = value
    cookie[name]["httponly"] = True
    cookie[name]["secure"] = True  # HTTPS only
    cookie[name]["samesite"] = "Strict"
    cookie[name]["max-age"] = 3600  # 1 hour
    return cookie[name].OutputString()

# 8. Rate limiting
from datetime import datetime, timedelta
from collections import defaultdict

class RateLimiter:
    """Simple rate limiter implementation."""

    def __init__(self, max_requests: int, window_seconds: int):
        self.max_requests = max_requests
        self.window = timedelta(seconds=window_seconds)
        self.requests: defaultdict[str, list[datetime]] = defaultdict(list)

    def is_allowed(self, client_id: str) -> bool:
        """Check if request is allowed for client."""
        now = datetime.now()
        cutoff = now - self.window

        # Remove old requests
        self.requests[client_id] = [
            ts for ts in self.requests[client_id]
            if ts > cutoff
        ]

        # Check if under limit
        if len(self.requests[client_id]) < self.max_requests:
            self.requests[client_id].append(now)
            return True

        return False

# Usage
limiter = RateLimiter(max_requests=100, window_seconds=60)
if not limiter.is_allowed(client_ip):
    raise HTTPException(status_code=429, detail="Rate limit exceeded")
```

## Modern Project Structure

Organize projects following best practices.

```
# Good: Modern Python project structure
myproject/
├── src/
│   └── myproject/
│       ├── __init__.py
│       ├── main.py
│       ├── config.py
│       ├── models/
│       │   ├── __init__.py
│       │   └── user.py
│       ├── services/
│       │   ├── __init__.py
│       │   └── user_service.py
│       ├── repositories/
│       │   ├── __init__.py
│       │   └── user_repository.py
│       └── api/
│           ├── __init__.py
│           └── routes.py
├── tests/
│   ├── __init__.py
│   ├── conftest.py
│   ├── unit/
│   │   └── test_user_service.py
│   └── integration/
│       └── test_api.py
├── docs/
│   ├── index.md
│   └── api.md
├── pyproject.toml
├── README.md
├── .gitignore
├── .env.example
└── requirements.txt  # or poetry.lock
```

**pyproject.toml (Modern Python packaging):**

```toml
[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[project]
name = "myproject"
version = "0.1.0"
description = "My awesome project"
readme = "README.md"
requires-python = ">=3.10"
license = {text = "MIT"}
authors = [
    {name = "Your Name", email = "you@example.com"}
]
keywords = ["example", "project"]
classifiers = [
    "Development Status :: 4 - Beta",
    "Programming Language :: Python :: 3.10",
    "Programming Language :: Python :: 3.11",
    "Programming Language :: Python :: 3.12",
]
dependencies = [
    "pydantic>=2.0",
    "httpx>=0.25",
]

[project.optional-dependencies]
dev = [
    "pytest>=7.0",
    "pytest-asyncio>=0.21",
    "pytest-cov>=4.0",
    "black>=23.0",
    "ruff>=0.1",
    "mypy>=1.0",
]

[project.urls]
Homepage = "https://github.com/user/myproject"
Documentation = "https://myproject.readthedocs.io"
Repository = "https://github.com/user/myproject"

[project.scripts]
myproject = "myproject.main:main"

[tool.pytest.ini_options]
testpaths = ["tests"]
python_files = ["test_*.py"]
python_functions = ["test_*"]
addopts = "--cov=src --cov-report=html --cov-report=term"

[tool.black]
line-length = 100
target-version = ["py310", "py311", "py312"]

[tool.ruff]
line-length = 100
target-version = "py310"
select = ["E", "F", "I", "N", "W", "UP"]

[tool.mypy]
python_version = "3.10"
warn_return_any = true
warn_unused_configs = true
disallow_untyped_defs = true
```

---

# Version-Specific Decision Tree

Use this decision tree to determine which features to use:

```python
import sys

PYTHON_VERSION = sys.version_info

# Python 3.9+: Always available
use_dict_merge = True  # d1 | d2
use_builtin_generics = True  # list[str], dict[str, int]
use_removeprefix = True  # str.removeprefix()
use_zoneinfo = True  # from zoneinfo import ZoneInfo

# Python 3.10+
use_match_case = PYTHON_VERSION >= (3, 10)  # match/case
use_union_operator = PYTHON_VERSION >= (3, 10)  # int | str
use_type_alias = PYTHON_VERSION >= (3, 10)  # TypeAlias

# Python 3.11+
use_exception_groups = PYTHON_VERSION >= (3, 11)  # ExceptionGroup, except*
use_tomllib = PYTHON_VERSION >= (3, 11)  # import tomllib
use_self_type = PYTHON_VERSION >= (3, 11)  # from typing import Self
use_task_group = PYTHON_VERSION >= (3, 11)  # asyncio.TaskGroup

# Python 3.12+
use_type_parameter_syntax = PYTHON_VERSION >= (3, 12)  # def func[T]()
use_enhanced_fstrings = PYTHON_VERSION >= (3, 12)  # f"{x = }"

# Python 3.13+
use_jit = PYTHON_VERSION >= (3, 13)  # Experimental JIT
use_readonly = PYTHON_VERSION >= (3, 13)  # ReadOnly type

# Python 3.14+
# Check for new features when released
```

---

# Quick Reference Checklist

## Core Practices (All Versions)
- [ ] Type hints on all functions and methods
- [ ] Descriptive names following PEP 8
- [ ] Google-style docstrings for public APIs
- [ ] Specific exception handling with context
- [ ] Context managers for resources
- [ ] Pathlib for file operations
- [ ] Logging instead of print
- [ ] Dataclasses for data structures

## Python 3.9+ Features
- [ ] Dictionary merge with `|` and `|=`
- [ ] Built-in generic types `list[T]`, `dict[K, V]`
- [ ] String `removeprefix()` and `removesuffix()`
- [ ] zoneinfo for timezone handling

## Python 3.10+ Features
- [ ] Pattern matching with `match/case`
- [ ] Union types with `|` (e.g., `int | str`)
- [ ] TypeAlias for explicit type aliases

## Python 3.11+ Features
- [ ] Exception groups with `except*`
- [ ] tomllib for TOML files
- [ ] Self type for method chaining
- [ ] asyncio.TaskGroup for structured concurrency

## Python 3.12+ Features
- [ ] Enhanced f-strings with complex expressions
- [ ] Type parameter syntax `def func[T]()`
- [ ] Generic class syntax `class Stack[T]`

## Python 3.13+ Features
- [ ] JIT compiler (experimental, enable with -X jit)
- [ ] ReadOnly type qualifier
- [ ] Enhanced asyncio performance

## Python 3.14+ Features
- [ ] Check official release notes when available

---

# When to Apply

Apply these practices to:
- All new Python projects (3.9+)
- Library and application code
- Scripts and automation tools
- API services and web applications
- Data processing pipelines
- CLI applications
- Async/await applications
- Type-checked codebases

## Version Selection Guide

- **Python 3.9**: Minimum recommended version (stable, widely deployed)
- **Python 3.10**: Pattern matching, better type hints
- **Python 3.11**: 10-60% faster, exception groups
- **Python 3.12**: New type parameter syntax, better error messages
- **Python 3.13**: Experimental JIT, performance improvements
- **Python 3.14**: Latest features (when released)
