# TypeScript Best Practices

TypeScript coding standards including type definitions, generics, utility types, and strict mode configuration. These guidelines help AI assistants generate type-safe, maintainable TypeScript code.

## Core Guidelines

### 1. Enable Strict Mode

Always use `strict: true` in tsconfig.json. This enables all strict type-checking options and catches errors at compile time.

**Example**:
```json
// Good (tsconfig.json)
{
  "compilerOptions": {
    "strict": true,
    "target": "ES2020",
    "module": "ESNext",
    "moduleResolution": "node"
  }
}

// Also Good (individual strict options)
{
  "compilerOptions": {
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "strictBindCallApply": true,
    "strictPropertyInitialization": true,
    "noImplicitThis": true,
    "alwaysStrict": true
  }
}

// Avoid (no strict mode)
{
  "compilerOptions": {
    "target": "ES2020"
  }
}
```

### 2. Define Explicit Types for Function Parameters and Returns

Always type function parameters and return values. Avoid relying on type inference for public APIs.

**Example**:
```typescript
// Good
function fetchUser(userId: number): Promise<User> {
    return fetch(`/api/users/${userId}`)
        .then(res => res.json());
}

function calculateTotal(items: Item[]): number {
    return items.reduce((sum, item) => sum + item.price, 0);
}

// Avoid (implicit any)
function fetchUser(userId) {
    return fetch(`/api/users/${userId}`)
        .then(res => res.json());
}

// Avoid (no return type)
function calculateTotal(items: Item[]) {
    return items.reduce((sum, item) => sum + item.price, 0);
}
```

### 3. Use Interfaces for Object Shapes

Define interfaces for object structures, especially for function parameters and return types. Use `type` for unions, intersections, and primitives.

**Example**:
```typescript
// Good (interfaces for objects)
interface User {
    id: number;
    name: string;
    email: string;
    role: UserRole;
}

interface UserCreateInput {
    name: string;
    email: string;
    password: string;
}

function createUser(input: UserCreateInput): Promise<User> {
    // Implementation
}

// Good (type for unions)
type Status = 'pending' | 'active' | 'cancelled';
type Result<T> = { success: true; data: T } | { success: false; error: string };

// Avoid (type for simple object)
type User = {
    id: number;
    name: string;
}; // Use interface instead
```

### 4. Leverage Utility Types

Use TypeScript's built-in utility types (Partial, Required, Pick, Omit, etc.) to transform existing types instead of duplicating definitions.

**Example**:
```typescript
// Good (utility types)
interface User {
    id: number;
    name: string;
    email: string;
    createdAt: Date;
}

// For updates, only some fields required
type UserUpdate = Partial<Pick<User, 'name' | 'email'>>;

// For creation, omit generated fields
type UserCreate = Omit<User, 'id' | 'createdAt'>;

// Make all fields required
type CompleteUser = Required<User>;

function updateUser(id: number, data: UserUpdate): Promise<User> {
    // Implementation
}

// Avoid (duplicate definitions)
interface UserUpdate {
    name?: string;
    email?: string;
}

interface UserCreate {
    name: string;
    email: string;
}
```

### 5. Use Generics for Reusable Components

Write generic functions and classes when logic applies to multiple types. Constrain generics with `extends` when needed.

**Example**:
```typescript
// Good (generic function)
function findById<T extends { id: number }>(
    items: T[],
    id: number
): T | undefined {
    return items.find(item => item.id === id);
}

// Generic API response wrapper
interface ApiResponse<T> {
    data: T;
    status: number;
    message: string;
}

async function fetchData<T>(url: string): Promise<ApiResponse<T>> {
    const response = await fetch(url);
    return response.json();
}

// Usage with type inference
const user = await fetchData<User>('/api/users/1');
const posts = await fetchData<Post[]>('/api/posts');

// Avoid (non-generic with any)
function findById(items: any[], id: number): any {
    return items.find(item => item.id === id);
}
```

### 6. Avoid Using `any` Type

Never use `any` except in rare cases. Use `unknown` for truly unknown types, then narrow with type guards. Use generics for flexible types.

**Example**:
```typescript
// Good (unknown with type guard)
function processData(data: unknown): string {
    if (typeof data === 'string') {
        return data.toUpperCase();
    }
    if (typeof data === 'number') {
        return data.toString();
    }
    throw new Error('Unsupported data type');
}

// Good (type guard function)
function isUser(value: unknown): value is User {
    return (
        typeof value === 'object' &&
        value !== null &&
        'id' in value &&
        'name' in value
    );
}

function handleUser(data: unknown): void {
    if (isUser(data)) {
        console.log(data.name); // TypeScript knows it's User
    }
}

// Avoid (any)
function processData(data: any): string {
    return data.toString(); // No type safety!
}
```

### 7. Use Const Assertions and Readonly

Use `as const` for literal types and `readonly` for immutable properties. This prevents accidental mutations and provides better type inference.

**Example**:
```typescript
// Good (const assertion)
const COLORS = {
    PRIMARY: '#007bff',
    SECONDARY: '#6c757d',
    SUCCESS: '#28a745'
} as const;

type Color = typeof COLORS[keyof typeof COLORS];
// Color = "#007bff" | "#6c757d" | "#28a745"

// Good (readonly interface)
interface Config {
    readonly apiUrl: string;
    readonly timeout: number;
    readonly retries: number;
}

const config: Config = {
    apiUrl: 'https://api.example.com',
    timeout: 5000,
    retries: 3
};

// config.timeout = 10000; // Error: Cannot assign to readonly property

// Good (readonly array)
function processItems(items: readonly string[]): number {
    // items.push('new'); // Error: Cannot modify readonly array
    return items.length;
}

// Avoid (mutable literals)
const COLORS = {
    PRIMARY: '#007bff',
    SECONDARY: '#6c757d'
};
// Type inference: { PRIMARY: string; SECONDARY: string }
```

## Quick Reference

- [ ] Strict mode enabled in tsconfig.json
- [ ] All function parameters and returns explicitly typed
- [ ] Interfaces used for object shapes, types for unions
- [ ] Utility types (Partial, Pick, Omit) used to transform types
- [ ] Generics used for reusable, type-safe functions
- [ ] `any` avoided; `unknown` with type guards used instead
- [ ] Const assertions and readonly used for immutability
- [ ] Type guards defined for runtime type checking
