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

## Branch Naming

<!-- CUSTOMIZE: Replace with your project's branch naming convention -->
Branches are typically named after their ticket:

```
origin/CAR-8538
origin/PLT-2499
```

---

## Best Practices

1. Keep descriptions concise — under 72 characters
2. Use imperative mood — "Add feature" not "Added feature"
3. Reference ticket in every commit
4. One logical change per commit
