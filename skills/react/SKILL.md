---
name: react
description: React and TypeScript development with Tailwind, shadcn/ui, and modern patterns
---

# React

You are a React and TypeScript expert. Apply these guidelines when working on React projects.

## Tech Stack

This skill assumes:
- **TypeScript** for type safety
- **React 18+** with functional components
- **Tailwind CSS** for styling
- **shadcn/ui** for components
- **pnpm** as package manager

## Project Structure

```
src/
├── app/                    # Next.js app router pages
├── components/
│   ├── ui/                 # shadcn/ui components
│   └── features/           # Feature-specific components
├── hooks/                  # Custom hooks
├── lib/                    # Utilities (cn, api clients)
├── types/                  # Shared TypeScript types
└── services/               # API and data fetching
```

## TypeScript Patterns

### Type Definitions

```typescript
// Prefer interfaces for extendable objects
interface User {
  id: string;
  name: string;
  email: string;
}

// Use type for unions and computed types
type Status = 'idle' | 'loading' | 'success' | 'error';
type UserWithRole = User & { role: Role };

// Props interfaces above components
interface ButtonProps {
  variant?: 'default' | 'destructive' | 'outline';
  size?: 'sm' | 'md' | 'lg';
  disabled?: boolean;
  onClick?: () => void;
  children: React.ReactNode;
}
```

### Generics

```typescript
// Reusable hooks with generics
function useLocalStorage<T>(key: string, initialValue: T) {
  const [value, setValue] = useState<T>(() => {
    const stored = localStorage.getItem(key);
    return stored ? JSON.parse(stored) : initialValue;
  });
  return [value, setValue] as const;
}

// Constrained generics
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}
```

### Event Handlers

```typescript
const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
  setValue(e.target.value);
};

const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
  e.preventDefault();
  onSubmit(formData);
};

const handleClick = (e: React.MouseEvent<HTMLButtonElement>) => {
  e.stopPropagation();
  onClick?.();
};
```

## Component Patterns

### Basic Structure

```typescript
interface CardProps {
  title: string;
  description?: string;
  children: React.ReactNode;
  className?: string;
}

export function Card({
  title,
  description,
  children,
  className
}: CardProps) {
  return (
    <div className={cn(
      "rounded-lg border bg-card p-6 shadow-sm",
      className
    )}>
      <h3 className="text-lg font-semibold">{title}</h3>
      {description && (
        <p className="mt-1 text-sm text-muted-foreground">
          {description}
        </p>
      )}
      <div className="mt-4">{children}</div>
    </div>
  );
}
```

### Using shadcn/ui

```typescript
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import {
  Dialog,
  DialogContent,
  DialogHeader,
  DialogTitle,
  DialogTrigger,
} from "@/components/ui/dialog";

export function CreateItemDialog() {
  const [open, setOpen] = useState(false);

  return (
    <Dialog open={open} onOpenChange={setOpen}>
      <DialogTrigger asChild>
        <Button>Create Item</Button>
      </DialogTrigger>
      <DialogContent>
        <DialogHeader>
          <DialogTitle>Create New Item</DialogTitle>
        </DialogHeader>
        <form onSubmit={handleSubmit}>
          <Input placeholder="Item name" />
          <Button type="submit" className="mt-4">
            Create
          </Button>
        </form>
      </DialogContent>
    </Dialog>
  );
}
```

## Tailwind Patterns

### The cn() Utility

```typescript
// lib/utils.ts
import { type ClassValue, clsx } from "clsx";
import { twMerge } from "tailwind-merge";

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}
```

### Conditional Classes

```typescript
<div
  className={cn(
    "rounded-lg border p-4 transition-colors",
    isActive && "border-primary bg-primary/5",
    isDisabled && "cursor-not-allowed opacity-50",
    className
  )}
>
  {children}
</div>
```

### Responsive Design

```typescript
<div className="grid grid-cols-1 gap-4 md:grid-cols-2 lg:grid-cols-3">
  {items.map((item) => (
    <ItemCard key={item.id} item={item} />
  ))}
</div>
```

### Common Patterns

```typescript
// Centering
<div className="flex min-h-screen items-center justify-center">

// Card container
<div className="rounded-lg border bg-card p-6 shadow-sm">

// Flex with gap
<div className="flex items-center gap-2">

// Truncated text
<p className="truncate text-sm text-muted-foreground">

// Hover states
<button className="hover:bg-accent hover:text-accent-foreground">
```

## Hooks

### State Management

```typescript
// useState with explicit type when needed
const [user, setUser] = useState<User | null>(null);
const [items, setItems] = useState<Item[]>([]);

// useReducer for complex state
type Action =
  | { type: 'SET_LOADING'; payload: boolean }
  | { type: 'SET_DATA'; payload: Data }
  | { type: 'SET_ERROR'; payload: Error };

const [state, dispatch] = useReducer(reducer, initialState);
```

### Effects

```typescript
// With cleanup
useEffect(() => {
  const controller = new AbortController();

  fetchData(controller.signal)
    .then(setData)
    .catch((error) => {
      if (!controller.signal.aborted) {
        setError(error);
      }
    });

  return () => controller.abort();
}, [dependency]);
```

### Custom Hooks

