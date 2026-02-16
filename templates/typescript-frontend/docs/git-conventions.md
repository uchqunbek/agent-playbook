# Git Conventions

> Commit and branch conventions. See [AGENTS.md](../AGENTS.md) for quick reference.

---

## Commit Message Format

<!-- CUSTOMIZE: Replace with your project's commit format -->
```
[TICKET-NUMBER] Short description of the change
```

- **Ticket number** in square brackets: `[FE-1234]`, `[UI-5678]`
- **Description** should be concise and describe *what* the change does

### Branch-Specific Rules

<!-- CUSTOMIZE: Replace with your project's branch patterns -->
| Branch Pattern | Commit Format | Notes |
|---|---|---|
| `hotfix*`, `chore*` | Free-form (non-empty) | No ticket number required |
| All other branches | `[PROJECT-XXXX] Description` | Enforced by git hook |

### Good Examples

<!-- CUSTOMIZE: Replace with examples from your project's recent history -->
```
[FE-1234] Add user profile page with avatar upload
[UI-567] Fix date picker not closing on outside click
[FE-890] Refactor form validation to use React Hook Form
[UI-234] Update button component to support loading state
```

### Bad Examples

```
Fixed bug                          # No ticket number
updated styles                     # No ticket number, vague
[FE-1234]                          # Empty description
WIP                                # Not descriptive
```

---

## Branch Naming

<!-- CUSTOMIZE: Replace with your project's branch naming convention -->
Common prefixes:

- `feature/FE-XXXX-short-description`
- `hotfix/short-description`
- `chore/short-description`

---

## Pre-commit Checks

<!-- CUSTOMIZE: Replace with your project's pre-commit hooks -->
Before committing, ensure:

1. `pnpm lint` passes (no ESLint/Prettier errors)
2. `pnpm typecheck` passes (no TypeScript errors)
3. `pnpm test` passes (no test failures)
