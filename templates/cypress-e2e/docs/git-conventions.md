# Git Conventions

> Commit and PR conventions. See [AGENTS.md](../AGENTS.md) for quick reference.

---

## Commit Messages

<!-- CUSTOMIZE: Replace with your project's commit conventions -->
Use descriptive commit messages explaining the change:

```
Add login E2E tests for 2FA and magic link flows
Fix flaky order creation test — wait for API response
Update page objects for new order details redesign
Remove deprecated load matching tests
```

---

## Pull Request Template

<!-- CUSTOMIZE: Replace with your project's PR template -->
Every PR should include:

- **PR description** — what changed and why
- **Implemented** — specific changes made
- **Checklist:**
  - [ ] Tests ran locally
  - [ ] Code follows style guidelines
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
feature/add-superpay-onboarding-tests
fix/flaky-login-test
chore/update-cypress-version
```

---

## Self-Review Before PR

Before creating a pull request, the agent MUST review its own changes:

<!-- CUSTOMIZE: Replace with your project's review checklist -->
- [ ] Run `git diff` and read every changed line
- [ ] Verify code follows conventions in [AGENTS.md](../AGENTS.md) and this docs/ directory
- [ ] Confirm no debug code, leftover TODOs, or `.only()` tests
- [ ] Add meaningful comments where logic isn't self-evident — avoid redundant or obvious comments
- [ ] Verify tests pass locally
- [ ] Ensure linting passes
- [ ] Confirm no unintended files are included in the changeset

---

## Pre-commit Hooks

<!-- CUSTOMIZE: Replace with your project's hooks -->
Husky runs lint-staged on commit:

- ESLint + Prettier auto-fix on all staged files
- Ensures no `.only()` tests are committed
- Ensures consistent formatting
