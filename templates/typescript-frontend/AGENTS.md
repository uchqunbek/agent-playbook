# Agents Guide

> Comprehensive guide for AI coding agents working on this codebase.
> For detailed rules with code examples, see the `docs/` directory.

---

## 0. Navigation Contract

Agents MUST traverse context in this order:

1. **This file** (`AGENTS.md`) — workflow, architecture, key rules
2. **`docs/` files** — detailed conventions (only when you need code examples)

---

## 1. Project Overview

<!-- CUSTOMIZE: Replace with your project description -->
Your Project Name — a TypeScript SPA serving your domain.
Describe what the application does in 1-2 sentences.

### Prerequisites

<!-- CUSTOMIZE: Replace with your project's versions and tools -->
- **Node.js 22+** (pinned via `.nvmrc`)
- **pnpm 9+** (enforced — no npm/yarn)
- **TypeScript 5.x** (strict mode enabled)

### Key Commands

<!-- CUSTOMIZE: Replace with your project's actual commands -->
```bash
pnpm dev                      # Start dev server (Vite)
pnpm test                     # Run Vitest
pnpm check                    # ESLint + Prettier (pre-commit hook)
pnpm check:types              # TypeScript type checking (pre-push hook)
```

### Directory Structure

<!-- CUSTOMIZE: Replace with your project's actual directory structure -->
```
src/
├── core/                     # Shared utilities, providers, API client
│   ├── api/                  # requestCarrierAPI, query helpers
│   ├── context/              # App-wide context providers
│   └── theme/                # MUI theme overrides, ColorDynamic tokens
├── drivers/                  # Feature module (example)
│   ├── core/                 # Shared hooks, types for this feature
│   ├── data/                 # API hooks, DTOs, Yup schemas
│   ├── list/                 # DriverListPage, DriverListTable
│   ├── detail/               # DriverDetailPage, DriverCard
│   └── __tests__/            # All tests for this feature
├── loads/                    # Another feature module
├── shared/                   # Cross-feature shared components
└── App.tsx                   # Route composition
```

### Architecture

<!-- CUSTOMIZE: Replace with your project's key patterns -->
```
Route → Page → Component → Hook → API Client → Backend
                 ↓              ↓
              Formik         React Query cache
```

- **Components:** `function` keyword, named exports, `styled-components`
- **State:** React Query (server) + URL params + Context (client)
- **API:** `useAPIQuery` / `useAPIMutation` wrapping `requestCarrierAPI`
- **Forms:** Formik + Yup via `@superdispatch/forms`
- **UI:** MUI v4 + `@superdispatch/ui` layout primitives
- **Routing:** `createBrowserRouter` with feature-based `RouteObject[]`

---

## 2. Code Conventions (Key Rules)

<!-- CUSTOMIZE: Replace with your project's conventions -->
- **`function` keyword** for components — no arrow-function components
- **Named exports only** — no default exports
- **`interface` over `type`** for object shapes (use `type` for unions only)
- **Boolean prefixes:** `is`, `has`, `should`, `can`, `did`, `will`, `does`
- **No enums** — use string union types
- **No inline styles or `className`** — use `styled()` from `styled-components`
- **ColorDynamic tokens** — never use raw hex/rgb colors
- **File ordering:** imports → interface → styled → component → helpers

```typescript
// ✅ function keyword + named export
export function DriverCard({ driver }: DriverCardProps) { ... }

// ❌ arrow function component
export const DriverCard = ({ driver }: DriverCardProps) => { ... }
```

**Full rules with code examples:** [docs/code-best-practices.md](docs/code-best-practices.md)

---

## 3. Testing Rules (Key Rules)

- **Framework:** Vitest + jsdom + React Testing Library
- **File location:** `__tests__/*.spec.tsx` inside each feature
- **Render helper:** `renderWithProviders()` — wraps QueryClient + Theme + Router
- **API mocking:** MSW **v1** — use `rest.get()`, `rest.post()` (**NOT** `http.get`)
- **Prefer `getByRole`** and `getByLabelText` over `getByTestId`
- **`userEvent.setup()`** for interactions — never `fireEvent`
- **Arrange-Act-Assert** pattern in every test

**Full rules with code examples:** [docs/test-conventions.md](docs/test-conventions.md)

---

## 4. API & State Management (Key Rules)

### State Hierarchy

When deciding where state lives, follow this priority (highest → lowest):

