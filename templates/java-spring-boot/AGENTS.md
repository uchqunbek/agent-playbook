# AI Agent Workflow Guide

> Comprehensive guide for AI coding agents working on this codebase.
> For detailed rules with code examples, see the `docs/` directory.

---

## 0. Navigation Contract

When starting a task, traverse documentation in this order:

1. **This file** (`AGENTS.md`) — workflow, bootstrapping, architecture, key rules
2. **`docs/` files** — detailed conventions (only when you need code examples)

---

## 1. Project Overview

<!-- CUSTOMIZE: Replace with your project description -->
Your Project Name — a Spring Boot application serving your domain.
Describe what the application does in 1-2 sentences.

### Prerequisites

<!-- CUSTOMIZE: Replace with your project's versions -->
- **Java 21** (toolchain enforced)
- **Gradle 9.x** (use `./gradlew` wrapper)
- **Spring Boot 3.x**

### Key Commands

<!-- CUSTOMIZE: Replace with your project's actual commands -->
```bash
./gradlew build                    # Full build + tests
./gradlew test                     # All tests
./gradlew test --tests "com.example.service.YourServiceTest"  # Single class
./gradlew test --tests "*.YourServiceTest.method_name*"       # Single method
./gradlew spotlessApply            # Format code
```

### Environment Variables

<!-- CUSTOMIZE: Describe where config lives -->
Check `application.yml` and `application-local.yml` for required config. Tests use
TestContainers (PostgreSQL, etc.).

### Directory Structure

<!-- CUSTOMIZE: Replace with your project's package structure -->
```
src/main/java/com/example/project/
├── client/          # External service HTTP clients
├── controller/      # REST controllers (internal + public APIs)
├── core/            # Config, security, logging, utilities
├── domain/          # Domain entities and value objects
├── dto/             # Request/response DTOs
├── repository/      # JPA repositories and entity definitions
└── service/         # Core business logic

src/main/resources/
├── config/          # Spring configuration (application*.yml)
└── db/migration/    # Flyway migrations

src/test/java/com/example/project/
└── core/
    └── config/      # Test base classes + test helpers
```

### Architecture

<!-- CUSTOMIZE: Replace with your project's actual architecture -->
```
Controller (@JSendResponse)
    → Service (@SpringTransactional, @CheckPermission)
        → Repository (JPA)
        → Client (external APIs)
        → Message (RabbitMQ producers)
```

<!-- CUSTOMIZE: Replace with your project's DI and auth patterns -->
- **DI via constructor** — `@RequiredArgsConstructor` + `final` fields
- **Transactions** — use `@SpringTransactional` (custom, rolls back on all Throwable)
- **Auth** — `@CurrentAuthenticatedUser AuthenticatedUser` in controller params
- **Permissions** — `@CheckPermission(PermissionObject.XXX)` on controller/service methods
- **Responses** — `@JSendResponse` on controller methods for standardized JSON format

---

## 2. Code Conventions (Key Rules)

<!-- CUSTOMIZE: Replace with your project's code rules -->
- **120-char line limit** (`.editorconfig`)
- Break long signatures after `(`, one argument per line
- `getFirst()` over `get(0)`
- No obvious comments — use self-descriptive names
- DI via constructor: `@RequiredArgsConstructor` + `final` fields
- Use `@SpringTransactional` (not plain `@Transactional`)
- Always use merge function with `Collectors.toMap`
- Use enums over string constants

### Custom Annotations

<!-- CUSTOMIZE: Replace with your project's custom annotations -->
| Annotation | Package | Purpose |
|---|---|---|
| `@JSendResponse` | `controller` | JSend-formatted JSON response wrapper |
| `@SpringTransactional` | `core` | `@Transactional(rollbackFor = Throwable.class)` with readOnly/propagation |
| `@CurrentAuthenticatedUser` | `core.config.security` | Extracts authenticated user from security context |
| `@CheckPermission` | `core.config.security.permissions` | Permission enforcement via `PermissionObject` enum |

**Full rules with code examples:** [docs/code-best-practices.md](docs/code-best-practices.md)

---

## 3. Testing Rules (Key Rules)

<!-- CUSTOMIZE: Replace with your project's test conventions -->
- **Naming:** `{method}_{condition(s)}__{expected_result}` — all lowercase, double underscore
- **Structure:** Given-When-Then with section comments
- **Integration tests:** extend `AbstractServiceTest`, use real repositories, use `FAKER` for data
- **Controller tests:** extend `AbstractWebMvcTest`, assert `handler().handlerType()` and `handler().method()`
- **DO NOT** use `@SpringTransactional` on tests
- **DO NOT** mock repositories in integration tests
- **DO NOT** hardcode test values — use `FAKER`
- **DO** use `@ParameterizedTest` + `@MethodSource` instead of duplicating tests

