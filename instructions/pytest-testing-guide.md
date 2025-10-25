# Pytest Testing Guide

Pytest patterns for writing effective Python tests including fixtures, parametrization, mocking, and test organization. These guidelines help AI assistants generate comprehensive, maintainable test suites.

## Core Guidelines

### 1. Use Descriptive Test Function Names

Name test functions with `test_` prefix followed by a clear description of what is being tested and the expected outcome. Use underscores for readability.

**Example**:
```python
# Good
def test_fetch_user_returns_user_object_when_id_exists():
    user = fetch_user(user_id=1)
    assert isinstance(user, User)
    assert user.id == 1

def test_fetch_user_raises_not_found_when_id_missing():
    with pytest.raises(UserNotFoundError):
        fetch_user(user_id=99999)

# Avoid
def test_user():
    assert fetch_user(1)

def test_fetch():
    user = fetch_user(1)
    assert user
```

### 2. Use Fixtures for Reusable Setup

Define fixtures with `@pytest.fixture` decorator for common test data or setup. Use fixtures via function parameters. Leverage fixture scopes (function, class, module, session) appropriately.

**Example**:
```python
# Good
@pytest.fixture
def sample_user():
    return User(id=1, name="John Doe", email="john@example.com")

@pytest.fixture
def db_session():
    session = create_db_session()
    yield session
    session.close()

def test_save_user_persists_to_database(db_session, sample_user):
    save_user(db_session, sample_user)
    retrieved = db_session.query(User).filter_by(id=1).first()
    assert retrieved.name == "John Doe"

# Avoid
def test_save_user_persists_to_database():
    # Repetitive setup in every test
    user = User(id=1, name="John Doe", email="john@example.com")
    session = create_db_session()
    save_user(session, user)
    retrieved = session.query(User).filter_by(id=1).first()
    assert retrieved.name == "John Doe"
    session.close()
```

### 3. Parametrize Tests for Multiple Cases

Use `@pytest.mark.parametrize` to run the same test with different inputs. This reduces code duplication and makes test cases explicit.

**Example**:
```python
# Good
@pytest.mark.parametrize("email,expected", [
    ("user@example.com", True),
    ("invalid.email", False),
    ("", False),
    ("user@domain", False),
])
def test_validate_email(email, expected):
    assert validate_email(email) == expected

# Avoid
def test_validate_email_valid():
    assert validate_email("user@example.com") == True

def test_validate_email_no_at():
    assert validate_email("invalid.email") == False

def test_validate_email_empty():
    assert validate_email("") == False
```

### 4. Use Mocks for External Dependencies

Mock external services, APIs, and databases with `pytest-mock` or `unittest.mock`. Verify mock calls when behavior matters, not just return values.

**Example**:
```python
# Good
def test_fetch_user_calls_api_with_correct_endpoint(mocker):
    mock_get = mocker.patch('httpx.get')
    mock_get.return_value.json.return_value = {"id": 1, "name": "John"}

    fetch_user(user_id=1)

    mock_get.assert_called_once_with("/api/users/1")

def test_send_email_handles_api_failure(mocker):
    mock_send = mocker.patch('email_service.send')
    mock_send.side_effect = APIError("Service unavailable")

    with pytest.raises(EmailSendError):
        send_welcome_email("user@example.com")

# Avoid (hitting real API in tests)
def test_fetch_user():
    user = fetch_user(user_id=1)  # Real HTTP call!
    assert user.name
```

### 5. Organize Tests by Functionality

Structure test files to mirror source code structure. Group related tests in classes. Use clear directory hierarchy for test organization.

**Example**:
```python
# Good
# tests/test_user_service.py
class TestUserCreation:
    def test_create_user_with_valid_data(self):
        user = create_user("John", "john@example.com")
        assert user.name == "John"

    def test_create_user_validates_email(self):
        with pytest.raises(ValidationError):
            create_user("John", "invalid")

class TestUserDeletion:
    def test_delete_user_removes_from_database(self):
        # ...

# Avoid (all tests in one flat file)
def test_user1():
    # ...
def test_user2():
    # ...
def test_order1():
    # ...
```

### 6. Assert with Specific Messages

Use specific assertions (`assert x == y`) rather than `assert x`. Include error messages for complex assertions to aid debugging.

**Example**:
```python
# Good
def test_calculate_total_sums_item_prices():
    items = [Item(price=10), Item(price=20)]
    total = calculate_total(items)
    assert total == 30, f"Expected total 30, got {total}"

def test_user_has_required_fields():
    user = fetch_user(1)
    assert user.email, "User must have email"
    assert user.name, "User must have name"

# Avoid
def test_calculate_total():
    total = calculate_total([Item(price=10), Item(price=20)])
    assert total  # What value should it be?
```

### 7. Use Markers for Test Organization

Tag tests with markers (`@pytest.mark.slow`, `@pytest.mark.integration`) to enable selective test execution. Document markers in pytest.ini.

**Example**:
```python
# Good
@pytest.mark.slow
def test_process_large_dataset():
    # Test that takes >5 seconds
    pass

@pytest.mark.integration
def test_full_user_workflow():
    # Tests multiple components together
    pass

# pytest.ini
# [pytest]
# markers =
#     slow: marks tests as slow (deselect with '-m "not slow"')
#     integration: marks tests as integration tests

# Run only fast tests: pytest -m "not slow"
```

## Quick Reference

- [ ] Test names clearly describe what and expected outcome
- [ ] Fixtures used for reusable setup and teardown
- [ ] Parametrize decorator used for multiple test cases
- [ ] External dependencies (APIs, databases) are mocked
- [ ] Tests organized in files/classes mirroring source structure
- [ ] Assertions are specific with helpful error messages
- [ ] Markers used to categorize tests (slow, integration, etc.)
