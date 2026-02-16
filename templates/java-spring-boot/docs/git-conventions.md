# Git Conventions

> Quick reference for commit and branch rules.

---

## Commit Message Format

```
[TICKET-NUMBER] Short description of the change
```

<!-- CUSTOMIZE: Replace ticket prefixes with your project's JIRA project keys -->
- **Ticket number** must be a JIRA project key in square brackets: `[STMS-1234]`, `[PAYM-5678]`
- **Description** should be concise and describe *what* the change does

### Branch-Specific Rules

<!-- CUSTOMIZE: Replace branch patterns with your project's conventions -->
| Branch Pattern | Commit Format | Notes |
|---|---|---|
| `hotfix*`, `chore*` | Free-form (non-empty) | No ticket number required |
| All other branches | `[PROJECT-XXXX] Description` | Enforced by git hook |

### Good Examples

<!-- CUSTOMIZE: Replace with examples from your project's recent history -->
```
[STMS-5515] Disable logo content validation on image download
[PAYM-3565] Fix payment type naming
[STMS-5593] Remove BE direct Heap related event
[STMS-5513] Add is_restricted to any Order/Vehicle DTOs
```

### Bad Examples

```
Fixed bug                          # No ticket number
[stms-1234] did stuff              # Project key must be UPPERCASE
Updated the order service          # No ticket number
[STMS-1234]                        # Empty description
```

---

## Branch Naming

<!-- CUSTOMIZE: Replace with your project's branch naming conventions -->
Common prefixes:

- `feature/STMS-XXXX-short-description`
- `hotfix/short-description`
- `chore/short-description`
