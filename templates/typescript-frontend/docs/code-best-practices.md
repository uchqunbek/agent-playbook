# Code Best Practices

> Detailed TypeScript/React conventions with examples. See [AGENTS.md](../AGENTS.md) for quick summaries.

---

## TypeScript Rules

### Strict Mode

TypeScript strict mode is enabled. Avoid `any`:

```typescript
// ✅ Good — explicit types
function getUser(id: string): Promise<User> {
  return apiClient.get(`/users/${id}`);
}

// ❌ Bad — any type
function getUser(id: any): Promise<any> {
  return apiClient.get(`/users/${id}`);
}
```

If `any` is truly unavoidable, document why:

```typescript
// eslint-disable-next-line @typescript-eslint/no-explicit-any -- third-party library returns untyped data
const rawData = externalLib.getData() as any;
```

### Interface vs Type

Use `interface` for object shapes, `type` for unions/intersections:

```typescript
// ✅ Interface for object shapes
interface User {
  id: string;
  name: string;
  email: string;
}

// ✅ Type for unions
type Status = "idle" | "loading" | "success" | "error";

// ✅ Type for intersections
type AdminUser = User & { permissions: string[] };
```

### Named Exports

```typescript
// ✅ Good — named export
export function UserProfile({ user }: UserProfileProps) { ... }

// ❌ Bad — default export
export default function UserProfile({ user }: UserProfileProps) { ... }
```

---

## Component Patterns

### Functional Components Only

```typescript
// ✅ Good — functional component with destructured props
interface UserCardProps {
  user: User;
  onEdit: (id: string) => void;
}

export function UserCard({ user, onEdit }: UserCardProps) {
  return (
    <div>
      <h2>{user.name}</h2>
      <button onClick={() => onEdit(user.id)}>Edit</button>
    </div>
  );
}

// ❌ Bad — class component
class UserCard extends React.Component { ... }
```

### Component File Structure

<!-- CUSTOMIZE: Replace with your project's component structure -->
```
UserCard/
├── UserCard.tsx           # Component implementation
├── UserCard.test.tsx      # Tests
└── UserCard.module.css    # Styles (if CSS Modules)
```

### Prop Types

Always define prop interfaces. Destructure in the function signature:

```typescript
interface DialogProps {
  isOpen: boolean;
  title: string;
  onClose: () => void;
  children: React.ReactNode;
}

export function Dialog({ isOpen, title, onClose, children }: DialogProps) {
  if (!isOpen) return null;
  // ...
}
```

---

## Hook Patterns

### Custom Hooks

Extract reusable logic into custom hooks:

```typescript
// ✅ Good — custom hook for data fetching
export function useUser(id: string) {
  return useQuery({
    queryKey: ["user", id],
    queryFn: () => fetchUser(id),
  });
}

// Usage
function UserProfile({ userId }: { userId: string }) {
  const { data: user, isLoading, error } = useUser(userId);
  // ...
}
```

### Hook Rules

- Prefix with `use`
- Return typed values
- Handle loading/error states
- Keep hooks focused on a single concern

---

## API Layer

<!-- CUSTOMIZE: Replace with your project's API patterns (React Query, SWR, etc.) -->
```typescript
// api/users.ts — API client functions
export async function fetchUser(id: string): Promise<User> {
  const response = await apiClient.get(`/users/${id}`);
  return response.data;
}

export async function updateUser(id: string, data: UpdateUserRequest): Promise<User> {
  const response = await apiClient.put(`/users/${id}`, data);
  return response.data;
}
```

---

## State Management

<!-- CUSTOMIZE: Replace with your project's state management pattern -->
Keep server state and client state separate:

- **Server state:** React Query / SWR (data from API)
- **Client state:** Zustand / Context (UI state, forms, preferences)

```typescript
// ✅ Good — server state via React Query
const { data: users } = useQuery({ queryKey: ["users"], queryFn: fetchUsers });

// ✅ Good — client state via Zustand
const isMenuOpen = useAppStore((state) => state.isMenuOpen);
```

---

## Imports

Use absolute imports via path aliases:

```typescript
// ✅ Good — absolute import
import { UserCard } from "@/components/UserCard";
import { useUser } from "@/hooks/useUser";

// ❌ Bad — relative import climbing up directories
import { UserCard } from "../../../components/UserCard";
```

---

## Error Handling

```typescript
// ✅ Good — error boundary for component tree errors
<ErrorBoundary fallback={<ErrorPage />}>
  <UserProfile />
</ErrorBoundary>

// ✅ Good — handling async errors
const { error } = useUser(userId);
if (error) return <ErrorMessage error={error} />;
```
