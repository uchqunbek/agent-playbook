# Git Conventions

> Commit, branch, and hook conventions. See [AGENTS.md](../AGENTS.md) for quick reference.

---

## Commit Message Format

<!-- CUSTOMIZE: Replace ticket prefixes with your project's -->
```
[TICKET-NUMBER] Short description of the change
```

- **Ticket number** in square brackets: `[TMS-1234]`, `[TMS-567]`
- **Description** should be concise and describe *what* the change does

### Branch-Specific Rules

<!-- CUSTOMIZE: Replace with your project's branch patterns -->
| Branch Pattern | Commit Format | Notes |
|---|---|---|
| `hotfix*`, `chore*` | Free-form (non-empty) | No ticket number required |
| All other branches | `[TMS-XXXX] Description` | Enforced by Husky commit-msg hook |

### Good Examples

<!-- CUSTOMIZE: Replace with examples from your project's recent history -->
```
[TMS-1234] Add driver detail page with card layout
[TMS-567] Fix phone field validation in driver form
[TMS-890] Refactor driver list to use useAPIListQuery
[TMS-234] Update DriverCard to show inactive badge
```

### Bad Examples

```
Fixed bug                          # No ticket number
updated styles                     # No ticket number, vague
[TMS-1234]                         # Empty description
WIP                                # Not descriptive
```

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

<!-- CUSTOMIZE: Replace with your project's branch naming convention -->
Common prefixes:

- `feature/TMS-XXXX-short-description`
- `hotfix/short-description`
- `chore/short-description`

---

## Self-Review Before PR

Before creating a pull request, the agent MUST review its own changes:

<!-- CUSTOMIZE: Replace with your project's review checklist -->
- [ ] Run `git diff` and read every changed line
- [ ] Verify code follows conventions in [AGENTS.md](../AGENTS.md) and this docs/ directory
- [ ] Confirm no debug code, leftover TODOs, or commented-out code
- [ ] Add meaningful comments where logic isn't self-evident — avoid redundant or obvious comments
- [ ] Verify `pnpm check` passes (ESLint + Prettier)
- [ ] Verify `pnpm check:types` passes (TypeScript)
- [ ] Verify `pnpm test` passes (all tests green)
- [ ] Ensure commit messages follow the format above
- [ ] Confirm no unintended files are included in the changeset

---

## Pre-commit Hooks (Husky)

<!-- CUSTOMIZE: Replace with your project's Husky hook setup -->
This project uses Husky for git hooks. **Never bypass with `--no-verify`.**

| Hook | Command | Purpose |
|---|---|---|
| `pre-commit` | `pnpm check` | ESLint + Prettier formatting |
| `pre-push` | `pnpm check:types` | TypeScript type checking |
| `commit-msg` | Ticket format validator | Enforces `[TMS-XXXX]` prefix |
| `post-merge` | `pnpm install --force` | Ensures dependencies are up to date |

**Important:** pnpm is enforced — npm and yarn are blocked by the `preinstall` script.

---

## PR Checklist

Before opening a pull request, verify:

<!-- CUSTOMIZE: Replace with your project's PR checks -->
- [ ] `pnpm check` passes (ESLint + Prettier)
- [ ] `pnpm check:types` passes (TypeScript)
- [ ] `pnpm test` passes (all tests green)
- [ ] JIRA ticket linked in PR description
- [ ] No cross-feature imports (use `shared/` or `core/` for shared code)
- [ ] New components use `function` keyword, named exports, `styled()`
- [ ] New tests use `renderWithProviders()` and MSW v1 syntax
