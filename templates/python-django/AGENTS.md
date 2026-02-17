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
Your Project Name — a Django-based backend application serving your domain.
Describe what the application does in 1-2 sentences.

### Prerequisites

<!-- CUSTOMIZE: Replace with your project's versions -->
- **Python 3.11**
- **Docker / Docker Compose**
- **Pre-commit hooks** (`pre-commit install`)

### Key Commands

<!-- CUSTOMIZE: Replace with your project's actual commands -->
```bash
# Setup (one-time)
make create-dev-network
docker-compose -f docker-compose.infra.yml up -d
docker-compose build && docker-compose up

# Common commands
make test-all                           # Run all tests
make test target=tests/apps/your_app/   # Run specific tests
make ruff                               # Format and lint
make shell                              # Django shell_plus

# Django commands (ALWAYS use Docker)
docker exec -it your-container python manage.py makemigrations
docker exec -it your-container python manage.py migrate
```

### Directory Structure

<!-- CUSTOMIZE: Replace with your project's directory structure -->
```
your-project/
├── api/                    # API layer (views, serializers)
│   └── internal/
│       ├── web/           # Web app APIs
│       ├── mobile/        # Mobile app APIs
│       └── microservices/ # Service-to-service APIs
├── apps/                   # Domain apps (business logic)
│   ├── common/            # Shared utilities, base exceptions
│   └── ...                # Domain-specific apps
├── config/                 # Django settings, celery, routing
├── pubsub/                 # Central messaging (RabbitMQ)
├── signals/                # Django signals per domain
└── tests/                  # Mirrors source structure
```

### Architecture

<!-- CUSTOMIZE: Replace with your project's architecture diagram -->
```
View (api/) → Use Case (apps/*/use_cases/) → Model (apps/*/models.py)
     ↓              ↓                              ↓
Serializer    Publisher (pubsub/)           Django ORM
```

**View Layer** (`api/`): Request handling, authentication, serialization.
**Use Case Layer** (`apps/*/use_cases/`): Business logic. Single `execute()` method per class.
**Model Layer** (`apps/*/models.py`): Django models. Derived state as properties; mutations in use cases.

---

## 2. Code Conventions (Key Rules)

<!-- CUSTOMIZE: Replace with your project's code rules -->
- Prefer **early returns** to reduce nesting
- Use **absolute imports** (`from apps.foo.x import y`)
- Log exceptions with `logger.exception(...)` (not `logger.error`)
- Prefer `enum.Enum` / `models.TextChoices` over raw strings
- **Never edit historical migration files** — create new migrations
- Pub/Sub publishing via dedicated `publisher.py` helpers, not inline
- GET-by-GUID via use case; list endpoints never raise "not found"
- Validate inputs via Serializers; handle N+1 with `select_related`/`prefetch_related` in use cases
- Celery tasks: specific exception retries with backoff, no blanket `Exception`

**Full rules with code examples:** [docs/code-best-practices.md](docs/code-best-practices.md)

---

## 3. Testing Rules (Key Rules)

<!-- CUSTOMIZE: Replace with your project's test conventions -->
- **Framework:** pytest for all tests
- **Naming:** `test__<method>__<expected_result>__given_<condition>`
- **Structure:** Given-When-Then with section comments
- **Mocking:** Mock where used, not where implemented; `mock_` prefix; `mocker` as first arg
- **Fixtures:** Shared mocks as fixtures; `autouse` for disabled feature toggles
- **Serializers:** Only test complex validation logic, not basic type checks

**Full rules with code examples:** [docs/test-conventions.md](docs/test-conventions.md)

---

## 4. Use Case Conventions (Key Rules)

<!-- CUSTOMIZE: Replace with your project's use case patterns -->
- **No `UseCase` suffix** — `RegisterUser`, not `RegisterUserUseCase`
- **Single `execute()` method** — all other methods are `_private`
- **Explicit parameters** — no `**kwargs` or `**data`
- **Do NOT pass serializers** to use cases — pass model instances or primitives
- **Do NOT call use cases inside other use cases**
- **Atomic transactions** for multiple write operations
- **Raise `UseCaseError` subclasses** for business rule violations
- **Emit signals outside atomic transactions** after commit

**Full rules with code examples:** [docs/use-case-conventions.md](docs/use-case-conventions.md)

---

## 5. Naming Conventions (Key Rules)

<!-- CUSTOMIZE: Replace with your project's naming patterns -->
- **Variables:** snake_case (`invoice_total`, `filter_params`)
- **Booleans:** `is_`, `has_`, `enabled_` prefix
- **Private methods:** `_` prefix
- **Constants:** ALL_CAPS (`MAX_RETRY_COUNT`)
- **Exceptions:** No `Error`/`Exception` suffix — use `DoesNotExist`, `Cannot*`, `Already*`, `Is*`

**Full rules with code examples:** [docs/naming-conventions.md](docs/naming-conventions.md)

---

## 6. Validation Layer Separation

<!-- CUSTOMIZE: Replace with your project's validation patterns -->
| Layer | Validates | Examples |
|-------|-----------|----------|
| **Serializer** | Request format, types, field rules | `validate_email()`, `validate()` |
| **View** | Permissions, access control | `IsDispatcher`, `ObjectBelongsToCarrier` |
| **Use Case** | Business logic, domain rules | State machine, cross-entity uniqueness |

**Full rules with code examples:** [docs/validation-conventions.md](docs/validation-conventions.md)

---

## 7. Agent Workflow

1. **Create a branch** — never commit directly to `main`; one branch per logical change
2. **Read this file** — build context from architecture and key rules
3. **Plan changes** — use feature toggles when risky/new
4. **Generate code** following Section 2 conventions; use dedicated publishers
5. **Add tests** using pytest with naming: `test__method__result__given_condition`
6. **Run locally** — `make test-all` or targeted `make test target=tests/apps/...`
7. **Review & submit** — self-review all changes (`git diff`), ensure comments are meaningful and necessary, commit following [docs/git-conventions.md](docs/git-conventions.md), then create a pull request

---

## 8. Git Conventions

<!-- CUSTOMIZE: Replace with your project's commit format -->
- **Never commit directly to `main`** — always work on a dedicated branch
- **Format:** `[TICKET-NUMBER] type: description`
- **Types:** `feature`, `fix`, `misc`, `refactor`
- **Ticket prefixes:** CAR-, MOBILE-, PLT-, PAYM-, LM-
- **Self-review required** — review all changes before creating a pull request
- **PRs required** — all changes merge through pull requests

```
# Good
[CAR-8532] feature: Redirect not built shipments
[MOBILE-7345] fix: Add default field for arrived_at_location

# Bad
fix bug                    # Missing ticket
[CAR-123] updated code     # Vague description
```

**Full details:** [docs/git-conventions.md](docs/git-conventions.md)

---

## Quick Links

| Document | Content |
|---|---|
| [docs/code-best-practices.md](docs/code-best-practices.md) | Code style, enums, logging, celery, early returns |
| [docs/test-conventions.md](docs/test-conventions.md) | pytest patterns, naming, mocking, fixtures |
| [docs/use-case-conventions.md](docs/use-case-conventions.md) | Single execute(), transactions, signals |
| [docs/git-conventions.md](docs/git-conventions.md) | Commit format, types, ticket prefixes |
| [docs/naming-conventions.md](docs/naming-conventions.md) | Variables, exceptions, abbreviations |
| [docs/validation-conventions.md](docs/validation-conventions.md) | Serializer vs view vs use case validation |
