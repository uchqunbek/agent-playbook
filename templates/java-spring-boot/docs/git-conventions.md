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
- **Never fabricate ticket numbers** — if no Jira ticket exists, omit the prefix entirely

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

## Branch Workflow

<!-- CUSTOMIZE: Replace with your project's branch workflow -->
**Never commit directly to `main`.** Every change requires a dedicated branch and pull request.

1. Create a branch from `main` before starting work
2. Make changes and commit to the branch
3. Self-review all changes before creating a PR
4. Create a pull request for review
5. Merge only after approval

### Branch Naming

<!-- CUSTOMIZE: Replace with your project's branch naming conventions -->
Common prefixes:

- `feature/STMS-XXXX-short-description`
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
- [ ] Verify all tests pass (`./gradlew test`)
- [ ] Verify code compiles (`./gradlew build`)
- [ ] Verify formatting is applied (`./gradlew spotlessApply`)
- [ ] Ensure commit messages follow the format above
- [ ] Confirm no unintended files are included in the changeset
- [ ] Follow `.github/PULL_REQUEST_TEMPLATE.md` if the project has one
