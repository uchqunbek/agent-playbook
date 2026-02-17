# Code Best Practices

> Detailed TypeScript/React conventions with examples. See [AGENTS.md](../AGENTS.md) for quick summaries.

---

## Component Patterns

### `function` Keyword Components

Always use the `function` keyword. Never use arrow functions:

```typescript
// ✅ Good — function keyword + named export
interface DriverCardProps {
  driver: DriverDTO;
  onEdit: (guid: string) => void;
}

export function DriverCard({ driver, onEdit }: DriverCardProps) {
  return (
    <CardContainer>
      <Typography variant="h6">{driver.name}</Typography>
      <Button onClick={() => onEdit(driver.guid)}>Edit</Button>
    </CardContainer>
  );
}

// ❌ Bad — arrow function component
export const DriverCard = ({ driver }: DriverCardProps) => { ... };
```

### Props & Booleans

- Define `interface` (not `type`) for props, destructure in signature
- Boolean props use prefixes: `is`, `has`, `should`, `can`, `did`, `will`, `does`

```typescript
interface DriverRowProps {
  driver: DriverDTO;
  isSelected: boolean;
  hasPermission: boolean;
  onSelect: (guid: string) => void;
}
```

### File Ordering

Every component file: **imports → interface → styled → component → helpers**

```typescript
import { ColorDynamic } from '@superdispatch/ui';        // 1. Imports
import styled from 'styled-components';

interface DriverCardProps { driver: DriverDTO; }          // 2. Props interface

const CardContainer = styled.div`                         // 3. Styled components
  border: 1px solid ${ColorDynamic.Silver400};
`;

export function DriverCard({ driver }: DriverCardProps) { // 4. Exported component
  return <CardContainer>{formatName(driver)}</CardContainer>;
}

function formatName(d: DriverDTO): string {               // 5. Helpers (private)
  return `${d.firstName} ${d.lastName}`;
}
```

---

## File Structure & Naming

### Feature Module Structure

<!-- CUSTOMIZE: Replace with your project's feature structure -->
```
src/drivers/
├── core/                # Shared: DriverDTO.ts, useDriverPermissions.ts
├── data/                # DriversAPI.ts (hooks), DriversService.ts (requests)
├── list/                # DriversPage.tsx, DriversTable.tsx
├── detail/              # DriverDetailPage.tsx, DriverCard.tsx
├── __tests__/           # DriversPage.spec.tsx, DriverCard.spec.tsx
└── routes.ts            # RouteObject[] for this feature
```

### Naming Rules

- **Components:** `PascalCase.tsx` — `DriverCard.tsx`
- **Hooks:** `camelCase.ts` with `use` prefix — `useDriverPermissions.ts`
- **DTOs:** `PascalCase.ts` — `DriverDTO.ts`
- **Tests:** `PascalCase.spec.tsx` inside `__tests__/`

---

## Styling

### styled-components + ColorDynamic

Use `styled()` from `styled-components`. Never use inline styles, `className`, or raw colors:

```typescript
import styled from 'styled-components';
import { ColorDynamic } from '@superdispatch/ui';

// ✅ Good — styled component with ColorDynamic tokens
const StatusBadge = styled.span<{ isActive: boolean }>`
  color: ${({ isActive }) => (isActive ? ColorDynamic.Green300 : ColorDynamic.Silver500)};
  background: ${ColorDynamic.White};
`;

// ❌ Bad — inline styles, className, or raw hex colors
<span style={{ color: '#4caf50' }}>Active</span>
<span className="status-badge">Active</span>
```

### MUI v4 + @superdispatch/ui Layout

<!-- CUSTOMIZE: Replace with your project's UI library versions -->
Use layout primitives from `@superdispatch/ui`: `Stack`, `Columns`, `Column`, `PageLayout`.

```typescript
import { Stack, Columns, Column, PageLayout } from '@superdispatch/ui';

export function DriversPage() {
  return (
    <PageLayout>
      <Stack space={2}>
        <DriversTable />
      </Stack>
    </PageLayout>
  );
}
```

---

## API Layer

### requestCarrierAPI

<!-- CUSTOMIZE: Replace with your project's API client name -->
The API client uses URI template syntax:

```typescript
requestCarrierAPI('GET /drivers/{guid}', { guid });                          // path params
requestCarrierAPI('GET /drivers{?page,size,status}', { page: 1, size: 20 }); // query params
requestCarrierAPI('PUT /drivers/{guid}', { guid, ...updateData });           // body
```

### API Hooks

