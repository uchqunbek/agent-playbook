# Git Conventions

> Commit and PR conventions. See [AGENTS.md](../AGENTS.md) for quick reference.

---

## Commit Messages

<!-- CUSTOMIZE: Replace ticket prefixes with your project's -->
Format: `[TICKET-NUMBER] type: description`

Types: `feature`, `fix`, `misc`, `refactor`

**Never fabricate ticket numbers** — if no Jira ticket exists, omit the prefix entirely.

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
If the project has a `.github/PULL_REQUEST_TEMPLATE.md`, read it and follow its structure exactly.

Otherwise, every PR should include:

- **PR description** — what changed and why
- **Implemented** — specific changes made
- **Checklist:**
  - [ ] `make ruff` passes
  - [ ] `make mypy` passes
  - [ ] `make test` passes
  - [ ] Alembic migration included (if DB changes)
- **JIRA link** — link to the related ticket

---

## Branch Workflow

<!-- CUSTOMIZE: Replace with your project's branch workflow -->
**Never commit directly to `main`.** Every change requires a dedicated branch and pull request.

1. Create a branch from `main` before starting work
2. Make changes and commit to the branch
3. Self-review all changes before creating a PR
4. Create a pull request for review
5. Merge only after approval

### Branch Naming

<!-- CUSTOMIZE: Replace with your project's branch convention -->
Branches are typically named after their ticket or feature:

```
feature/GPS-123-add-location-filtering
fix/GPS-456-cache-timeout
chore/upgrade-fastapi-version
```

---

## Self-Review Before PR

Before creating a pull request, the agent MUST review its own changes:

<!-- CUSTOMIZE: Replace with your project's review checklist -->
- [ ] Run `git diff` and read every changed line
- [ ] Verify code follows conventions in [AGENTS.md](../AGENTS.md) and this docs/ directory
- [ ] Confirm no debug code, leftover TODOs, or commented-out code
- [ ] Add meaningful comments where logic isn't self-evident — avoid redundant or obvious comments
- [ ] Verify all tests pass (`make test`)
- [ ] Verify type checking passes (`make mypy`)
- [ ] Verify formatting and linting pass (`make ruff`)
- [ ] Ensure commit messages follow the format above
- [ ] Confirm no unintended files are included in the changeset

---

## Pre-commit Hooks

<!-- CUSTOMIZE: Replace with your project's hooks -->
Ruff runs on commit via pre-commit:

- Lint check (`ruff check --fix`)
- Format check (`ruff format`)
- Ensures consistent code style
