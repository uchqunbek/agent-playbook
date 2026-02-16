# Test Conventions

> Detailed test writing rules for Cypress E2E. See [AGENTS.md](../AGENTS.md) for quick summaries.

---

## Test Structure

### File Organization

```typescript
// src/e2e/stms/login.spec.ts

describe('Login tests', () => {
  beforeEach(() => {
    cy.visit(Cypress.env('STMS_URL'));
    cy.isExpectedRoute('/signin');
  });

  it('ATC-416 Log in with valid credentials', () => {
    cy.findByLabelText('Username').type(Cypress.env('STMS_EMAIL'));
    cy.findByLabelText('Password').type(Cypress.env('STMS_PASSWORD'));
    cy.findByText('Log In').click();
    cy.isExpectedRoute('/orders');
  });

  it('ATC-422 Log in with invalid email', () => {
    cy.findByLabelText('Username').type('test@mail.com');
    cy.findByLabelText('Password').type(Cypress.env('STMS_PASSWORD'));
    cy.findByText('Log In').click();
    cy.findByRole('alert').should(
      'have.text',
      'Incorrect email or password. Please double-check and try again.',
    );
  });
});
```

### Rules

<!-- CUSTOMIZE: Replace test ID prefix with your project's convention -->
- **Test ID prefix:** `ATC-XXX` in test name, linked to test management tool
- **One `describe` per feature or page**
- **One `it` per test case** — test one behavior per block
- **`beforeEach` for common setup** — auth, navigation, data cleanup
- **`before` for one-time setup** — only when state must persist across tests

---

## Data Setup and Teardown

### API Setup (Preferred)

Set up test data via API calls, not UI interactions:

```typescript
describe('Order management', () => {
  let orderId: number;

  beforeEach(() => {
    cy.authSTMS();
    // Clean up any leftover test data
    cy.searchOrderAPI(testVin, 'vin').then((orders) => {
      const orderIds = orders.map((order) => order.id);
      cy.deleteOrderAPI(orderIds);
    });
  });

  it('ATC-90 Create and delete order', () => {
    const orderName = random.OrderName();
    cy.createOrderAPI(getOrderBody(orderName)).then((order) => {
      orderId = order.id;
      // ... test assertions
    });
  });
});
```

### Why API Over UI

| Approach | Speed | Reliability | Maintenance |
|----------|-------|-------------|-------------|
| API setup | Fast | High | Low (decoupled from UI) |
| UI setup | Slow | Flaky (UI changes break setup) | High |

---

## Waiting Strategies

### Use Assertions (Not `cy.wait()`)

```typescript
// ✅ Good — wait for element state
cy.findByRole('button', { name: 'Submit' }).should('be.enabled');
cy.findByText('Order created').should('be.visible');

// ✅ Good — wait for route
cy.isExpectedRoute('/orders');

// ✅ Good — wait for API response
cy.intercept('POST', '/api/orders').as('createOrder');
cy.findByRole('button', { name: 'Submit' }).click();
cy.wait('@createOrder');

// ❌ Bad — arbitrary wait
cy.wait(3000);
```

### Retry with `cypress-recurse`

For polling scenarios (e.g., waiting for email):

```typescript
import { recurse } from 'cypress-recurse';

recurse(
  () => cy.task('gmail:get-messages', { options }),
  (emails) => emails.length > 0,
  { limit: 5, delay: 4000, timeout: 20000 },
);
```

---

## Cross-Application Tests

<!-- CUSTOMIZE: Replace app names and auth commands -->
Tests spanning multiple apps (e.g., shipper creates order, carrier accepts):

```typescript
describe('Cross-app flow', () => {
  it('Shipper sends offer, carrier accepts', () => {
    // Shipper side
    cy.authSTMS();
    cy.createOrderAPI(orderBody).then((order) => {
      cy.sendOfferAPI(order);

      // Switch to carrier
      cy.authCTMS();
      cy.acceptOfferAPI(order.id);

      // Verify on shipper side
      cy.authSTMS();
      cy.visit(`${Cypress.env('STMS_URL')}/orders`);
      // ... assertions
    });
  });
});
```

---

## Anti-Patterns

| Anti-Pattern | Correct Approach |
|-------------|-----------------|
| `cy.wait(5000)` | Use assertions or `cy.intercept()` |
| `cy.get('.class-name')` | Use `findByRole`, `findByLabelText`, `findByText` |
| `.click({ force: true })` | Fix the element visibility issue |
| `.only()` in committed code | Remove before committing (ESLint enforces) |
| UI-based data setup | Use API custom commands |
| Hardcoded test data | Use `testutils/random.*` generators |
| Shared mutable state between tests | Independent test data per `it` block |
| `async/await` in test functions | Use Cypress command chaining |

---

## Spec File Naming

<!-- CUSTOMIZE: Replace with your project's naming convention -->
```
src/e2e/
├── stms/
│   ├── login.spec.ts              # Feature-based naming
│   ├── creation.spec.ts
│   └── offers.spec.ts
├── ctms/
│   ├── 001_login.spec.ts          # Numbered for execution order
│   ├── 005_loads.spec.ts
│   └── 015_offers.spec.ts
```

Use feature-based names for new specs. Numbered prefixes are a legacy convention.
