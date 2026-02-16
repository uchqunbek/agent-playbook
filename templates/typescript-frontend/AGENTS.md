# AI Agent Workflow Guide

> Comprehensive guide for AI coding agents working on this codebase.
> For detailed rules with code examples, see the `docs/` directory.

---

## 0. Navigation Contract

When starting a task, traverse documentation in this order:

1. **This file** (`AGENTS.md`) — workflow, architecture, key rules
2. **`docs/` files** — detailed conventions (only when you need code examples)

---

## 1. Project Overview

<!-- CUSTOMIZE: Replace with your project description -->
Your Project Name — a TypeScript frontend application built with your framework.
Describe what the application does in 1-2 sentences.

### Prerequisites

<!-- CUSTOMIZE: Replace with your project's versions and tools -->
- **Node.js 20+**
- **pnpm 9+** (or npm/yarn — specify your package manager)
- **TypeScript 5.x** (strict mode enabled)

### Key Commands

<!-- CUSTOMIZE: Replace with your project's actual commands -->
```bash
pnpm install                  # Install dependencies
pnpm dev                      # Start dev server
pnpm build                    # Production build
pnpm test                     # Run all tests
pnpm test -- --watch          # Watch mode
pnpm lint                     # ESLint + Prettier check
pnpm lint:fix                 # Auto-fix lint issues
pnpm typecheck                # TypeScript type checking
```

### Directory Structure

<!-- CUSTOMIZE: Replace with your project's actual directory structure -->
```
src/
├── api/               # API client functions and types
├── components/        # Reusable UI components
│   ├── common/        # Shared components (Button, Input, Modal)
│   └── [feature]/     # Feature-specific components
├── hooks/             # Custom React hooks
├── pages/             # Page/route components
├── stores/            # State management (Zustand/Redux/Context)
├── types/             # Shared TypeScript types and interfaces
├── utils/             # Utility functions
└── __tests__/         # Test files (or colocated with source)

public/                # Static assets
```

### Architecture

<!-- CUSTOMIZE: Replace with your project's actual architecture -->
```
Page → Component → Hook → API Client → Backend
                     ↓
                   Store (state management)
```

<!-- CUSTOMIZE: Replace with your project's key patterns -->
- **Component pattern:** Functional components with hooks
- **State management:** Zustand / Redux Toolkit / React Context (pick one)
- **API layer:** React Query / SWR for server state
- **Routing:** React Router / Next.js file-based routing
- **Styling:** Tailwind CSS / CSS Modules / styled-components (pick one)

---

## 2. Code Conventions (Key Rules)

<!-- CUSTOMIZE: Replace with your project's TypeScript conventions -->
- **TypeScript strict mode** — no `any` unless absolutely necessary (document why)
- **Functional components** — no class components
- **Named exports** over default exports
- **Interface over type** for object shapes (unless unions/intersections needed)
- **Destructure props** in function signature
- **Use `const` assertions** for literal types
- **No inline styles** — use your styling solution
- **Absolute imports** via path aliases (`@/components/...`)
- **No barrel files** (`index.ts` re-exports) unless the directory is a public API

### Component Patterns

<!-- CUSTOMIZE: Replace with your project's component conventions -->
```
ComponentName/
├── ComponentName.tsx       # Component implementation
├── ComponentName.test.tsx  # Tests
├── ComponentName.module.css # Styles (if CSS Modules)
└── index.ts               # Public export (if used as a module boundary)
```

**Full rules with code examples:** [docs/code-best-practices.md](docs/code-best-practices.md)

---

## 3. Testing Rules (Key Rules)

<!-- CUSTOMIZE: Replace with your project's test conventions -->
- **Framework:** Vitest / Jest + React Testing Library
- **Test user behavior**, not implementation details
- **Use `userEvent`** over `fireEvent` for user interactions
- **Use MSW** (Mock Service Worker) for API mocking
- **No testing internal state** — test what the user sees
- **Naming:** `describe('ComponentName', () => { it('should do X when Y', ...) })`
- **Arrange-Act-Assert** pattern in every test
- **DO NOT** test styling or CSS classes
- **DO NOT** use `container.querySelector` — use Testing Library queries
- **DO** prefer `getByRole`, `getByLabelText` over `getByTestId`

**Full rules with code examples:** [docs/test-conventions.md](docs/test-conventions.md)

---

<!-- CUSTOMIZE: Add domain-specific sections as needed -->
<!-- Example: -->
<!-- ## 4. API Integration -->
<!-- - React Query for server state management -->
<!-- - Custom hooks wrapping API calls -->
<!-- - Error boundary for API failures -->

<!-- ## 5. State Management -->
<!-- - Zustand stores in `src/stores/` -->
<!-- - Server state in React Query, client state in Zustand -->

---

## 6. Agent Workflow

Follow this step-by-step process for every task:

### Step 1: Read

- Read this file for project context
- Read the relevant `docs/` files for detailed conventions

### Step 2: Find Similar Code

- Search for similar components, hooks, or patterns in the codebase
- Study how existing code handles the same concerns (data fetching, state, forms, tests)

### Step 3: Plan

- Identify which files to create or modify
- Consider component composition and hook extraction
- Plan test strategy before writing production code

### Step 4: Generate

- Follow existing patterns exactly (copy structure from similar components)
- Use the project's conventions for styling, state, and API calls
- Place code in the correct directory (see Section 1)

### Step 5: Test

- Write tests following [docs/test-conventions.md](docs/test-conventions.md)
<!-- CUSTOMIZE: Replace with your test command -->
- Run tests: `pnpm test -- ComponentName`
- Run type check: `pnpm typecheck`
- Fix any failures before proceeding

### Step 6: Commit

<!-- CUSTOMIZE: Replace with your commit format -->
- Format: `[TICKET-NUMBER] Description` (see [docs/git-conventions.md](docs/git-conventions.md))
- Run `pnpm lint:fix` before committing
- Do not use `--no-verify`

---

## 7. Git Conventions

<!-- CUSTOMIZE: Replace with your project's commit format -->
- **Commit format:** `[PROJECT-XXXX] Description`
- **Relaxed branches:** `hotfix*` and `chore*` — free-form messages allowed
- **All others:** ticket number required

**Full details:** [docs/git-conventions.md](docs/git-conventions.md)

---

## Quick Links

| Document | Content |
|---|---|
| [docs/code-best-practices.md](docs/code-best-practices.md) | TypeScript, component, hook, and styling conventions |
| [docs/test-conventions.md](docs/test-conventions.md) | Testing Library, MSW, test patterns |
| [docs/git-conventions.md](docs/git-conventions.md) | Commit format, branch naming |
