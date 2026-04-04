# 1. Deep Mental Model of TypeScript

## Structural Typing

TypeScript checks **shape**, not **name**. Two types with different names but identical structure are interchangeable.

```ts
type Dog = { name: string; breed: string };
type Cat = { name: string; breed: string };

const dog: Dog = { name: "Rex", breed: "Lab" };
const cat: Cat = dog; // valid — same shape
```

This is structural typing. Languages like Java/C# use nominal typing — types must match by name, not just shape.

A function requiring a type will accept any object that satisfies its shape, even with extra fields:

```ts
type User = { id: number; name: string };

function greet(user: User) {
  console.log(user.name);
}

const admin = { id: 1, name: "Alice", role: "admin" };
greet(admin); // valid — admin has everything User requires, plus more
```

## Type System as a Functional Language

TypeScript has two layers running at once:

- **Runtime layer** — JavaScript, executes in browser/Node
- **Type layer** — a separate computation, exists only at compile time

The type layer is its own programming language. You can write logic in it — conditionals, recursion, pattern matching:

```ts
type IsString<T> = T extends string ? true : false;

type A = IsString<string>; // true
type B = IsString<number>; // false
```

`IsString` is a "function" at the type level — takes an input, returns an output, evaluated by the compiler, erased at runtime.

## Compile Time vs Runtime

```
tsc runs → checks types → writes .js to disk    ← compile time
                                ↓
          browser loads .js → executes it         ← runtime
```

**Compile time** — `tsc` transforms and checks your code. Nothing is running yet.

**Runtime** — the `.js` output is executed. TypeScript is completely gone.

```ts
// What you write (.ts)
const greet = (user: User): string => {
  return `Hello ${user.name}`;
};

// What runs in the browser (.js)
const greet = (user) => {
  return `Hello ${user.name}`;
};
```

All type annotations are stripped. The browser has never heard of TypeScript.

## Type Erasure & Runtime Boundaries

Because types are erased, they provide **zero protection at runtime**. Data from outside your TypeScript world — API responses, `JSON.parse`, user input — can be anything:

```ts
const res = await fetch("/api/user");
const data = await res.json(); // typed as any — TS has no idea what this is

const name: string = data.name; // no error, but not guaranteed at runtime
```

`fetch` and `JSON.parse` are **runtime boundaries** — points where data crosses from the outside world into your app. TypeScript cannot check what happens there.

This is why runtime validation libraries like Zod exist.

## Soundness vs Practicality

A sound type system guarantees that if TS says something is a `string`, it is always a string at runtime. TypeScript is **intentionally unsound** in places:

```ts
const arr: string[] = [];
const first: string = arr[0]; // TS says string — actually undefined
```

TS trades some safety for usability. The biggest escape hatch is `any` — it disables the type checker entirely for that value. Prefer `unknown` when the type is genuinely unknown, as it forces you to narrow before use.

## Key Takeaways

1. TS checks **shape, not name** — structural typing
2. Types are a **compile-time computation** — erased before runtime
3. Type annotations do **not validate at runtime** — runtime boundaries are your responsibility
4. TS is **pragmatically unsound** — don't trust types at system boundaries
