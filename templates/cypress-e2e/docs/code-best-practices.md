# Code Best Practices

> Detailed code conventions for Cypress E2E tests. See [AGENTS.md](../AGENTS.md) for quick summaries.

---

## Page Objects

### Structure

One class per page. Export both the class and a singleton instance:

```typescript
// support/po/STMS/ordersListPage.ts

export class STMSOrdersListPage {
  // 1. Element getters (return Chainable)
  orderCheckbox(orderNumber: string): Cypress.Chainable<JQuery<HTMLElement>> {
    return cy
      .findByLabelText(`order-number-${orderNumber}`)
      .find('input[type="checkbox"]');
  }

  submitButton(): Cypress.Chainable<JQuery<HTMLElement>> {
    return cy.findByRole('button', { name: 'Submit' });
  }

  // 2. Actions (return void)
  visit(): void {
    cy.visit(Cypress.env('STMS_URL'));
    cy.isExpectedRoute('/orders');
  }

  selectOrder(orderNumber: string): void {
    this.orderCheckbox(orderNumber).check();
  }

  // 3. Assertions (return void)
  expectOrderSelected(orderNumber: string): void {
    this.orderCheckbox(orderNumber).should('be.checked');
  }
}

export const stmsOrdersListPage = new STMSOrdersListPage();
```

### Rules

<!-- CUSTOMIZE: Replace app abbreviations with your project's -->
- File naming: `camelCase` matching page name (e.g., `ordersListPage.ts`)
- Organize by app: `po/STMS/`, `po/CTMS/`, `po/CP/`
- Use `@testing-library/cypress` queries inside page objects
- Keep assertions in page objects when they describe page-specific state
- Use `.within()` to scope queries to a container

---

## Custom Commands

### Auth Commands

<!-- CUSTOMIZE: Replace with your project's auth commands -->
Auth via API, not UI — faster and more reliable:

```typescript
// support/commands/auth.ts
Cypress.Commands.add('authSTMS', (email = Cypress.env('STMS_EMAIL'), password = Cypress.env('STMS_PASSWORD')) => {
  cy.request({
    method: 'POST',
    url: `${Cypress.env('STMS_API_URL')}/auth/login`,
    body: { username: email, password },
  }).then((response) => {
    const token: string = response.body.data.object.token;
    Cypress.env('STMS_BEARER_TOKEN', token);
    window.localStorage.setItem('token', token);
  });
});
```

### CRUD Commands

API commands for test data setup/teardown:

```typescript
// Usage in tests
cy.createOrderAPI(orderBody);       // Create via API
cy.deleteOrderAPI(orderId);         // Cleanup via API
cy.searchOrderAPI(vin, 'vin');      // Search via API
```

### Type Declarations

All custom commands must be declared in `support/index.ts`:

```typescript
declare global {
  namespace Cypress {
    interface Chainable {
      authSTMS(email?: string, password?: string): void;
      createOrderAPI(orderBody?: order): Chainable<any>;
      deleteOrderAPI(orderId: number | number[]): void;
    }
  }
}
```

---

## Selectors

### Priority Order

1. `cy.findByRole()` — buttons, links, headings, textboxes
2. `cy.findByLabelText()` — form inputs with labels
3. `cy.findByText()` — visible text content
4. `cy.findByTestId()` — **last resort** only

```typescript
// ✅ Good — accessible queries
cy.findByRole('button', { name: 'Create Order' }).click();
cy.findByLabelText('Username').type(email);
cy.findByText('Log In').click();

// ❌ Bad — CSS selectors
cy.get('.btn-primary').click();
cy.get('#username-input').type(email);
```

### Scoping with `.within()`

```typescript
cy.findByLabelText('pickup counterparty').within(() => {
  cy.findByLabelText('Address').type('123 Main St');
  cy.findByLabelText('ZIP Code').type('10001');
});
```

---

## Test Data Generation

<!-- CUSTOMIZE: Replace with your project's random utilities -->
Use `testutils/random.*` for dynamic test data:

```typescript
import { random } from '../../testutils/random';

const orderName = random.OrderName();       // "cy.order.4821"
const price = random.Price(150, 999);       // Random number 150-999
const address = random.Address();           // Random US address
const vin = random.String(17);             // Random 17-char string
const email = random.GenerateEmail('test', 'example.com');
```

---

## TypeScript

- Strict mode enabled
- Use proper types for Cypress chainables: `Cypress.Chainable<JQuery<HTMLElement>>`
- Import types explicitly: `import { type order } from '../../testutils/types'`
- Avoid `any` — use specific types from `testutils/types.ts`

---

## ESLint Rules

<!-- CUSTOMIZE: Replace with your project's ESLint rules if different -->
| Rule | Level | Purpose |
|------|-------|---------|
| `cypress/no-unnecessary-waiting` | error | No `cy.wait(number)` |
| `cypress/no-async-tests` | error | No async test functions |
| `cypress/no-pause` | error | No `cy.pause()` in committed code |
| `cypress/no-force` | warn | Avoid `.click({ force: true })` |
| `no-only-tests/no-only-tests` | error | No `.only()` in committed code |
| `cypress/unsafe-to-chain-command` | warn | Avoid unsafe command chaining |