1. **URL search params** — filters, pagination, selected IDs (`useLocationParams`)
2. **React Query cache** — server data (`useAPIQuery`, `useAPIListQuery`)
3. **React Context** — shared UI state across a subtree (`useNullableContext`)
4. **Zustand/Store** — rare, app-wide client state
5. **`useState`** — component-local UI state
6. **Feature flags** — runtime toggles from config

### API Hooks

<!-- CUSTOMIZE: Replace with your project's API hook names -->
```typescript
useAPIQuery(['drivers', guid], () => requestCarrierAPI('GET /drivers/{guid}', { guid }));
useAPIListQuery(['drivers', filters], (page) => requestCarrierAPI('GET /drivers{?page,size}', { ...filters, page }));
useAPIMutation(() => requestCarrierAPI('PUT /drivers/{guid}', { guid, ...values }));
```

**Full rules with code examples:** [docs/code-best-practices.md](docs/code-best-practices.md)

---

## 5. Forms & Validation (Key Rules)

<!-- CUSTOMIZE: Replace with your project's form patterns -->
- **`useAppFormik`** — project wrapper around `useFormik` with typed values
- **`FormikDrawer`** — standard drawer form pattern (open/close/submit)
- **`@superdispatch/forms`** — `FormikTextField`, `FormikDateField`, `FormikPhoneField`
- **Yup schemas** — define DTO shape, then `type DriverDTO = InferType<typeof driverSchema>`
- **Custom Yup helpers** — `yupPhone()`, `yupEnum()` from shared utils

```typescript
const driverSchema = yup.object({
  name: yup.string().required(),
  phone: yupPhone().required(),
  email: yup.string().email().nullable(),
});
type DriverDTO = InferType<typeof driverSchema>;
```

**Full rules with code examples:** [docs/code-best-practices.md](docs/code-best-practices.md)

---

## 6. Routing (Key Rules)

<!-- CUSTOMIZE: Replace with your project's routing pattern -->
- **`createBrowserRouter`** in `App.tsx` — single entry point
- **Feature modules export `RouteObject[]`** — composed via spread in the app router
- **`useLocationParams`** for URL-driven state (filters, pagination, selected IDs)

```typescript
// drivers/routes.ts
export const driverRoutes: RouteObject[] = [
  { path: 'drivers', element: <DriversPage /> },
  { path: 'drivers/:guid', element: <DriverDetailPage /> },
];

// App.tsx — compose all feature routes
const router = createBrowserRouter([
  { path: '/', element: <Layout />, children: [...driverRoutes, ...loadRoutes] },
]);
```

---

## 7. Agent Workflow

1. **Read** — this file for context, then relevant `docs/` files for code examples
2. **Find Similar Code** — search for similar components, hooks, or API patterns in the codebase
3. **Plan** — identify files to create/modify, decide where state lives (Section 4), plan tests
4. **Generate** — follow existing patterns exactly; `function` keyword, named exports, `styled()`
5. **Test** — use `renderWithProviders()` + MSW **v1** (`rest.get`, not `http.get`)
<!-- CUSTOMIZE: Replace with your test command -->
   - Run: `pnpm test -- DriverCard`
6. **Lint** — run `pnpm check` + `pnpm check:types`, fix all errors
<!-- CUSTOMIZE: Replace with your lint commands -->
7. **Commit** — `[TICKET-NUMBER] Description` format, never `--no-verify`
<!-- CUSTOMIZE: Replace with your commit format -->
   - See [docs/git-conventions.md](docs/git-conventions.md)

---

## 8. Git Conventions

<!-- CUSTOMIZE: Replace with your project's commit format -->
- **Format:** `[TICKET-NUMBER] Description`
- **Relaxed branches:** `hotfix*` and `chore*` — free-form messages allowed
- **All others:** ticket number required (enforced by Husky hook)

```
# Good
[TMS-1234] Add driver detail page with card layout
[TMS-567] Fix phone field validation in driver form

# Bad
fix bug                    # Missing ticket
[TMS-123] updated code     # Vague description
```

**Full details:** [docs/git-conventions.md](docs/git-conventions.md)

---

## Quick Links

| Document | Content |
|---|---|
| [docs/code-best-practices.md](docs/code-best-practices.md) | Components, styling, API layer, state, forms, routing, packages |
| [docs/test-conventions.md](docs/test-conventions.md) | Vitest, renderWithProviders, MSW v1, mock factories, forms |
| [docs/git-conventions.md](docs/git-conventions.md) | Commit format, Husky hooks, branch naming, PR checklist |
