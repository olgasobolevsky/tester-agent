---
name: api-test-implementor
description: >
  Implement Python pytest API automation tests for REST or GraphQL endpoints.
  Generates runnable test files, fixtures, and pydantic schema validators.
  Use when: implement API tests, write pytest, api automation, REST test,
  GraphQL test, generate test file, httpx, requests, pydantic, conftest, fixture.
---

# API Test Implementor

## Purpose

Write complete, runnable Python `pytest` API test files from a test plan, API spec, or endpoint description. Do not create test plans — use `test-plan-creator` for that first.

## Prerequisites

Before implementing, confirm you have at least one of:
- A test plan from `test-plan-creator`
- An OpenAPI / Swagger spec or explicit endpoint description
- Example request/response payloads

If none are available, ask the user to provide them or run `test-plan-creator` first.

## Conventions

```python
# Use pytest fixtures for client setup and teardown
# Use pydantic models or TypedDict for response schema validation
# Group tests in classes when they share a resource (e.g., a created entity)
# Use parametrize for data-driven tests
# Assert status codes AND response body — never only one
# Name tests: test_should_<behavior>_when_<condition>
```

### Project Structure

| Path | Purpose |
|---|---|
| `tests/api/test_*.py` | Test files, one per resource or feature area |
| `tests/conftest.py` | Shared fixtures (base URL, auth, HTTP client) |
| `tests/schemas/*.py` | Pydantic response models |

### HTTP Client Pattern

```python
# conftest.py
import pytest
import httpx

@pytest.fixture(scope="session")
def base_url() -> str:
    return "https://api.example.com"

@pytest.fixture(scope="session")
def client(base_url: str):
    with httpx.Client(base_url=base_url, timeout=10.0) as c:
        yield c
```

### Schema Validation Pattern

```python
# schemas/user.py
from pydantic import BaseModel

class UserResponse(BaseModel):
    id: int
    email: str
    name: str
```

### Test Structure Pattern

```python
# tests/api/test_users.py
import pytest
from schemas.user import UserResponse

class TestGetUser:
    def test_should_return_user_when_valid_id(self, client):
        response = client.get("/users/1")
        assert response.status_code == 200
        user = UserResponse(**response.json())
        assert user.id == 1

    def test_should_return_404_when_user_not_found(self, client):
        response = client.get("/users/99999")
        assert response.status_code == 404
        assert response.json()["error"] == "User not found"

    @pytest.mark.parametrize("invalid_id", [0, -1, "abc"])
    def test_should_return_400_when_invalid_id(self, client, invalid_id):
        response = client.get(f"/users/{invalid_id}")
        assert response.status_code == 400
```

## Workflow

1. Read the test cases from the test plan (or derive them from the spec)
2. Identify shared fixtures needed (client, auth tokens, test data setup)
3. Create or update `tests/conftest.py` with shared fixtures
4. Create schema models in `tests/schemas/` for each response type
5. Write test file(s) in `tests/api/`, grouped by resource
6. Verify every test case has: status code assertion + body assertion

## Output Format

- Full, runnable Python files with all imports
- One file per resource or feature area
- Fixtures and schemas in separate files
- Brief inline comment per test class explaining what resource/flow is under test
