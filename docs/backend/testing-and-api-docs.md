# Backend Testing & API Documentation

## Overview

The RICA backend is thoroughly tested and documented with multiple tools for development, testing, and integration.

## API Documentation

### Swagger UI
Interactive API documentation is available at:
- **Local:** `http://localhost:8000/docs`
- **Production:** Available at the deployed backend URL + `/docs`

Swagger UI provides interactive testing of all API endpoints with request/response schemas.

### ReDoc
Alternative API documentation with a clean, readable format:
- **Local:** `http://localhost:8000/redoc`
- **Production:** Available at the deployed backend URL + `/redoc`

## Postman Collection

A Postman collection is available for testing all API endpoints. The collection includes:

- Authentication flows (login, logout, MFA, password reset)
- User management
- Mineral batch operations
- Processing stages
- Value calculations
- Scenarios
- Decision support
- Environment variables for local and production testing

**Location:** `postman/RICA_API.postman_collection.json` in the backend repository

## Testing with pytest

### Running Tests

```bash
# Run all tests
pytest -q

# Run with verbose output
pytest -v

# Run with coverage report
pytest --cov=. --cov-report=html

# Run specific test file
pytest tests/test_auth.py

# Run tests matching a pattern
pytest -k "auth"
```

### Test Structure

Tests are organized by feature/module:
- `tests/test_auth.py` - Authentication and authorization tests
- `tests/test_users.py` - User management tests
- `tests/test_batches.py` - Mineral batch tests
- `tests/test_values.py` - Value calculation tests
- `tests/test_scenarios.py` - Scenario planning tests
- `tests/test_decision_support.py` - Decision support tests

### Testing Tools

| Tool | Purpose |
|---|---|
| pytest | Primary testing framework |
| pytest-cov | Code coverage reporting |
| httpx | Async HTTP testing for FastAPI |
| Postman | Manual API testing and collection sharing |
| Swagger UI | Interactive API exploration and testing |

## Health Check

The backend provides a health check endpoint for monitoring:
- `GET /health` returns `{"status": "ok"}`
