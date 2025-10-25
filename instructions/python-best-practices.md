# Python Best Practices

Modern Python coding standards emphasizing type safety, readability, and maintainability. These guidelines help AI assistants generate production-quality Python code that follows PEP 8 and contemporary best practices.

## Core Guidelines

### 1. Always Use Type Hints

Include type hints for all function parameters, return values, and class attributes. Type hints improve code clarity, enable better IDE support, and help AI understand expected data types.

**Example**:
```python
# Good
def fetch_user(user_id: int) -> dict[str, Any]:
    return {"id": user_id, "name": "John"}

# Avoid
def fetch_user(user_id):
    return {"id": user_id, "name": "John"}
```

### 2. Write Descriptive Names

Use clear, specific names that reveal intent. Follow snake_case for functions and variables, PascalCase for classes. Avoid abbreviations unless universally understood.

**Example**:
```python
# Good
def calculate_total_price(items: list[Item]) -> Decimal:
    return sum(item.price for item in items)

# Avoid
def calc_tot(itms):
    return sum(i.p for i in itms)
```

### 3. Include Docstrings

Add Google-style docstrings to all public functions, classes, and modules. Document parameters, return values, and raised exceptions.

**Example**:
```python
# Good
def validate_email(email: str) -> bool:
    """Validate email address format.

    Args:
        email: Email address to validate

    Returns:
        True if valid email format, False otherwise

    Raises:
        ValueError: If email is empty string
    """
    if not email:
        raise ValueError("Email cannot be empty")
    return "@" in email and "." in email

# Avoid
def validate_email(email):
    return "@" in email and "." in email
```

### 4. Handle Errors Explicitly

Use try-except blocks for operations that can fail. Catch specific exceptions, not bare except clauses. Provide meaningful error messages.

**Example**:
```python
# Good
def read_config(path: Path) -> dict:
    try:
        return json.loads(path.read_text())
    except FileNotFoundError:
        logger.error(f"Config file not found: {path}")
        raise
    except json.JSONDecodeError as e:
        logger.error(f"Invalid JSON in config: {e}")
        raise ValueError(f"Config file has invalid JSON: {path}")

# Avoid
def read_config(path):
    try:
        return json.loads(path.read_text())
    except:
        return {}
```

### 5. Follow PEP 8 Formatting

Limit lines to 88 characters (Black formatter default). Use 4 spaces for indentation. Place imports at top of file, grouped by standard library, third-party, and local imports.

**Example**:
```python
# Good
import json
import logging
from pathlib import Path

import httpx
from pydantic import BaseModel

from myapp.models import User

# Avoid
from myapp.models import User
import json, logging, httpx
from pydantic import BaseModel
```

### 6. Use List Comprehensions Appropriately

Prefer list comprehensions for simple transformations. Use generator expressions for large datasets. Fall back to explicit loops for complex logic.

**Example**:
```python
# Good
active_users = [u for u in users if u.is_active]
total = sum(item.price for item in items)

# Avoid (too complex for comprehension)
result = [
    process_item(item) if item.is_valid()
    else handle_invalid(item) if item.has_errors()
    else None
    for item in items
]
```

### 7. Import Only What You Need

Import specific functions or classes rather than entire modules when possible. Avoid wildcard imports (`from module import *`).

**Example**:
```python
# Good
from pathlib import Path
from typing import Optional, List

# Avoid
from pathlib import *
from typing import *
```

## Quick Reference

- [ ] All function parameters and return values have type hints
- [ ] Variable and function names clearly describe their purpose
- [ ] Public functions have Google-style docstrings
- [ ] Error handling uses specific exceptions with meaningful messages
- [ ] Code follows PEP 8 (88 char lines, 4-space indent, proper imports)
- [ ] List comprehensions used only for simple transformations
- [ ] Imports are specific, not wildcard
