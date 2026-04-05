# 2. Advanced Type System Foundations

## `any`, `unknown`, `never`

### `any`

Completely opts out of the type system. You can do anything with it — call it, index it, assign it anywhere. The compiler goes silent.

```ts
const x: any = "hello";
x.toUpperCase(); // fine
x.foo.bar.baz;   // fine — no errors, but will blow up at runtime
x();             // fine
```

Avoid it — it's contagious and spreads through your code.

### `unknown`

The safe alternative to `any`. Accepts any value, but you can't do anything with it until you narrow it first.

```ts
const x: unknown = "hello";
x.toUpperCase(); // error — TS doesn't know what x is

if (typeof x === "string") {
  x.toUpperCase(); // fine — narrowed to string
}
```

Use `unknown` at runtime boundaries (`JSON.parse`, API responses) — it forces you to validate before use.

### `never`

Represents something that can never happen — a value that can never exist.

**A function that never returns:**

```ts
function throwError(message: string): never {
  throw new Error(message);
}
```

**Exhaustiveness checking — the key interview use:**

```ts
type Status = "active" | "inactive";

function handleStatus(status: never): never {
  throw new Error(`Unhandled status: ${status}`);
}

function process(status: Status) {
  switch (status) {
    case "active": return doSomething();
    case "inactive": return doOther();
    default: return handleStatus(status); // status is never here
  }
}
```

In the `default` branch TS has eliminated all known cases — `status` is `never`. If you add `"pending"` to `Status` and forget to handle it, passing `"pending"` to a `never` parameter errors at compile time.

The `throw` is a last line of defense for runtime (e.g. value came from `any`) — the `never` parameter is the real compile-time guard.

---

## Variance

Variance describes how subtyping relationships carry through when you wrap a type.

```ts
class Animal { breathe() {} }
class Dog extends Animal { bark() {} }
// Dog is the subtype (more specific), Animal is the supertype (more general)
```

### Covariance — same direction

Arrays are covariant. `Dog[]` is assignable to `Animal[]`:

```ts
const dogs: Dog[] = [new Dog()];
const animals: Animal[] = dogs; // valid
```

### Contravariance — flipped direction

Function parameters are contravariant. A function that handles `Animal` can stand in for one that handles `Dog` — not the other way around:

```ts
type HandleAnimal = (a: Animal) => void;
type HandleDog = (d: Dog) => void;

const handleAnimal: HandleAnimal = (a) => a.breathe();
const handleDog: HandleDog = handleAnimal; // valid — can handle any animal, including dogs
```

### Bivariance — both directions (default for methods)

Method parameters in TS are bivariant by default — unsound but practical. Use `--strictFunctionTypes` to enforce contravariance on function type aliases.

**Memory hook:**
- Covariant — same direction (arrays, return types)
- Contravariant — flipped direction (function parameters)
- Bivariant — both allowed (method parameters by default)

---

## Subtyping Rules

A type `A` is a subtype of `B` if `A` has everything `B` has, plus potentially more.

- **Supertype** — more general, fewer properties (`Animal`)
- **Subtype** — more specific, more properties (`Dog`)

```ts
// Widening — subtype to supertype. Always safe.
const dog: Dog = { breathe: () => {}, bark: () => {} };
const animal: Animal = dog; // valid

// Narrowing — supertype to subtype. Not safe — TS errors.
const animal: Animal = { breathe: () => {} };
const dog: Dog = animal; // error — animal might not have bark
```

Memory hook: subtype = subset = more specific = more properties.

---

## Type Widening vs Narrowing

### Widening — TS making a type more general

```ts
let name = "Alice";   // inferred as string — widened
const name = "Alice"; // inferred as "Alice" — literal preserved
```

`let` widens because the value might change. `const` keeps the literal.

### Narrowing — TS making a type more specific

```ts
function process(value: string | number) {
  if (typeof value === "string") {
    value.toUpperCase(); // string here
  } else {
    value.toFixed(2); // number here
  }
}
```

TS tracks control flow and narrows the type in each branch.

---

## Literal Types & `as const` + `typeof`

### Literal types

A type that is exactly one specific value:

```ts
type Direction = "north" | "south" | "east" | "west";
type StatusCode = 200 | 404 | 500;
```

### `as const` + `typeof` — deriving unions from data

Instead of manually writing a union, derive it from real data:

```ts
const DIRECTIONS = ["north", "south", "east", "west"] as const;
type Direction = typeof DIRECTIONS[number]; // "north" | "south" | "east" | "west"
```

- `as const` — freezes the array, each item becomes its literal type
- `typeof DIRECTIONS` — gets the type of the array
- `[number]` — indexes into it, producing the union of all values

Works with objects too:

```ts
const ROLES = {
  admin: "admin",
  user: "user",
  guest: "guest",
} as const;

type Role = typeof ROLES[keyof typeof ROLES]; // "admin" | "user" | "guest"
```

The source of truth is the data — add a value to `DIRECTIONS` and the union updates automatically.
