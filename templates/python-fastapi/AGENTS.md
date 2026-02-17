# Agents Guide

> Comprehensive guide for AI coding agents working on this codebase.
> For detailed rules with code examples, see the `docs/` directory.

---

## 0. Navigation Contract

Agents MUST traverse context in this order:

1. **This file** (`AGENTS.md`) — workflow, bootstrapping, architecture, key rules
2. **`docs/` files** — detailed conventions (only when you need code examples)
3. **`README.md`** — setup commands, infra details (fallback only)

---

## 1. Project Overview

<!-- CUSTOMIZE: Replace with your project description -->
Your Project Name — a FastAPI microservice serving your domain.
Describe what the service does in 1-2 sentences.

### Prerequisites

<!-- CUSTOMIZE: Replace with your project's versions -->
- **Python 3.13** (pinned via `~=3.13.0` in `pyproject.toml`)
- **uv** (package manager)
- **Docker / Docker Compose**

### Key Commands

<!-- CUSTOMIZE: Replace with your project's actual commands -->
```bash
# Setup (one-time)
make create-dev-network
make infra-up                          # Start PostgreSQL, Redis, RabbitMQ
make build && make up

# Common commands
make test                              # Run pytest (inside Docker)
make ruff                              # Lint + format (Ruff)
make mypy                              # Type check
make logs                              # Tail application logs
make bash                              # Shell into running container

# Database (ALWAYS use Docker)
make alb-revision m="describe change"  # Create Alembic migration
make alb-upgrade                       # Apply migrations
make alb-downgrade                     # Rollback last migration

# Dependencies
make update-lock                       # Update uv.lock inside container
```

### Directory Structure

<!-- CUSTOMIZE: Replace with your project's directory structure -->
```
src/
├── api/                      # FastAPI routers and views
│   ├── external/             # External-facing endpoints
│   ├── internal/             # Internal/service-to-service endpoints
│   ├── router.py             # Route registration
│   ├── responses.py          # JSend response wrappers (Success/Fail/Error)
│   ├── exception_handlers.py # Global exception handling
│   └── dependencies.py       # FastAPI Depends (DI)
├── db/                       # Database layer
│   ├── models/               # SQLModel entities
│   ├── repositories/         # Data access (generic base + specialized)
│   ├── migrations/           # Alembic migrations
│   ├── base.py               # Engine and session factory
│   └── uow.py                # Unit of Work pattern
├── use_cases/                # Business logic (one class per use case)
│   └── exceptions.py         # UseCaseError subclasses
├── services/                 # External integrations (cache, messaging)
├── schemas/                  # Pydantic request/response models
├── auth/                     # Authentication (JWT, token backends)
├── tasks/                    # Async task queue (Taskiq)
├── config/
│   └── settings.py           # Pydantic BaseSettings (env-based)
└── application.py            # FastAPI app factory
tests/
├── conftest.py               # Global fixtures (db_session, app, async_client)
├── api/                      # API endpoint tests
├── use_cases/                # Use case tests (domain-specific conftest.py)
├── auth/                     # Auth tests
└── schemas/                  # Schema validation tests
```

### Architecture

<!-- CUSTOMIZE: Replace with your project's architecture diagram -->
```
View (api/) → Use Case (use_cases/) → Repository (db/repositories/)
     ↓              ↓                          ↓
Schema (schemas/)  Task Queue (tasks/)    SQLModel / AsyncSession
```

**API Layer** (`api/`): Request handling, auth, routing. Separate `routers.py` from `views.py`.
**Use Case Layer** (`use_cases/`): Business logic. Single `execute()` method per class.
**Repository Layer** (`db/repositories/`): Data access via generic base + specialized repos.
**Unit of Work** (`db/uow.py`): Transaction management via async context manager.

---

## 2. Code Conventions (Key Rules)

<!-- CUSTOMIZE: Replace with your project's code rules -->
- **Type hints required** — mypy enforced (`disallow_untyped_defs=true`)
- Use **`|` for unions** (`str | None`, not `Optional[str]`)
- Use **absolute imports** (`from src.db.models import Location`)
- Prefer **early returns** to reduce nesting
- Use **`cast()`** for type narrowing on ORM results
- Log exceptions with `logger.exception(...)` (not `logger.error`)
- Prefer `enum.Enum` over raw strings for settings and states
- **Never edit historical migration files** — create new migrations
- JSend response format: `SuccessResponse`, `FailResponse`, `ErrorResponse`
- Async task dispatch via Taskiq `.kiq()` — never call tasks synchronously

**Full rules with code examples:** [docs/code-best-practices.md](docs/code-best-practices.md)

---

## 3. Testing Rules (Key Rules)

