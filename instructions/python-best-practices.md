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

Use `@dataclass` for data-holding classes.

```python
# Good: Using dataclasses
from dataclasses import dataclass, field
from datetime import datetime

@dataclass
class User:
    """User data model."""
    id: int
    username: str
    email: str
    created_at: datetime = field(default_factory=datetime.now)
    is_active: bool = True
    metadata: dict = field(default_factory=dict)

    def __post_init__(self):
        """Validate user data."""
        if not self.email or "@" not in self.email:
            raise ValueError(f"Invalid email: {self.email}")
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
