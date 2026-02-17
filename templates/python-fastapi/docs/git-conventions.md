# Git Conventions

> Commit and PR conventions. See [AGENTS.md](../AGENTS.md) for quick reference.

---

## Commit Messages

<!-- CUSTOMIZE: Replace ticket prefixes with your project's -->
Format: `[TICKET-NUMBER] type: description`

Types: `feature`, `fix`, `misc`, `refactor`

```
# Good
[GPS-123] feature: Add location filtering by time range
[GPS-456] fix: Handle cache timeout in auth backend
[GPS-789] refactor: Extract location validation to private method
[GPS-101] misc: Update uv.lock dependencies

# Bad
fix bug                    # Missing ticket
[GPS-123] updated code     # Vague description
updated stuff              # No ticket, no type, vague
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
feature/GPS-123-add-location-filtering
fix/GPS-456-cache-timeout
chore/upgrade-fastapi-version
```

---

## Pre-commit Hooks

<!-- CUSTOMIZE: Replace with your project's hooks -->
Ruff runs on commit via pre-commit:

- Lint check (`ruff check --fix`)
- Format check (`ruff format`)
- Ensures consistent code style
