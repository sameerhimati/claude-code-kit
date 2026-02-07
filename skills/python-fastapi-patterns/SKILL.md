---
name: python-fastapi-patterns
description: Python/FastAPI patterns. Auto-loaded in FastAPI projects.
---

# Python / FastAPI Patterns

## Structure
```
src/
├── main.py          # App, middleware, startup
├── routers/         # Route handlers by domain
├── models/          # Pydantic schemas
├── services/        # Business logic
├── db/              # DB models, migrations, queries
├── core/            # Config, dependencies, security
└── utils/           # Shared utilities
tests/
├── conftest.py
├── test_routers/
└── test_services/
```

## Rules
- Pydantic models for ALL request/response bodies
- Dependency injection for shared logic
- async def for route handlers
- httpx.AsyncClient for external calls (not requests)
- Separate request models from response models
- pytest with httpx.AsyncClient + ASGITransport for testing
