# Git Conventions

> Commit and PR conventions. See [AGENTS.md](../AGENTS.md) for quick reference.

---

## Commit Messages

<!-- CUSTOMIZE: Replace with your project's commit conventions -->
Use descriptive commit messages explaining the change:

```
Add location storage use case with validation
Fix cache timeout handling for Redis failures
Update SQLModel schema for new user fields
Remove deprecated authentication endpoint
```

---

## Pull Request Template

<!-- CUSTOMIZE: Replace with your project's PR template -->
Every PR should include:

- **PR description** — what changed and why
- **Implemented** — specific changes made
- **Checklist:**
  - [ ] `make ruff` passes
  - [ ] `make mypy` passes
  - [ ] `make test` passes
  - [ ] Alembic migration included (if DB changes)
- **JIRA link** — link to the related ticket

---

## Branch Naming

<!-- CUSTOMIZE: Replace with your project's branch convention -->
Branches are typically named after their ticket or feature:

```
feature/add-location-storage
fix/cache-timeout-handling
chore/upgrade-fastapi-version
```

---

## Pre-commit Hooks

<!-- CUSTOMIZE: Replace with your project's hooks -->
Ruff runs on commit via pre-commit:

- Lint check (`ruff check --fix`)
- Format check (`ruff format`)
- Ensures consistent code style