```typescript
function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const timer = setTimeout(() => setDebouncedValue(value), delay);
    return () => clearTimeout(timer);
  }, [value, delay]);

  return debouncedValue;
}

function useToggle(initial = false) {
  const [value, setValue] = useState(initial);
  const toggle = useCallback(() => setValue((v) => !v), []);
  const setTrue = useCallback(() => setValue(true), []);
  const setFalse = useCallback(() => setValue(false), []);
  return { value, toggle, setTrue, setFalse };
}
```

## Data Fetching

### Server Components (Next.js)

```typescript
// Server Component - no 'use client' directive
async function UserList() {
  const users = await fetchUsers();

  return (
    <ul className="space-y-2">
      {users.map((user) => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

### Client-Side Fetching

```typescript
'use client';

function useUsers() {
  const [users, setUsers] = useState<User[]>([]);
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    const controller = new AbortController();

    async function load() {
      try {
        const data = await fetchUsers(controller.signal);
        setUsers(data);
      } catch (e) {
        if (!controller.signal.aborted) {
          setError(e as Error);
        }
      } finally {
        setIsLoading(false);
      }
    }

    load();
    return () => controller.abort();
  }, []);

  return { users, isLoading, error };
}
```

## Forms

### With React Hook Form

```typescript
import { useForm } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";
import { z } from "zod";

const schema = z.object({
  email: z.string().email("Invalid email"),
  password: z.string().min(8, "Password must be at least 8 characters"),
});

type FormData = z.infer<typeof schema>;

export function LoginForm() {
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting },
  } = useForm<FormData>({
    resolver: zodResolver(schema),
  });

  const onSubmit = async (data: FormData) => {
    await login(data);
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)} className="space-y-4">
      <div>
        <Input {...register("email")} placeholder="Email" />
        {errors.email && (
          <p className="mt-1 text-sm text-destructive">
            {errors.email.message}
          </p>
        )}
      </div>
      <div>
        <Input
          {...register("password")}
          type="password"
          placeholder="Password"
        />
        {errors.password && (
          <p className="mt-1 text-sm text-destructive">
            {errors.password.message}
          </p>
        )}
      </div>
      <Button type="submit" disabled={isSubmitting}>
        {isSubmitting ? "Signing in..." : "Sign In"}
      </Button>
    </form>
  );
}
```

## Naming Conventions

### Files and Folders

- Components: `kebab-case.tsx` or `PascalCase.tsx`
- Hooks: `use-hook-name.ts`
- Utilities: `kebab-case.ts`
- Types: `types.ts` or `type-name.ts`

### Code

```typescript
// Components: PascalCase
function UserProfile() {}

// Hooks: camelCase with 'use' prefix
function useAuth() {}

// Constants: SCREAMING_SNAKE_CASE
const API_BASE_URL = '/api';
const MAX_RETRY_COUNT = 3;

// Event handlers: handle + Event
const handleClick = () => {};
const handleSubmit = () => {};
const handleInputChange = () => {};

// Boolean variables: is/has/can/should prefix
const isLoading = true;
const hasError = false;
const canSubmit = true;
```

## Error Handling

### Error Boundaries

```typescript
'use client';

interface ErrorBoundaryProps {
  children: React.ReactNode;
  fallback: React.ReactNode;
}

class ErrorBoundary extends Component<ErrorBoundaryProps, { hasError: boolean }> {
  state = { hasError: false };

  static getDerivedStateFromError() {
    return { hasError: true };
  }

  render() {
    if (this.state.hasError) {
      return this.props.fallback;
    }
    return this.props.children;
  }
}
```

### API Error Handling

```typescript
class ApiError extends Error {
  constructor(
    message: string,
    public statusCode: number,
    public code?: string
  ) {
    super(message);
    this.name = 'ApiError';
  }
}

async function fetchWithError<T>(url: string): Promise<T> {
  const response = await fetch(url);

  if (!response.ok) {
    throw new ApiError(
      'Request failed',
      response.status,
      response.statusText
    );
  }

  return response.json();
}
```

## Performance

### Memoization

```typescript
// Memoize expensive computations
const sortedItems = useMemo(
  () => items.sort((a, b) => a.name.localeCompare(b.name)),
  [items]
);

// Memoize callbacks passed to children
const handleDelete = useCallback((id: string) => {
  setItems((prev) => prev.filter((item) => item.id !== id));
}, []);

// Memoize components that render often
const MemoizedRow = memo(function Row({ item }: { item: Item }) {
  return <div>{item.name}</div>;
});
```

### Code Splitting

```typescript
// Lazy load heavy components
const HeavyChart = lazy(() => import('./HeavyChart'));

function Dashboard() {
  return (
    <Suspense fallback={<Skeleton className="h-64 w-full" />}>
      <HeavyChart data={data} />
    </Suspense>
  );
}
```

## Common Commands

```bash
# Create new Next.js project
pnpm create next-app@latest my-app --typescript --tailwind --eslint

# Add shadcn/ui
pnpm dlx shadcn@latest init

# Add shadcn components
pnpm dlx shadcn@latest add button input dialog

# Add dependencies
pnpm add zod react-hook-form @hookform/resolvers

# Development
pnpm dev

# Build
pnpm build
```

## Common Pitfalls

1. **Missing keys** - Always provide stable, unique keys in lists
2. **Stale closures** - Include all dependencies in useEffect/useCallback
3. **Prop drilling** - Use context or composition for deep props
4. **Over-memoization** - Profile before optimizing
5. **Type assertions** - Prefer type guards over `as` casting
6. **Missing 'use client'** - Add directive for client-only features
7. **Inline objects in JSX** - Creates new references on every render
