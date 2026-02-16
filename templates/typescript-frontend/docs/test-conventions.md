# Test Conventions

> Detailed testing rules. See [AGENTS.md](../AGENTS.md) for quick summaries.

---

## Framework

<!-- CUSTOMIZE: Replace with your project's test framework -->
- **Test runner:** Vitest / Jest
- **Component testing:** React Testing Library
- **User interactions:** `@testing-library/user-event`
- **API mocking:** MSW (Mock Service Worker)

---

## Test Naming

Use `describe` + `it` with descriptive names:

```typescript
describe("UserCard", () => {
  it("should display user name and email", () => { ... });
  it("should call onEdit when edit button is clicked", () => { ... });
  it("should show loading skeleton when data is pending", () => { ... });
});
```

---

## Arrange-Act-Assert Pattern

```typescript
it("should submit form with valid data", async () => {
  // Arrange
  const onSubmit = vi.fn();
  const user = userEvent.setup();
  render(<LoginForm onSubmit={onSubmit} />);

  // Act
  await user.type(screen.getByLabelText("Email"), "test@example.com");
  await user.type(screen.getByLabelText("Password"), "password123");
  await user.click(screen.getByRole("button", { name: "Sign In" }));

  // Assert
  expect(onSubmit).toHaveBeenCalledWith({
    email: "test@example.com",
    password: "password123",
  });
});
```

---

## Testing Library Queries

Prefer accessible queries in this order:

1. `getByRole` — best for most elements
2. `getByLabelText` — form inputs
3. `getByText` — non-interactive elements
4. `getByTestId` — **last resort only**

```typescript
// ✅ Good — accessible queries
screen.getByRole("button", { name: "Submit" });
screen.getByLabelText("Email address");
screen.getByText("Welcome back");

// ❌ Bad — implementation-dependent queries
container.querySelector(".submit-btn");
screen.getByTestId("submit-button");
```

---

## User Interactions

Always use `userEvent` over `fireEvent`:

```typescript
import userEvent from "@testing-library/user-event";

// ✅ Good — userEvent (simulates real user behavior)
const user = userEvent.setup();
await user.click(screen.getByRole("button"));
await user.type(screen.getByLabelText("Name"), "John");

// ❌ Bad — fireEvent (synthetic events, less realistic)
fireEvent.click(screen.getByRole("button"));
fireEvent.change(screen.getByLabelText("Name"), { target: { value: "John" } });
```

---

## API Mocking with MSW

<!-- CUSTOMIZE: Replace with your project's API mocking setup -->
```typescript
import { http, HttpResponse } from "msw";
import { setupServer } from "msw/node";

const server = setupServer(
  http.get("/api/users/:id", ({ params }) => {
    return HttpResponse.json({
      id: params.id,
      name: "Test User",
      email: "test@example.com",
    });
  })
);

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());

it("should display user data from API", async () => {
  render(<UserProfile userId="123" />);
  expect(await screen.findByText("Test User")).toBeInTheDocument();
});
```

### Override handlers for specific tests:

```typescript
it("should show error when API fails", async () => {
  server.use(
    http.get("/api/users/:id", () => {
      return new HttpResponse(null, { status: 500 });
    })
  );

  render(<UserProfile userId="123" />);
  expect(await screen.findByText("Something went wrong")).toBeInTheDocument();
});
```

---

## Async Testing

Use `findBy*` queries for async content:

```typescript
// ✅ Good — waits for element to appear
expect(await screen.findByText("User loaded")).toBeInTheDocument();

// ✅ Good — wait for element to disappear
await waitForElementToBeRemoved(() => screen.queryByText("Loading..."));

// ❌ Bad — no await, test is flaky
expect(screen.getByText("User loaded")).toBeInTheDocument();
```

---

## Anti-Patterns

1. **DO NOT** test implementation details (internal state, private methods)
2. **DO NOT** use `container.querySelector` — use Testing Library queries
3. **DO NOT** test styling or CSS classes
4. **DO NOT** use `fireEvent` when `userEvent` is available
5. **DO NOT** mock React hooks directly — mock the data layer instead
6. **DO** prefer integration tests (render full component tree) over unit tests of individual functions
7. **DO** test error states and loading states, not just happy paths