<!-- CUSTOMIZE: Replace with your project's test conventions -->
- **Framework:** pytest + pytest-asyncio (`asyncio_mode = "auto"`)
- **Naming:** `test__<method>__<expected_result>__given_<condition>`
- **Mocking:** Mock where used, not where implemented; `mock_` prefix; `mocker` as first arg
- **Fixtures:** Domain-specific in per-directory `conftest.py`; `autouse` for cache cleanup
- **Tests run in Docker** — `make test`

**Full rules with code examples:** [docs/test-conventions.md](docs/test-conventions.md)

---

## 4. Use Case Conventions (Key Rules)

<!-- CUSTOMIZE: Replace with your project's use case patterns -->
- **No `UseCase` suffix** — `StoreLocation`, not `StoreLocationUseCase`
- **Single `execute()` method** — all other methods are `_private`
- **Explicit parameters** — no `**kwargs` or `**data`
- **Do NOT pass schemas** to use cases — pass model instances or primitives
- **Do NOT call use cases inside other use cases**
- **Unit of Work** for transactions: `async with self.uow:`
- **Raise `UseCaseError` subclasses** for business rule violations
- **Dispatch async tasks** after UoW commit, not inside

**Full rules with code examples:** [docs/use-case-conventions.md](docs/use-case-conventions.md)

---

## 5. Naming Conventions (Key Rules)

<!-- CUSTOMIZE: Replace with your project's naming patterns -->
- **Variables:** snake_case (`driver_guid`, `filter_params`)
- **Booleans:** `is_`, `has_` prefix (`is_location_from_future`)
- **Private methods:** `_` prefix (`_filter_locations`)
- **Constants:** ALL_CAPS (`GPS_TRACKING_EXCHANGE`)
- **Exceptions:** No `Error`/`Exception` suffix — use `DoesNotExist`, `Cannot*`, `Already*`, `Is*`

**Full rules with code examples:** [docs/naming-conventions.md](docs/naming-conventions.md)

---

## 6. Validation Layer Separation

<!-- CUSTOMIZE: Replace with your project's validation patterns -->
| Layer | Validates | Examples |
|-------|-----------|----------|
| **Pydantic Schema** | Request format, types, field rules | `LocationInput`, field validators |
| **View/Dependency** | Permissions, authentication | `Depends(get_current_driver)` |
| **Use Case** | Business logic, domain rules | Time range filtering, entity existence |

**Full rules with code examples:** [docs/validation-conventions.md](docs/validation-conventions.md)

---

## 7. Agent Workflow

1. **Create a branch** — never commit directly to `main`; one branch per logical change
2. **Read this file** — build context from architecture and key rules
3. **Plan changes** — identify which layers to modify (schema → model → repo → use case → router)
4. **Generate code** following Section 2 conventions; use Unit of Work for transactions
5. **Add tests** using pytest with naming: `test__method__result__given_condition`
6. **Run locally** — `make ruff && make mypy && make test`
7. **Review & submit** — self-review all changes (`git diff`), ensure comments are meaningful and necessary, commit following [docs/git-conventions.md](docs/git-conventions.md), then create a pull request

---

## 8. Git Conventions

<!-- CUSTOMIZE: Replace with your project's commit format -->
- **Never commit directly to `main`** — always work on a dedicated branch
- **Format:** `[TICKET-NUMBER] type: description`
- **Never fabricate ticket numbers** — if no Jira ticket exists, omit the prefix entirely
- **Types:** `feature`, `fix`, `misc`, `refactor`
- **Self-review required** — review all changes before creating a pull request
- **PRs required** — all changes merge through pull requests
- **Follow `.github/PULL_REQUEST_TEMPLATE.md`** if the project has one

```
# Good
[GPS-123] feature: Add location filtering by time range
[GPS-456] fix: Handle cache timeout in auth backend

# Bad
fix bug                    # Missing ticket
[GPS-123] updated code     # Vague description
```

**Full details:** [docs/git-conventions.md](docs/git-conventions.md)

---

## Quick Links

| Document | Content |
|---|---|
| [docs/code-best-practices.md](docs/code-best-practices.md) | Async patterns, DI, repositories, UoW, JSend responses, settings |
| [docs/test-conventions.md](docs/test-conventions.md) | pytest patterns, naming, fixtures, mocking, async tests |
| [docs/use-case-conventions.md](docs/use-case-conventions.md) | Single execute(), transactions, task dispatch, exceptions |
| [docs/git-conventions.md](docs/git-conventions.md) | Commit format, types, ticket prefixes |
| [docs/naming-conventions.md](docs/naming-conventions.md) | Variables, booleans, exceptions, abbreviations |
| [docs/validation-conventions.md](docs/validation-conventions.md) | Schema vs dependency vs use case validation |
