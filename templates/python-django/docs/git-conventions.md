# Git Conventions

> Commit and branch conventions. See [AGENTS.md](../AGENTS.md) for quick reference.

---

## Commit Message Format

```
[TICKET-NUMBER] type: description
```

### Components

| Component | Description | Example |
|-----------|-------------|---------|
| `TICKET-NUMBER` | Jira ticket ID | `CAR-8532`, `MOBILE-7345` |
| `type` | Change category | `feature`, `fix`, `misc` |
| `description` | Brief summary | `Add driver deactivation reason` |

**Never fabricate ticket numbers** — if no Jira ticket exists, omit the prefix entirely.

### Commit Types

<!-- CUSTOMIZE: Replace type frequencies with your project's actual usage -->
| Type | Usage |
|------|-------|
| `feature` | New functionality |
| `fix` | Bug fixes |
| `misc` | Cleanup, minor changes |
| `refactor` | Code restructuring (no behavior change) |
| `test` | Test additions/changes only |
| `docs` | Documentation only |
| `config` | Configuration changes |

### Ticket Prefixes

<!-- CUSTOMIZE: Replace with your project's JIRA prefixes -->
| Prefix | Domain |
|--------|--------|
| `CAR-` | Carrier TMS features |
| `MOBILE-` | Mobile app features |
| `PLT-` | Platform/infrastructure |
| `PAYM-` | Payments domain |

---

## Examples

**Good commits:**

```
[CAR-8532] feature: Redirect not built shipments
[MOBILE-7345] fix: Add default field for arrived_at_location
[PLT-2499] misc: Remove deprecated carrier endpoint
[CAR-8500] refactor: Extract shipment status logic to use case
```

**Bad commits:**

```
fix bug                              # Missing ticket, vague description
[CAR-123] updated code               # Vague description
CAR-456: Add feature                 # Missing brackets
[CAR-789] Feature: Add button        # Type should be lowercase
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
Branches are typically named after their ticket:

```
origin/CAR-8538
origin/PLT-2499
```

---

## Self-Review Before PR

Before creating a pull request, the agent MUST review its own changes:

<!-- CUSTOMIZE: Replace with your project's review checklist -->
- [ ] Run `git diff` and read every changed line
- [ ] Verify code follows conventions in [AGENTS.md](../AGENTS.md) and this docs/ directory
- [ ] Confirm no debug code, leftover TODOs, or commented-out code
- [ ] Add meaningful comments where logic isn't self-evident — avoid redundant or obvious comments
- [ ] Verify all tests pass (`make test-all`)
- [ ] Verify formatting and linting pass (`make ruff`)
- [ ] Ensure commit messages follow the format above
- [ ] Confirm no unintended files are included in the changeset
- [ ] Follow `.github/PULL_REQUEST_TEMPLATE.md` if the project has one

---

## Best Practices

1. Keep descriptions concise — under 72 characters
2. Use imperative mood — "Add feature" not "Added feature"
3. Reference ticket in every commit
4. One logical change per commit
