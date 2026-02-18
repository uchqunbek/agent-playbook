# AI Agent Workflow Guide

> Comprehensive guide for AI coding agents working on this Swift/iOS codebase.
> For detailed rules with code examples, see the `docs/` directory.

---

## 0. Navigation Contract

When starting a task, traverse documentation in this order:

1. **This file** (`AGENTS.md`) — project structure, patterns, key rules
2. **`docs/` files** — detailed conventions (only when you need code examples)

---

## 1. Project Overview

<!-- CUSTOMIZE: Replace with your project description -->
Your Project Name — a Swift/iOS application.
Describe what the app does, its domain, and key responsibilities.

### Prerequisites

<!-- CUSTOMIZE: Replace with your project's versions -->
- **Xcode 16.x+**
- **Swift 6**
- **iOS 16+ deployment target**
- **Swift Package Manager** (SPM) for dependencies

### Key Commands

<!-- CUSTOMIZE: Replace with your project's actual commands -->
```bash
xcodebuild test -scheme YourApp -destination 'platform=iOS Simulator,name=iPhone 16'
swift build                       # SPM packages only
swiftlint                         # Lint check
swiftformat .                     # Format all Swift files
fastlane tests                    # Run full test suite via Fastlane
```

### Environment Variables

<!-- CUSTOMIZE: Replace with your project's config approach -->
Configuration via xcconfig files per scheme (Debug / Release).
Secrets stored in `Secrets.swift` (git-ignored) or injected via CI environment.

**Never commit secrets.** Use `.xcconfig` overrides for local development only.

### Directory Structure

<!-- CUSTOMIZE: Replace with your project's actual directory structure -->
```
YourApp/
├── App/
│   ├── AppDelegate.swift         # App lifecycle (UIKit-based)
│   ├── SceneDelegate.swift       # Scene configuration
│   └── AppCoordinator.swift      # Root navigation coordinator
├── Features/
│   ├── Drivers/
│   │   ├── DriversCoordinator.swift
│   │   ├── DriverListViewModel.swift
│   │   ├── DriverListView.swift
│   │   └── DriverDetailView.swift
│   └── Loads/
│       └── ...
├── Core/
│   ├── Networking/               # API client, request/response types
│   ├── Persistence/              # CoreData / SwiftData stack
│   ├── Extensions/               # Foundation & UIKit extensions
│   └── DI/                       # Dependency container
├── Resources/
│   ├── Assets.xcassets
│   └── Localizable.strings
├── DesignSystem/                  # Reusable UI components (SPM module)
YourAppTests/                      # Unit tests (mirrors Features/ structure)
YourAppUITests/                    # UI tests
fastlane/                          # Fastlane configuration
```

---

## 2. Architecture

<!-- CUSTOMIZE: Replace with your project's architecture -->
```
User Action
    ↓
[View] — SwiftUI View or UIViewController
    ↓
[ViewModel] — Business logic, state management (@Published)
    ↓
[Service / Repository] — Data access layer
    ↓
[Networking / Persistence] — URLSession, CoreData
```

### Key Patterns

| Pattern | Location | Description |
|---------|----------|-------------|
| MVVM | `Features/*/` | View + ViewModel per screen |
| Coordinator | `*Coordinator.swift` | Navigation flow management |
| Repository | `Core/Persistence/` | Data access abstraction |
| DI via initializer | `init(service:)` | All deps passed as params |
| Protocol-first | Protocols for services | Interfaces for testability |

---

## 3. Code Conventions (Key Rules)

<!-- CUSTOMIZE: Replace with your project's conventions -->
- **SwiftLint + SwiftFormat** enforced (pre-commit hook)
- **4-space indentation**, 120-character line limit
- **`struct` over `class`** for value types; `class` only for identity/inheritance
- **`@MainActor`** on all UI-bound types (ViewModels, Coordinators)
- **DI via initializers** — protocol-typed parameters
- **`async/await`** for new asynchronous code; avoid raw completion handlers
- **`// MARK: -`** sections in files over 100 lines
- **Access control:** `private` by default, expose only what's needed
- **No force unwraps** (`!`) except in tests and IBOutlets

