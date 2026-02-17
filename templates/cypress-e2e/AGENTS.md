# AI Agent Workflow Guide

> Comprehensive guide for AI coding agents working on this Cypress E2E test codebase.
> For detailed rules with code examples, see the `docs/` directory.

---

## 0. Navigation Contract

When starting a task, traverse documentation in this order:

1. **This file** (`AGENTS.md`) — project structure, patterns, key rules
2. **`docs/` files** — detailed conventions (only when you need code examples)

---

## 1. Project Overview

<!-- CUSTOMIZE: Replace with your project description -->
Your Project Name — a Cypress E2E test suite covering web applications.
Describe what applications are tested (e.g., Carrier TMS, Shipper TMS, Customer Portal).

### Prerequisites

<!-- CUSTOMIZE: Replace with your project's versions -->
- **Node.js 18+**
- **pnpm 8+**
- **Cypress 15.x**
- **TypeScript 5.x**

### Key Commands

<!-- CUSTOMIZE: Replace with your project's actual commands -->
```bash
pnpm install                              # Install dependencies
pnpm e2e --target staging                 # Run all E2E tests against staging
pnpm e2e:open --target staging            # Open Cypress UI against staging
pnpm e2e --target production              # Run against production
```

### Environment Variables

<!-- CUSTOMIZE: Replace with your project's env file pattern -->
Application loads env files from root: `.env.staging`, `.env.production`.
Follow Cypress env naming convention: `CYPRESS_` prefix (e.g., `CYPRESS_STMS_URL`).

**Never commit `.env.*` files.** They contain credentials.

### Directory Structure

<!-- CUSTOMIZE: Replace with your project's actual directory structure -->
```
src/
├── e2e/                    # Test specs organized by application
│   ├── stms/              # Shipper TMS tests
│   ├── ctms/              # Carrier TMS tests
│   ├── cp/                # Customer Portal tests
│   ├── payments/          # Payment flow tests (cross-app)
│   │   ├── stms/
│   │   └── ctms/
│   ├── load_matching/     # Load matching tests
│   └── trust/             # Trust & safety tests
├── support/
│   ├── commands/          # Custom Cypress commands (API calls, auth, helpers)
│   ├── helpers/           # Reusable helper functions
│   ├── po/                # Page Objects organized by app
│   │   ├── STMS/
│   │   ├── CTMS/
│   │   ├── CP/
│   │   └── SupportPages/
│   ├── prerequisites/     # Setup specs (data preparation)
│   └── index.ts           # Command imports + type declarations
├── fixtures/              # Static test data (JSON files)
├── testutils/             # Test utilities
│   ├── schemas/           # Request body builders by domain
│   ├── random.ts          # Random data generators
│   ├── dates.ts           # Date helpers
│   └── types.ts           # Shared TypeScript types
└── cypress.config.ts      # Cypress configuration
```

---

## 2. Key Patterns

### Page Objects (`support/po/`)

<!-- CUSTOMIZE: Replace with your project's PO conventions -->
Page Objects encapsulate UI interactions for a specific page:

- One class per page, exported as both class and singleton instance
- Methods return `Cypress.Chainable` for element getters, `void` for actions
- Group: element getters → actions → assertions
- Use `@testing-library/cypress` queries (`findByRole`, `findByLabelText`, `findByText`)

### Custom Commands (`support/commands/`)

API-level operations registered as `cy.*` commands:

- Auth commands: `cy.authSTMS()`, `cy.authCTMS()`, `cy.authCP()`
- CRUD commands: `cy.createOrderAPI()`, `cy.deleteOrderAPI()`
- All commands declared in `support/index.ts` (Cypress namespace augmentation)

### Test Data (`testutils/`)

- `random.*` — random generators for names, prices, addresses, VINs, emails
- `schemas/*` — request body builders for API calls
- `dates.*` — date formatting and calculation helpers

**Full patterns with code examples:** [docs/code-best-practices.md](docs/code-best-practices.md)

---

## 3. Test Writing Rules (Key Rules)

<!-- CUSTOMIZE: Replace test ID pattern with your project's convention -->
- **Test IDs:** `ATC-XXX` prefix in test name (linked to test management tool)
- **Structure:** `describe` per feature → `it` per test case
- **Setup:** Use `beforeEach` for auth and navigation, API calls for data setup
- **Selectors:** Prefer `findByRole` > `findByLabelText` > `findByText` > `findByTestId`
- **No `cy.wait()`** — use assertions or `cy.intercept()` to wait for conditions
- **No `cy.get()` with CSS selectors** when Testing Library queries work
- **No `.force()`** unless absolutely necessary (ESLint warns)
- **No `.only()`** — ESLint enforces `no-only-tests`
- **API setup over UI setup** — use custom commands to create test data via API
- **Clean up test data** — delete created entities in `beforeEach` or `afterEach`

**Full rules with code examples:** [docs/test-conventions.md](docs/test-conventions.md)

---

## 4. Agent Workflow

### Step 1: Create Branch

- **Never commit directly to `main`** — always create a dedicated branch
- Branch naming: `feature/short-description`, `fix/short-description`
- One branch per logical change

### Step 2: Read

- Read this file for project structure and patterns
- Read the relevant `docs/` files for detailed conventions

### Step 3: Find Similar Tests

- Search for tests covering similar features or apps in `src/e2e/`
- Study existing page objects in `support/po/` for the target app
- Check `support/commands/` for available API helpers

### Step 4: Plan

- Identify which spec file to create or modify
- Check if page objects exist for the pages being tested
- Check if custom commands exist for needed API operations
- Plan data setup and teardown

### Step 5: Generate

- Follow existing patterns (copy structure from similar test files)
- Use page objects for UI interactions
- Use custom commands for API calls and auth
- Use `testutils/random.*` for test data generation
- Place specs in the correct `src/e2e/{app}/` directory

### Step 6: Run

<!-- CUSTOMIZE: Replace with your test commands -->
- Run locally: `pnpm e2e:open --target staging`
- Verify test passes in Cypress UI before committing

### Step 7: Review & Submit

- **Self-review all changes** before creating a pull request
  - Run `git diff` and review every changed file for correctness, style, and conventions
  - Verify no debug code, leftover TODOs, or unintended changes are included
  - Add meaningful comments where logic isn't self-evident — avoid redundant or obvious comments
- **Commit** — follow [docs/git-conventions.md](docs/git-conventions.md); run linter before committing
- **Create a pull request** — PRs are required for all changes to be merged

---

## 5. Git Conventions

<!-- CUSTOMIZE: Replace with your project's commit format -->
- **Never commit directly to `main`** — always work on a dedicated branch
- **Commit format:** Descriptive message explaining the change
- **Never fabricate ticket numbers or issue IDs** — if none exists, omit it
- **PR template:** Fill in description, implemented changes, JIRA link
- **Self-review required** — review all changes before creating a pull request
- **PRs required** — all changes merge through pull requests
- **Follow `.github/PULL_REQUEST_TEMPLATE.md`** if the project has one

**Full details:** [docs/git-conventions.md](docs/git-conventions.md)

---

## Quick Links

| Document | Content |
|---|---|
| [docs/code-best-practices.md](docs/code-best-practices.md) | Page objects, custom commands, selectors, TypeScript patterns |
| [docs/test-conventions.md](docs/test-conventions.md) | Test structure, naming, data setup, anti-patterns |
| [docs/git-conventions.md](docs/git-conventions.md) | Commit format, PR template, branch conventions |