```typescript
// useAPIQuery — single resource
export function useDriver(guid: string) {
  return useAPIQuery(['drivers', guid], () =>
    requestCarrierAPI('GET /drivers/{guid}', { guid }),
  );
}

// useAPIListQuery — paginated list
export function useDrivers(filters: DriverFilters) {
  return useAPIListQuery(['drivers', filters], (page) =>
    requestCarrierAPI('GET /drivers{?page,size,status}', { ...filters, page }),
  );
}

// useAPIMutation — create/update/delete
export function useUpdateDriver(guid: string) {
  const queryClient = useQueryClient();
  return useAPIMutation(
    (values: DriverDTO) => requestCarrierAPI('PUT /drivers/{guid}', { guid, ...values }),
    { onSuccess: () => queryClient.invalidateQueries(['drivers']) },
  );
}
```

### Query Keys

Keys: `['drivers']` (list), `['drivers', guid]` (detail), `['drivers', { status }]` (filtered).

---

## State Management

Use the highest-priority option that fits:

**1. URL Search Params** — filters, pagination, selected IDs:
```typescript
const [params, setParams] = useLocationParams({ page: 1, status: 'all' });
```

**2. React Query Cache** — all server data:
```typescript
const { data: driver } = useAPIQuery(['drivers', guid], fetchDriver);
```

**3. React Context** — shared UI state across a subtree:
```typescript
const DrawerContext = createContext<DrawerState | null>(null);
export const useDrawerContext = () => useNullableContext(DrawerContext, 'DrawerProvider');
```

**4. useState** — component-local UI state:
```typescript
const [isDrawerOpen, setIsDrawerOpen] = useState(false);
```

---

## Forms

### useAppFormik + FormikDrawer

<!-- CUSTOMIZE: Replace with your project's form setup -->
```typescript
export function DriverFormDrawer({ open, onClose, driver }: DriverFormDrawerProps) {
  const { mutate: updateDriver } = useUpdateDriver(driver.guid);
  const formik = useAppFormik<DriverDTO>({
    initialValues: toDriverDTO(driver),
    validationSchema: driverSchema,
    onSubmit: (values) => updateDriver(values, { onSuccess: onClose }),
  });

  return (
    <FormikDrawer open={open} onClose={onClose} formik={formik}>
      <FormikDrawerContent title="Edit Driver">
        <FormikTextField name="name" label="Full Name" fullWidth />
        <FormikPhoneField name="phone" label="Phone Number" fullWidth />
      </FormikDrawerContent>
    </FormikDrawer>
  );
}
```

### DTO with Yup Schema + InferType

```typescript
// drivers/core/DriverDTO.ts
export const driverSchema = yup.object({
  name: yup.string().required('Name is required'),
  phone: yupPhone().required('Phone is required'),
  email: yup.string().email('Invalid email').nullable(),
  isActive: yup.boolean().default(true),
});
export type DriverDTO = InferType<typeof driverSchema>;
```

---

## Routing

<!-- CUSTOMIZE: Replace with your project's router setup -->
```typescript
// App.tsx — compose feature routes via spread
const router = createBrowserRouter([
  { path: '/', element: <AppLayout />, children: [...driverRoutes, ...loadRoutes] },
]);

// drivers/routes.ts — each feature exports RouteObject[]
export const driverRoutes: RouteObject[] = [
  { path: 'drivers', element: <DriversPage /> },
  { path: 'drivers/:guid', element: <DriverDetailPage /> },
];

// useLocationParams for URL-driven state
const [params, setParams] = useLocationParams({ page: 1, size: 20, status: 'all' });
```

---

## @superdispatch Packages

<!-- CUSTOMIZE: Replace with your project's package list -->
| Package | Purpose |
|---|---|
| `@superdispatch/ui` | Layout (Stack, Columns, PageLayout), ColorDynamic tokens, FormikDrawer |
| `@superdispatch/forms` | FormikTextField, FormikDateField, FormikPhoneField, FormikCheckboxField |
| `@superdispatch/dates` | Date formatting utilities, date range pickers |
| `@superdispatch/phones` | Phone formatting, validation helpers |
| `@superdispatch/hooks` | useNullableContext, useLocationParams |

---

## Anti-Patterns

| Anti-Pattern | Correct Approach |
|---|---|
| Arrow function components | `function` keyword with named export |
| Default exports | Named exports only |
| `className` or inline styles | `styled()` from `styled-components` |
| Raw hex/rgb colors | `ColorDynamic` tokens from `@superdispatch/ui` |
| `enum` keyword | String union types: `type Status = 'active' \| 'inactive'` |
| `any` type | Explicit types; document with eslint-disable if unavoidable |
| `type` for object shapes | `interface` (use `type` only for unions/intersections) |
| Cross-feature imports | Move shared code to `shared/` or `core/` |
| `React.FC` / `React.FunctionComponent` | Explicit return types or implicit inference |