**Full rules with code examples:** [docs/code-best-practices.md](docs/code-best-practices.md)

---

## 4. Testing Rules (Key Rules)

<!-- CUSTOMIZE: Replace with your project's test conventions -->
- **XCTest** for unit and integration tests
- **Test naming:** `test_{method}_{condition}__{expected}` (double underscore)
- **Arrange / Act / Assert** with `// Arrange`, `// Act`, `// Assert` comments
- **Protocol fakes** — manual conformances, no mocking frameworks
- **No real network or filesystem** in unit tests
- **SwiftUI testing:** ViewInspector or snapshot tests
- **Separate test targets** for unit vs. UI tests

**Full rules with code examples:** [docs/test-conventions.md](docs/test-conventions.md)

---

## 5. Agent Workflow

### Step 1: Create Branch

- **Never commit directly to `main`** — always create a dedicated branch
- Branch naming: `feature/TICKET-123-short-description`, `bugfix/TICKET-456-fix`
- One branch per logical change

### Step 2: Read

- Read this file for project structure and patterns
- Read the relevant `docs/` files for detailed conventions

### Step 3: Find Similar Code

- Search for similar Views/ViewModels in `Features/`
- Study existing Coordinators for navigation patterns
- Check `Core/` for available services and extensions

### Step 4: Plan

- Identify which features/modules to create or modify
- Check if protocols exist for the required services
- Plan the full stack: View → ViewModel → Service → Repository

### Step 5: Generate

- Follow existing patterns (copy structure from similar features)
- Use initializer injection for all dependencies
- Define protocols for new services
- Place files in correct `Features/` or `Core/` subdirectories

### Step 6: Verify

<!-- CUSTOMIZE: Replace with your test/lint commands -->
```bash
xcodebuild test -scheme YourApp -destination 'platform=iOS Simulator,name=iPhone 16'
swiftlint                         # Lint check
swiftformat --lint .              # Format check
```

### Step 7: Review & Submit

- **Self-review all changes** before creating a pull request
  - Run `git diff` and review every changed file for correctness, style, and conventions
  - Verify no debug code, leftover TODOs, or unintended changes are included
  - Add meaningful comments where logic isn't self-evident — avoid redundant or obvious comments
  - Confirm tests pass and project builds without warnings
- **Commit** — follow [docs/git-conventions.md](docs/git-conventions.md); run tests before committing
- **Create a pull request** — PRs are required for all changes to be merged

---

## 6. Git Conventions

<!-- CUSTOMIZE: Replace with your project's commit format -->
- **Never commit directly to `main`** — always work on a dedicated branch
- **Commit format:** `[TICKET-123] Description of the change`
- **Never fabricate ticket numbers or issue IDs** — if none exists, omit it
- **Branch naming:** `feature/`, `bugfix/`, `improvement/`, `chore/` + ticket
- **Self-review required** — review all changes before creating a pull request
- **PRs required** — all changes merge through pull requests
- **Follow `.github/PULL_REQUEST_TEMPLATE.md`** if the project has one

**Full details:** [docs/git-conventions.md](docs/git-conventions.md)

---

## Quick Links

| Document | Content |
|---|---|
| [docs/code-best-practices.md](docs/code-best-practices.md) | Naming, struct vs class, SwiftUI/UIKit patterns, async/await, error handling |
| [docs/test-conventions.md](docs/test-conventions.md) | XCTest, protocol fakes, ViewInspector, snapshot tests, test data helpers |
| [docs/git-conventions.md](docs/git-conventions.md) | Commit format, branch naming, self-review checklist, pre-commit hooks |
| `.swiftlint.yml` | SwiftLint rules configuration |
| `.editorconfig` | Editor settings (indent, line length, trailing whitespace) |
