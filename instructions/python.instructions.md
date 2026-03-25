---
name: Python
description: Instructions for general coding practices in Python. Apply these rules anytime when you're writing or reviewing Python code.
applyTo: "**/*.py"
---

# Copilot Python Coding Style

Use this skill whenever generating Python code.

## General Python & Formatting

- Python 3.12+, standard OOP and functional style.
- Follow PEP 8 as the baseline; defer to project-specific overrides when present.
- Four-space indentation, no tabs.
- Line width: 120 characters.
- Use double quotes for strings unless the project convention differs.
- Use f-strings for string interpolation; avoid `.format()` and `%`-formatting.
- Use type hints on all function signatures, class attributes, and return types (PEP 484, PEP 604 union syntax `X | Y`).
- Use `from __future__ import annotations` when targeting compatibility below 3.10.
- Keep functions and methods short and focused; extract private helpers instead of deeply nested logic.
- Use early returns and guard clauses to reduce nesting.
- Use meaningful, intention-revealing names for classes, functions, and variables.
- Prefer composition over inheritance; use protocols (`typing.Protocol`) for structural subtyping.
- Use `dataclasses` or `attrs` for plain data containers; use `Pydantic` for validated/serialized data.
- Prefer immutable data structures: `tuple`, `frozenset`, `NamedTuple`, or frozen dataclasses where applicable.
- Use `pathlib.Path` instead of `os.path` for filesystem operations.
- Use context managers (`with`) for resource management.
- Use list/dict/set comprehensions over `map()`/`filter()` when they improve readability.
- Avoid mutable default arguments; use `None` with a sentinel pattern.
- Use `__all__` to explicitly define public module APIs.

```python
# Bad
def process(items=[]):
    for i in range(len(items)):
        print(items[i])

# Good
def process(items: list[str] | None = None) -> None:
    for item in items or []:
        print(item)
```

## Project Structure

- Use `src/` layout for packages (`src/package_name/`).
- Keep `__init__.py` files minimal; avoid heavy logic in module init.
- Organize by feature or domain, not by layer, for larger projects.
- Place configuration in `pyproject.toml`; avoid `setup.py` and `setup.cfg` for new projects.
- Use a single `pyproject.toml` for tool configuration (Black, Ruff, mypy, pytest).

## Formatting and Linting

- Use **Ruff** as the primary linter and formatter (replaces flake8, isort, and most of Black).
- Run formatting with:
  ```bash
  ruff format .
  ```
- Run linting with:
  ```bash
  ruff check --fix .
  ```
- Use **mypy** in strict mode for type checking:
  ```bash
  mypy --strict .
  ```

## Imports

- Group imports in order: standard library, third-party, local — separated by blank lines.
- Use absolute imports; avoid relative imports except within packages.
- Import specific names (`from module import ClassName`) rather than entire modules when the usage is clear.
- Never use wildcard imports (`from module import *`).
- Sort imports with Ruff's isort-compatible rules.

```python
import os
from collections.abc import Iterator
from pathlib import Path

import httpx
from pydantic import BaseModel

from myapp.services import UserService
```

## Classes and Data Models

- Use `@dataclass` for simple value objects; use `@dataclass(frozen=True)` when immutability is desired.
- Use `Pydantic BaseModel` for data validation, serialization, and API schemas.
- Prefix private methods and attributes with a single underscore `_`.
- Do not use double underscore name mangling unless truly necessary.
- Use `@property` for computed attributes; avoid getter/setter patterns.
- Use `@classmethod` for alternative constructors; use `@staticmethod` sparingly and only when the method needs no class or instance context.
- Use `__slots__` for performance-critical classes with many instances.

```python
from dataclasses import dataclass


@dataclass(frozen=True)
class Coordinate:
    latitude: float
    longitude: float

    @property
    def as_tuple(self) -> tuple[float, float]:
        return (self.latitude, self.longitude)
```

## Error Handling

- Catch specific exceptions; never use bare `except:` or `except Exception:` without re-raising.
- Use custom exception classes derived from domain-specific base exceptions.
- Use `raise ... from err` to preserve exception chains.
- Let unexpected errors propagate; do not silently suppress exceptions.

```python
class ServiceError(Exception):
    pass

class NotFoundError(ServiceError):
    pass

def fetch_user(user_id: int) -> User:
    try:
        return repository.get(user_id)
    except KeyError as err:
        raise NotFoundError(f"User {user_id} not found") from err
```

## Async Python

- Use `async`/`await` with `asyncio` for concurrent I/O-bound work.
- Prefer `httpx.AsyncClient` over `aiohttp` for HTTP requests in async code.
- Use `asyncio.TaskGroup` (3.11+) for structured concurrency.
- Avoid mixing sync and async code in the same call chain; use `asyncio.to_thread()` for blocking operations.

## Dependency Management

- Use `uv` or `pip` with `pyproject.toml` for dependency management.
- Pin direct dependencies in `pyproject.toml`; use lock files (`uv.lock`, `pip-compile`) for reproducible builds.
- Separate dev dependencies from production dependencies.

## Logging

- Use the `logging` module with named loggers (`logging.getLogger(__name__)`).
- Use structured logging with `structlog` when available.
- Prefer lazy formatting with `%s` or f-strings inside `logger.debug()` calls behind level guards.
- Prefix messages with the module or class name for traceability.

```python
import logging

logger = logging.getLogger(__name__)

def process_order(order_id: int) -> None:
    logger.info("OrderService: Processing order [%s]", order_id)
```

## Testing

- Use **pytest** as the test framework.
- Name test files `test_<module>.py`; name test functions `test_<behavior>`.
- Use fixtures for setup and teardown; prefer function-scoped fixtures.
- Use `pytest.raises` for expected exceptions.
- Use `unittest.mock.patch` or `pytest-mock` for mocking; prefer dependency injection over patching.
- Use `factory_boy` or custom fixtures for test data.
- Use `pytest.mark.parametrize` for data-driven tests.

```python
import pytest
from myapp.services import UserService


@pytest.fixture
def user_service() -> UserService:
    return UserService(repository=InMemoryUserRepository())


def test_creates_user(user_service: UserService) -> None:
    user = user_service.create(name="Alice", email="alice@example.com")

    assert user.name == "Alice"
    assert user.email == "alice@example.com"


@pytest.mark.parametrize("email", ["", "invalid", "@missing.local"])
def test_rejects_invalid_email(user_service: UserService, email: str) -> None:
    with pytest.raises(ValueError, match="invalid email"):
        user_service.create(name="Alice", email=email)
```

## Django (when applicable)

- Follow thin-view pattern; move business logic into services, models, or query objects.
- Use Django REST Framework (DRF) serializers for API input/output validation.
- Use `select_related` and `prefetch_related` to avoid N+1 queries.
- Use Django's ORM queryset chaining; avoid raw SQL unless strictly necessary.
- Use Django migrations for all schema changes; never modify the database schema manually.
- Use `django-filter` for queryset filtering in API views.

## FastAPI (when applicable)

- Use Pydantic models for request/response schemas.
- Use dependency injection via `Depends()` for shared logic.
- Use `APIRouter` to organize routes by domain.
- Use async endpoints for I/O-bound operations.
- Use `HTTPException` with specific status codes for error responses.

## When Generating Python Code

- Always include type hints on function signatures and return types.
- Use modern Python syntax (match statements, union types with `|`, walrus operator where it improves clarity).
- Prefer `pathlib` over `os.path`, `httpx` over `requests` for new code.
- Add `if __name__ == "__main__":` guard to scripts.
- Structure packages with `__init__.py` and explicit `__all__`.
