# Test Conventions

> Detailed testing rules. See [AGENTS.md](../AGENTS.md) for quick summaries.

---

## Framework & Setup

<!-- CUSTOMIZE: Replace with your project's test framework -->
- **Test runner:** Vitest with `jsdom` environment and `globals: true`
- **Component testing:** React Testing Library
- **User interactions:** `@testing-library/user-event`
- **API mocking:** MSW **v1** (Mock Service Worker)
- **Setup file:** `setupTests.ts` — configures MSW, cleanup, custom matchers

---

## Test File Organization

Tests live in `__tests__/` inside each feature directory:

<!-- CUSTOMIZE: Replace with your project's test structure -->
```
src/drivers/
├── __tests__/
│   ├── DriversPage.spec.tsx     # Page-level integration tests
│   ├── DriverCard.spec.tsx      # Component tests
│   └── DriverFormDrawer.spec.tsx
├── testutils/                    # Feature-specific test helpers
│   ├── mockDrivers.ts           # Mock factory functions
│   └── driverMswHandlers.ts     # MSW handlers for this feature
└── ...
```

---

## renderWithProviders

<!-- CUSTOMIZE: Replace with your project's test render helper -->
Always use `renderWithProviders()` instead of bare `render()`. It wraps components with QueryClient, Theme, and Router:

```typescript
import { renderWithProviders } from 'shared/testutils/renderWithProviders';

it('should display driver name', async () => {
  renderWithProviders(<DriverCard driver={mockDriver} />);
  expect(screen.getByText('John Doe')).toBeInTheDocument();
});

// With routing context (initialEntries for URL params)
it('should render driver detail page', async () => {
  renderWithProviders(<DriverDetailPage />, {
    initialEntries: ['/drivers/abc-123'],
  });
  expect(await screen.findByText('John Doe')).toBeInTheDocument();
});
```

What `renderWithProviders` wraps:
- Fresh `QueryClient` (prevents cache leakage between tests)
- `ThemeProvider` with the app theme
- `MemoryRouter` with optional `initialEntries`

---

## Test Naming & Structure

### describe/it Pattern

```typescript
describe('DriverCard', () => {
  it('should display driver name and phone number', () => { ... });
  it('should call onEdit when edit button is clicked', () => { ... });
  it('should show inactive badge when driver is not active', () => { ... });
});
```

### Arrange-Act-Assert

```typescript
it('should update driver name on form submit', async () => {
  // Arrange
  const user = userEvent.setup();
  renderWithProviders(<DriverFormDrawer open={true} driver={mockDriver} onClose={vi.fn()} />);

  // Act
  await user.clear(screen.getByLabelText('Full Name'));
  await user.type(screen.getByLabelText('Full Name'), 'Jane Doe');
  await user.click(screen.getByRole('button', { name: 'Save' }));

  // Assert
  await waitFor(() => {
    expect(screen.queryByRole('dialog')).not.toBeInTheDocument();
  });
});
```

---

## Testing Library Queries

Prefer accessible queries in this priority order:

1. **`getByRole`** — best for most elements (`button`, `textbox`, `heading`)
2. **`getByLabelText`** — form inputs
3. **`getByText`** — non-interactive text content
4. **`getByTestId`** — **last resort only**

```typescript
// ✅ Good — accessible queries
screen.getByRole('button', { name: 'Save' });
screen.getByLabelText('Full Name');
screen.getByRole('heading', { name: 'Edit Driver' });

// ❌ Bad — implementation-dependent queries
container.querySelector('.save-btn');
screen.getByTestId('save-button');
```

---

## User Interactions

Always use `userEvent.setup()`. Never use `fireEvent`:

```typescript
import userEvent from '@testing-library/user-event';

// ✅ Good — userEvent (simulates real user behavior)
const user = userEvent.setup();
await user.click(screen.getByRole('button', { name: 'Edit' }));
await user.type(screen.getByLabelText('Name'), 'John');
await user.selectOptions(screen.getByRole('combobox'), 'active');

// ❌ Bad — fireEvent (synthetic events, less realistic)
fireEvent.click(screen.getByRole('button'));
fireEvent.change(input, { target: { value: 'John' } });
```

---

## MSW v1 Mocking

> **CRITICAL:** This project uses MSW **v1**. Use `rest.get()` / `rest.post()` syntax.
> Do **NOT** use MSW v2 syntax (`http.get`, `HttpResponse.json`).

### Setup

