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

## Branch Naming

<!-- CUSTOMIZE: Replace with your project's branch convention -->
Branches are typically named after their ticket or feature:

```
feature/add-superpay-onboarding-tests
fix/flaky-login-test
chore/update-cypress-version
```

---

## Pre-commit Hooks

<!-- CUSTOMIZE: Replace with your project's hooks -->
Husky runs lint-staged on commit:

- ESLint + Prettier auto-fix on all staged files
- Ensures no `.only()` tests are committed
- Ensures consistent formatting
