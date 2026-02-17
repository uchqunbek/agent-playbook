# Git Conventions

> Commit and PR conventions. See [AGENTS.md](../AGENTS.md) for quick reference.

---

## Commit Messages

<!-- CUSTOMIZE: Replace with your project's commit conventions -->
Use descriptive commit messages explaining the change.

**Never fabricate ticket numbers or issue IDs** — if no Jira ticket exists, omit it entirely.

```
Add token revocation cache with RWMutex
Fix connection manager retry backoff logic
Update RabbitMQ stream consumer error handling
Remove deprecated health check endpoint
```

---

## Pull Request Template

<!-- CUSTOMIZE: Replace with your project's PR template -->
If the project has a `.github/PULL_REQUEST_TEMPLATE.md`, read it and follow its structure exactly.

Otherwise, every PR should include:

- **PR description** — what changed and why
- **Implemented** — specific changes made
- **Checklist:**
  - [ ] `make test` passes
  - [ ] `make integration-test` passes (if applicable)
  - [ ] `go vet ./...` clean
  - [ ] No new linter warnings
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
feature/add-token-cache
fix/connection-retry-logic
chore/upgrade-go-version
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
- [ ] Verify `go vet ./...` is clean
- [ ] Ensure commit messages follow the format above
- [ ] Confirm no unintended files are included in the changeset

---

## Pre-commit Checks

<!-- CUSTOMIZE: Replace with your project's pre-commit checks -->
Run before committing:

```bash
go test ./... -v              # All unit tests pass
go vet ./...                  # Static analysis clean
```