<!-- CUSTOMIZE: Replace with your project's MSW setup -->
```typescript
import { rest } from 'msw';
import { setupServer } from 'msw/node';

// ✅ Correct — MSW v1 syntax
const mockServer = setupServer(
  rest.get('/api/drivers/:guid', (req, res, ctx) => {
    return res(
      ctx.status(200),
      ctx.json({
        data: {
          guid: req.params.guid,
          name: 'John Doe',
          phone: '+1234567890',
          is_active: true,
        },
      }),
    );
  }),
);

// ❌ WRONG — MSW v2 syntax (will NOT work)
// http.get('/api/drivers/:guid', () => HttpResponse.json({ ... }))

beforeAll(() => mockServer.listen());
afterEach(() => mockServer.resetHandlers());
afterAll(() => mockServer.close());
```

### API Response Shape

Match the actual API response format used by `requestCarrierAPI`:

```typescript
// Single resource — wrap in { data: ... }
rest.get('/api/drivers/:guid', (req, res, ctx) => {
  return res(ctx.json({ data: createMockDriver({ guid: req.params.guid as string }) }));
});

// Paginated list — { data: [...], pagination: { ... } }
rest.get('/api/drivers', (req, res, ctx) => {
  return res(ctx.json({
    data: [createMockDriver(), createMockDriver({ name: 'Jane Doe' })],
    pagination: { page: 1, size: 20, total: 2 },
  }));
});
```

### Error Overrides

Override handlers for specific error test cases:

```typescript
it('should show error when API returns 500', async () => {
  mockServer.use(
    rest.get('/api/drivers/:guid', (req, res, ctx) => {
      return res(ctx.status(500), ctx.json({ message: 'Internal Server Error' }));
    }),
  );

  renderWithProviders(<DriverDetailPage />, { initialEntries: ['/drivers/abc-123'] });
  expect(await screen.findByText(/something went wrong/i)).toBeInTheDocument();
});
```

---

## Mock Factories

Create per-feature mock factory functions in `testutils/`:

```typescript
// drivers/testutils/mockDrivers.ts
interface MockDriverOverrides {
  guid?: string;
  name?: string;
  phone?: string;
  isActive?: boolean;
}

export function createMockDriver(overrides: MockDriverOverrides = {}): DriverDTO {
  return {
    guid: 'driver-guid-001',
    name: 'John Doe',
    phone: '+1234567890',
    email: 'john@example.com',
    isActive: true,
    ...overrides,
  };
}
```

---

## Async Testing

Use `findBy*` queries for content that appears after async operations:

```typescript
// ✅ Good — findBy waits for element to appear
expect(await screen.findByText('John Doe')).toBeInTheDocument();

// ✅ Good — waitFor for complex assertions
await waitFor(() => {
  expect(screen.getAllByRole('row')).toHaveLength(3);
});

// ✅ Good — wait for loading to resolve
await waitForElementToBeRemoved(() => screen.queryByText('Loading...'));

// ❌ Bad — no await, test will be flaky
expect(screen.getByText('John Doe')).toBeInTheDocument();
```

---

## Testing Forms

Test form submission flow with `userEvent`:

```typescript
it('should submit driver form with valid data', async () => {
  const onClose = vi.fn();
  const user = userEvent.setup();
  renderWithProviders(
    <DriverFormDrawer open={true} driver={createMockDriver()} onClose={onClose} />,
  );

  await user.clear(screen.getByLabelText('Full Name'));
  await user.type(screen.getByLabelText('Full Name'), 'Updated Name');
  await user.click(screen.getByRole('button', { name: 'Save' }));

  await waitFor(() => expect(onClose).toHaveBeenCalled());
});
```

---

## Anti-Patterns

| Anti-Pattern | Correct Approach |
|---|---|
| MSW v2 syntax (`http.get`, `HttpResponse`) | MSW v1: `rest.get()`, `res(ctx.json())` |
| Bare `render()` without providers | `renderWithProviders()` |
| Mocking React hooks (`vi.mock(useDriver)`) | Mock the API layer with MSW |
| `container.querySelector()` | Use Testing Library queries (`getByRole`, etc.) |
| `fireEvent` for user interactions | `userEvent.setup()` + `user.click()` / `user.type()` |
| Hardcoded mock data inline | Mock factory functions (`createMockDriver()`) |
| Testing internal state or implementation | Test what the user sees via rendered output |
| Missing `await` on async queries | Use `findBy*` or `waitFor()` for async content |