### Test Infrastructure

<!-- CUSTOMIZE: Replace with your project's test base classes -->
| Class | Purpose |
|---|---|
| `AbstractServiceTest` | Full Spring context, real repos, mocked external services |
| `AbstractWebMvcTest` | MockMvc, all services mocked |
| `AbstractWebMvcFullContextTest` | Full context + MockMvc |

<!-- CUSTOMIZE: Replace with your project's test helpers -->
**Test helpers:** `TestUserService`, `TestOrderService` — builder-pattern services for
creating test entities. `@Autowired` in tests extending `AbstractServiceTest`.

**Test data:** `import static com.example.project.core.TestUtils.FAKER;` — DataFaker for random values.

**Full rules with code examples:** [docs/test-conventions.md](docs/test-conventions.md)

---

<!-- CUSTOMIZE: Add domain-specific sections as needed (Messaging, Migrations, etc.) -->
<!-- Example: -->
<!-- ## 4. Messaging (RabbitMQ) -->
<!-- - Spring AMQP with `@EnableRabbit` -->
<!-- - Message producers live in `message/` package -->

<!-- ## 5. Database Migrations (Flyway) -->
<!-- - Path: `src/main/resources/db/migration/` -->
<!-- - Naming: `V{VERSION}__{DESCRIPTION}.sql` -->

---

## 6. Agent Workflow

Follow this step-by-step process for every task:

### Step 1: Create Branch

- **Never commit directly to `main`** — always create a dedicated branch
- Branch naming: `feature/TICKET-XXXX-short-description`, `fix/TICKET-XXXX-short-description`
- One branch per logical change

### Step 2: Read

- Read this file for project context
- Read the relevant `docs/` files for detailed conventions

### Step 3: Find Similar Code

- Search for similar features, patterns, or components in the codebase
- Study how existing code handles the same concerns (auth, transactions, DTOs, tests)

### Step 4: Plan

- Identify which files to create or modify
- List dependencies and side effects
- Consider test strategy before writing production code

### Step 5: Generate

- Follow existing patterns exactly (copy structure from similar code)
- Use the project's annotations and conventions
- Place code in the correct package (see Section 1)

### Step 6: Test

- Write tests following [docs/test-conventions.md](docs/test-conventions.md)
<!-- CUSTOMIZE: Replace with your test command -->
- Run tests: `./gradlew test --tests "*.YourNewTest"`
- Fix any failures before proceeding

### Step 7: Review & Submit

- **Self-review all changes** before creating a pull request
  - Run `git diff` and review every changed file for correctness, style, and conventions
  - Verify no debug code, leftover TODOs, or unintended changes are included
  - Add meaningful comments where logic isn't self-evident — avoid redundant or obvious comments
  - Confirm tests pass and code compiles
- **Commit** — format: `[TICKET-NUMBER] Description` (see [docs/git-conventions.md](docs/git-conventions.md))
<!-- CUSTOMIZE: Replace with your lint/format command -->
  - Run `./gradlew spotlessApply` before committing
  - Do not use `--no-verify`
- **Create a pull request** — PRs are required for all changes to be merged

---

## 7. Git Conventions

<!-- CUSTOMIZE: Replace with your project's commit format and ticket prefixes -->
- **Never commit directly to `main`** — always work on a dedicated branch
- **Commit format:** `[PROJECT-XXXX] Description` (e.g., `STMS`, `PAYM`, `SA`)
- **Never fabricate ticket numbers** — if no Jira ticket exists, omit the prefix entirely
- **Relaxed branches:** `hotfix*` and `chore*` — free-form messages allowed
- **All others:** ticket number required (enforced by git hook)
- **Self-review required** — review all changes before creating a pull request
- **PRs required** — all changes merge through pull requests
- **Follow `.github/PULL_REQUEST_TEMPLATE.md`** if the project has one

**Full details:** [docs/git-conventions.md](docs/git-conventions.md)

---

## Quick Links

| Document | Content |
|---|---|
| [docs/code-best-practices.md](docs/code-best-practices.md) | Code style, all code examples |
| [docs/test-conventions.md](docs/test-conventions.md) | Test base classes, helpers, naming, anti-patterns |
| [docs/git-conventions.md](docs/git-conventions.md) | Commit format, branch naming |
| `.editorconfig` | Editor settings (120 chars, 4-space indent, UTF-8, LF) |
