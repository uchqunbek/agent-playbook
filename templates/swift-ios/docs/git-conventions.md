# Git Conventions

> Commit and PR conventions. See [AGENTS.md](../AGENTS.md) for quick reference.

---

## Commit Messages

<!-- CUSTOMIZE: Replace with your project's commit conventions -->
Format: `[TICKET-123] Description of the change`

**Never fabricate ticket numbers or issue IDs** — if no Jira ticket exists, omit the prefix entirely.

```
# Good
[TMS-1234] Add driver list view with search filtering
[TMS-567] Fix crash when driver phone is nil
[TMS-890] Update networking layer to async/await

# Relaxed (chore/hotfix — no ticket required)
Update SwiftLint rules for Swift 6 compatibility
Fix Fastlane test lane configuration

# Bad
fix bug                    # Missing ticket and vague
[TMS-123] updated code     # Vague description
wip                        # Never commit WIP to shared branches
```

---

## Branch Naming

<!-- CUSTOMIZE: Replace with your project's branch convention -->

| Prefix | Use Case | Example |
|--------|----------|---------|
| `feature/` | New functionality | `feature/TMS-1234-driver-list` |
| `bugfix/` | Bug fixes | `bugfix/TMS-567-nil-phone-crash` |
| `improvement/` | Refactoring, optimization | `improvement/TMS-890-async-networking` |
| `chore/` | Tooling, CI, dependencies | `chore/update-swiftlint-rules` |

---

## Branch Workflow

<!-- CUSTOMIZE: Replace with your project's branch workflow -->
**Never commit directly to `main`.** Every change requires a dedicated branch and pull request.

1. **Pull latest** `main` before branching
2. **Create branch** from `main` using the naming convention above
3. **Make changes** in small, focused commits
4. **Run tests** — `xcodebuild test` must pass before committing
5. **Self-review** all changes before creating a PR
6. **Create pull request** targeting `main`
7. **Merge** only after approval

---

## Pull Request Template

<!-- CUSTOMIZE: Replace with your project's PR template -->
If the project has a `.github/PULL_REQUEST_TEMPLATE.md`, read it and follow its structure exactly.

Otherwise, every PR should include:

- **Description** — what changed and why
- **Changes** — bullet list of specific changes
- **Screenshots** — for UI changes (before/after)
- **Checklist:**
  - [ ] Project builds without warnings
  - [ ] All tests pass (`xcodebuild test`)
  - [ ] SwiftLint passes (no new warnings)
  - [ ] SwiftFormat applied
  - [ ] UI tested on multiple device sizes (if applicable)
- **Ticket link** — link to the related Jira ticket

---

## Self-Review Before PR

Before creating a pull request, the agent MUST review its own changes:

<!-- CUSTOMIZE: Replace with your project's review checklist -->
- [ ] Run `git diff` and read every changed line
- [ ] Verify code follows conventions in [AGENTS.md](../AGENTS.md) and this docs/ directory
- [ ] Confirm no debug code, `print()` statements, or leftover TODOs
- [ ] Add meaningful comments where logic isn't self-evident — avoid redundant or obvious comments
- [ ] Build the project — zero warnings
- [ ] Run all tests — `xcodebuild test` passes
- [ ] Run SwiftLint — no new violations
- [ ] Run SwiftFormat — all files formatted
- [ ] Ensure commit messages follow the format above
- [ ] Confirm no unintended files are included in the changeset (e.g., `.DS_Store`, `xcuserdata/`)

---

## Pre-commit Checks

<!-- CUSTOMIZE: Replace with your project's pre-commit hook setup -->
Run before committing (enforced via Git pre-commit hook):

```bash
# Pre-commit hook runs automatically:
swiftformat --lint .              # Check formatting
swiftlint                         # Check lint rules

# Full verification before PR:
xcodebuild test -scheme YourApp -destination 'platform=iOS Simulator,name=iPhone 16'
```

### Hook Setup

<!-- CUSTOMIZE: Replace with your project's hook installation -->
```bash
# Install pre-commit hook (one-time setup)
cp scripts/pre-commit .git/hooks/pre-commit
chmod +x .git/hooks/pre-commit
```

---

## .gitignore Essentials

<!-- CUSTOMIZE: Verify these match your project's .gitignore -->
Ensure these are in `.gitignore`:

```
# Xcode
xcuserdata/
*.xcworkspace/xcuserdata/
DerivedData/

# SPM
.build/
.swiftpm/

# Secrets
Secrets.swift
*.xcconfig.local

# System
.DS_Store
```
