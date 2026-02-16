# AI Agent Workflow Guide

> Comprehensive guide for AI coding agents working on this Python + FastAPI codebase.
> For detailed rules with code examples, see the `docs/` directory.

---

## 0. Navigation Contract

When starting a task, traverse documentation in this order:

1. **This file** (`AGENTS.md`) — project structure, patterns, key rules
2. **`docs/` files** — detailed conventions (only when you need code examples)

---

## 1. Project Overview

<!-- CUSTOMIZE: Replace with your project description -->
Your Project Name — a Python FastAPI microservice.
Describe what the service does, its domain, and key responsibilities.

### Prerequisites

<!-- CUSTOMIZE: Replace with your project's versions -->
- **Python 3.13+**
- **uv** (package manager)
- **Docker + Docker Compose**

### Key Commands

<!-- CUSTOMIZE: Replace with your project's actual commands -->
```bash
make build                    # Build Docker containers
make up                       # Start app container
make infra-up                 # Start infrastructure (PostgreSQL, Redis, RabbitMQ)
make down                     # Stop all services
make logs                     # Tail application logs
make bash                     # Shell into running container
```

### Quality Gates

<!-- CUSTOMIZE: Replace with your project's quality commands -->
```bash
make ruff                     # Lint + format (Ruff)
make mypy                     # Type check (mypy)
make test                     # Run pytest suite
```

### Environment Variables

<!-- CUSTOMIZE: Replace with your project's env pattern -->
Configuration via environment variables. Use `.env` files for local development.

**Never commit `.env` files.** They contain credentials.

### Directory Structure

<!-- CUSTOMIZE: Replace with your project's actual directory structure -->
```
src/
├── api/                      # FastAPI routers and views
│   ├── external/             # External-facing endpoints
│   ├── internal/             # Internal/service-to-service endpoints
│   ├── router.py             # Route registration
│   ├── responses.py          # Response wrappers
│   ├── exception_handlers.py # Global exception handling
│   └── dependencies.py       # FastAPI Depends (DI)
├── db/                       # Database layer
│   ├── models/               # SQLModel entities
│   ├── repositories/         # Data access (generic base + specialized)
│   ├── migrations/           # Alembic migrations
│   ├── base.py               # Engine and session factory
│   └── uow.py                # Unit of Work pattern
├── use_cases/                # Business logic (one class per use case)
├── services/                 # External integrations (cache, messaging)
├── schemas/                  # Pydantic request/response models
├── auth/                     # Authentication (JWT, token backends)
├── config/
│   └── settings.py           # Pydantic BaseSettings (env-based)
├── tasks/                    # Async task queue (Taskiq/Celery)
└── application.py            # FastAPI app factory
tests/
├── conftest.py               # Global fixtures
├── api/                      # API endpoint tests
├── use_cases/                # Use case tests
├── auth/                     # Auth tests
└── schemas/                  # Schema validation tests
```

---

## 2. Architecture

<!-- CUSTOMIZE: Replace with your project's architecture -->
```
HTTP Request
    ↓
[API Layer] — Views + Routers (FastAPI)
    ↓
[Dependency Injection] — Depends(get_uow), Depends(get_current_user)
    ↓
[Use Cases] — Business logic (execute methods)
    ↓
[Unit of Work] — Transaction management (async context manager)
    ↓
[Repositories] — Data access (Generic base + specialized)
    ↓
[Database] — SQLModel / AsyncSession (PostgreSQL)
```

### Key Patterns

| Pattern | Location | Description |
|---------|----------|-------------|
| Unit of Work | `src/db/uow.py` | Wraps transactions; auto-commit/rollback |
| Generic Repository | `src/db/repositories/base.py` | Base CRUD with type parameter |
| Use Cases | `src/use_cases/` | One class per business operation with `execute()` |
| Dependency Injection | `src/api/dependencies.py` | FastAPI `Depends()` for session, UoW, auth |
| Response Wrappers | `src/api/responses.py` | Consistent response format |
| Settings | `src/config/settings.py` | Pydantic BaseSettings, env-based |

---

## 3. Code Conventions (Key Rules)

<!-- CUSTOMIZE: Replace with your project's conventions -->
- **Type hints required** — mypy enforced (`disallow_untyped_defs=true`)
- **Use `|` for unions** — `str | None`, not `Optional[str]`
- **Async everywhere** — `async def`, `AsyncSession`, `async with`
- **Ruff for lint + format** — line length 120, isort enabled
- **Pydantic for validation** — request/response schemas in `src/schemas/`
- **SQLModel for ORM** — models in `src/db/models/`
- **No `Any` types** — use specific types from schema definitions
- **Absolute imports** — `from src.db.models import Location`

**Full rules with code examples:** [docs/code-best-practices.md](docs/code-best-practices.md)

---

## 4. Testing Rules (Key Rules)

<!-- CUSTOMIZE: Replace with your project's test conventions -->
- **pytest + pytest-asyncio** — all async tests with `@pytest.mark.asyncio`
- **Fixtures in conftest.py** — `db_session`, `app`, `async_client`, `uow`
- **Separate test DB** — test schema with fresh tables per session
- **Mock external services** — use `pytest-mock` for task queues, APIs
- **Test file mirrors source** — `tests/api/`, `tests/use_cases/`, etc.
- **No hardcoded data** — use factories or fixtures for test data

**Full rules with code examples:** [docs/test-conventions.md](docs/test-conventions.md)

---

## 5. Database Migrations

<!-- CUSTOMIZE: Replace with your project's migration commands -->
```bash
make alb-revision m="describe change"   # Create Alembic migration (autogenerate)
make alb-upgrade                         # Apply migrations
make alb-downgrade                       # Rollback last migration
```

- Always review autogenerated migrations before committing
- One migration per logical change
- Test migrations apply cleanly on a fresh database

---

## 6. Agent Workflow

### Step 1: Read

- Read this file for project structure and patterns
- Read the relevant `docs/` files for detailed conventions

### Step 2: Find Similar Code

- Search for similar features in `src/` (use cases, routers, models)
- Study existing repositories and use cases for patterns
- Check `src/api/dependencies.py` for available DI providers

### Step 3: Plan

- Identify which files to create or modify
- Check if models, repositories, use cases exist for the domain
- Plan the full stack: schema → model → repository → use case → router

### Step 4: Generate

- Follow existing patterns (copy structure from similar features)
- Use dependency injection via `Depends()`
- Use Unit of Work for database transactions
- Place files in the correct directories per the structure above

### Step 5: Verify

<!-- CUSTOMIZE: Replace with your quality commands -->
```bash
make ruff                     # Lint + format
make mypy                     # Type check
make test                     # Run tests
```

### Step 6: Commit

- Follow [docs/git-conventions.md](docs/git-conventions.md)
- Run quality gates before committing

---

## 7. Git Conventions

<!-- CUSTOMIZE: Replace with your project's commit format -->
- **Commit format:** Descriptive message explaining the change
- **PR template:** Description, implemented changes, ticket link
- **Branch naming:** `feature/`, `fix/`, `chore/` prefixes

**Full details:** [docs/git-conventions.md](docs/git-conventions.md)

---

## Quick Links

| Document | Content |
|---|---|
| [docs/code-best-practices.md](docs/code-best-practices.md) | Type hints, async patterns, DI, repository pattern, error handling |
| [docs/test-conventions.md](docs/test-conventions.md) | Fixtures, async tests, mocking, test structure |
| [docs/git-conventions.md](docs/git-conventions.md) | Commit format, PR template, branch conventions |
