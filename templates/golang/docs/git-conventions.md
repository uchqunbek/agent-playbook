# Git Conventions

> Commit and PR conventions. See [AGENTS.md](../AGENTS.md) for quick reference.

---

## Commit Messages

<!-- CUSTOMIZE: Replace with your project's commit conventions -->
Use descriptive commit messages explaining the change:

```
Add token revocation cache with RWMutex
Fix connection manager retry backoff logic
Update RabbitMQ stream consumer error handling
Remove deprecated health check endpoint
```

---

## Pull Request Template

<!-- CUSTOMIZE: Replace with your project's PR template -->
Every PR should include:

- **PR description** — what changed and why
- **Implemented** — specific changes made
- **Checklist:**
  - [ ] `make test` passes
  - [ ] `make integration-test` passes (if applicable)
  - [ ] `go vet ./...` clean
  - [ ] No new linter warnings
- **JIRA link** — link to the related ticket

---

## Branch Naming

<!-- CUSTOMIZE: Replace with your project's branch convention -->
Branches are typically named after their ticket or feature:

```
feature/add-token-cache
fix/connection-retry-logic
chore/upgrade-go-version
```

---

## Pre-commit Checks

<!-- CUSTOMIZE: Replace with your project's pre-commit checks -->
Run before committing:

```bash
go test ./... -v              # All unit tests pass
go vet ./...                  # Static analysis clean
```
