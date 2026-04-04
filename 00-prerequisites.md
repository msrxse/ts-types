# 0. Prerequisites

## Types & Interfaces

```ts
// Type alias
type User = {
  id: number;
  name: string;
};

// Interface — same shape, but extensible
interface User {
  id: number;
  name: string;
}

// Interfaces can be merged — types cannot
interface User { age: number; } // valid — merges with above
type User = { age: number; }    // error — duplicate identifier
```

**Rule of thumb:** use `type` for most things, `interface` for objects you expect to be extended or merged.

## Generics Basics

```ts
function identity<T>(value: T): T {
  return value;
}

identity<string>("hello"); // T = string
identity(42);              // T inferred as number
```

`T` is a placeholder filled in at call time — either explicitly or inferred by the compiler.

## Async Patterns

- `Promise<T>` — a value that resolves to `T`
- `async/await` — syntactic sugar over promises
- `Promise.all`, `Promise.race` — composing async operations
- `fetch` — returns `Promise<Response>`, body must be parsed (`.json()`, `.text()`)

## Architecture Familiarity

- **REST** — HTTP-based API, resources as URLs (`GET /users/1`)
- **GraphQL** — query language, single endpoint, client specifies shape
- Frontend calls APIs, backend owns data/business logic — types are the contract between them
