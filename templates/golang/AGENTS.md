# AI Agent Workflow Guide

> Comprehensive guide for AI coding agents working on this Go codebase.
> For detailed rules with code examples, see the `docs/` directory.

---

## 0. Navigation Contract

When starting a task, traverse documentation in this order:

1. **This file** (`AGENTS.md`) — project structure, patterns, key rules
2. **`docs/` files** — detailed conventions (only when you need code examples)

---

## 1. Project Overview

<!-- CUSTOMIZE: Replace with your project description -->
Your Project Name — a Go microservice.
Describe what the service does, its domain, and key responsibilities.

### Prerequisites

<!-- CUSTOMIZE: Replace with your project's versions -->
- **Go 1.22+**
- **Docker + Docker Compose**

### Key Commands

<!-- CUSTOMIZE: Replace with your project's actual commands -->
```bash
make build                    # Build Docker containers
make run                      # Start containers (docker-compose up -d)
make stop                     # Stop containers (docker-compose down)
make restart                  # Stop + run
make test                     # Run all unit tests (go test ./... -v)
make integration-test         # Run integration tests with testcontainers
make logs                     # Tail application logs
```

### Environment Variables

<!-- CUSTOMIZE: Replace with your project's config -->
Configuration via environment variables (Viper `AutomaticEnv`):

| Variable | Default | Description |
|----------|---------|-------------|
| `ENVIRONMENT` | `development` | Environment name |
| `HTTP_PORT` | `:8080` | HTTP listen address |
| `LOG_LEVEL` | `debug` | debug, info, warn, error |

**Never commit secrets.** Use `.env` files for local development only.

### Directory Structure

<!-- CUSTOMIZE: Replace with your project's actual directory structure -->
```
cmd/
└── your-service/
    └── main.go               # Entry point: config, deps, server, shutdown
config/
└── config.go                 # Viper-based config from env vars
internal/                     # Private packages (not importable externally)
├── cache/                    # In-memory or Redis cache
├── domain/                   # Domain types and interfaces
├── service/                  # Business logic
├── repository/               # Data access layer
├── transport/
│   └── http/
│       ├── router.go         # HTTP router setup
│       ├── middleware/        # Request logging, auth, etc.
│       ├── healthcheck/      # GET /livez, /readyz
│       └── handlers/         # HTTP handlers by domain
pkg/                          # Public reusable packages
├── http/                     # JSON response writer, error types
└── logger/                   # Structured logger interface
tests/
└── integration/              # Integration tests (testcontainers)
deployments/
└── Dockerfile                # Multi-stage Alpine build
```

---

## 2. Architecture

<!-- CUSTOMIZE: Replace with your project's architecture -->
```
HTTP Request
    ↓
[Transport Layer] — Chi/Gin router + middleware
    ↓
[Handlers] — Parse request, call service, write response
    ↓
[Service Layer] — Business logic
    ↓
[Repository] — Data access (DB, cache, external APIs)
```

### Key Patterns

| Pattern | Location | Description |
|---------|----------|-------------|
| Constructor injection | `New*()` functions | All deps passed as params |
| Interface-based DI | `internal/domain/` | Interfaces for testability |
| `cmd/` entry point | `cmd/your-service/main.go` | Wires all dependencies |
| `internal/` privacy | `internal/` | Packages not importable externally |
| `pkg/` reusability | `pkg/` | Shared utilities (logger, HTTP helpers) |

---

## 3. Code Conventions (Key Rules)

<!-- CUSTOMIZE: Replace with your project's conventions -->
- **`New*()` constructors** — `NewHandler()`, `NewService()`, `NewCache()`
- **Unexported fields** — struct fields are lowercase unless public API
- **`error` returns** — functions return `(value, error)` tuple
- **`errors.Is()` / `errors.As()`** — for error checking, not `==`
- **`context.Context`** — first param in functions that do I/O
- **`sync.RWMutex`** — for concurrent map access
- **No global state** — dependencies injected via constructors
- **`internal/`** — private packages; `pkg/` for reusable code
- **Interface in consumer** — define interfaces where they're used

**Full rules with code examples:** [docs/code-best-practices.md](docs/code-best-practices.md)

---

## 4. Testing Rules (Key Rules)

<!-- CUSTOMIZE: Replace with your project's test conventions -->
- **Table-driven tests** — `[]struct{ name, input, want }` pattern
- **`_test.go` in same package** — unit tests alongside source
- **`tests/integration/`** — integration tests with `testcontainers`
- **`httptest`** — `httptest.NewRequest()` + `httptest.NewRecorder()`
- **Test naming** — `TestFunctionName__scenario` (double underscore)
- **No test frameworks** — use stdlib `testing` package
- **Subtests** — `t.Run("scenario", func(t *testing.T) { ... })`

**Full rules with code examples:** [docs/test-conventions.md](docs/test-conventions.md)

---

## 5. Agent Workflow

### Step 1: Create Branch

- **Never commit directly to `main`** — always create a dedicated branch
- Branch naming: `feature/short-description`, `fix/short-description`
- One branch per logical change

### Step 2: Read

- Read this file for project structure and patterns
- Read the relevant `docs/` files for detailed conventions

### Step 3: Find Similar Code

- Search for similar handlers/services in `internal/`
- Study existing patterns in `cmd/` for dependency wiring
- Check `pkg/` for available utilities

### Step 4: Plan

- Identify which packages to create or modify
- Check if interfaces exist for the domain
- Plan the full stack: handler → service → repository

### Step 5: Generate

- Follow existing patterns (copy structure from similar packages)
- Use constructor injection for all dependencies
- Define interfaces in the consumer package
- Place files in correct `internal/` subdirectories

### Step 6: Verify

<!-- CUSTOMIZE: Replace with your test commands -->
```bash
make test                     # Unit tests
make integration-test         # Integration tests
go vet ./...                  # Static analysis
```

### Step 7: Review & Submit

- **Self-review all changes** before creating a pull request
  - Run `git diff` and review every changed file for correctness, style, and conventions
  - Verify no debug code, leftover TODOs, or unintended changes are included
  - Add meaningful comments where logic isn't self-evident — avoid redundant or obvious comments
  - Confirm tests pass and code compiles
- **Commit** — follow [docs/git-conventions.md](docs/git-conventions.md); run tests before committing
- **Create a pull request** — PRs are required for all changes to be merged

---

## 6. Git Conventions

<!-- CUSTOMIZE: Replace with your project's commit format -->
- **Never commit directly to `main`** — always work on a dedicated branch
- **Commit format:** Descriptive message explaining the change
- **PR template:** Description, implemented changes, ticket link
- **Branch naming:** `feature/`, `fix/`, `chore/` prefixes
- **Self-review required** — review all changes before creating a pull request
- **PRs required** — all changes merge through pull requests

**Full details:** [docs/git-conventions.md](docs/git-conventions.md)

---

## Quick Links

| Document | Content |
|---|---|
| [docs/code-best-practices.md](docs/code-best-practices.md) | Constructor injection, error handling, concurrency, interfaces |
| [docs/test-conventions.md](docs/test-conventions.md) | Table-driven tests, httptest, testcontainers, mocking |
| [docs/git-conventions.md](docs/git-conventions.md) | Commit format, PR template, branch conventions |
