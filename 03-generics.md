# 3. Generics — Beyond Basics

## Generic Constraints with `extends`

Constraints restrict what `T` can be. Without a constraint, TS won't let you access any properties on `T`:

```ts
function getLength<T>(value: T): number {
  return value.length; // error — T might not have length
}
```

Add a constraint:

```ts
function getLength<T extends { length: number }>(value: T): number {
  return value.length; // valid — T is guaranteed to have length
}

getLength("hello");               // valid — string has length
getLength([1, 2, 3]);             // valid — array has length
getLength({ length: 10, id: 1 }); // valid — has length
getLength(42);                    // error — number has no length
```

`extends` here means "must satisfy this shape" — not inheritance. Structural typing rules apply.

### `T extends { length: number }` vs `T[]`

- `T[]` — argument must be an array. Strings, custom objects excluded.
- `T extends { length: number }` — anything with a `length` property. More flexible.

With `T[]` you also lose the original type. With a constraint, `T` is still the full original type — matters when returning `T` or accessing other properties.

---

## Default Type Parameters

Type parameters can have defaults:

```ts
type ApiResponse<T = unknown> = {
  data: T;
  error?: string;
};

type UserResponse = ApiResponse<User>; // T = User
type RawResponse = ApiResponse;        // T = unknown — default kicks in
```

Useful when you want a generic type to work without always specifying the type argument.

```ts
type EventHandler<T = Event> = (event: T) => void;

const handleClick: EventHandler<MouseEvent> = (e) => e.clientX;
const handleAny: EventHandler = (e) => e.preventDefault(); // uses default
```

**Rule:** once you use a default, all subsequent type parameters must also have defaults:

```ts
type Foo<A, B = string, C = number> // valid
type Foo<A, B = string, C>          // error
```

---

## Higher-Order Generics

Higher-order generics take **type constructors** as type arguments — types that are themselves generic.

```ts
type Maybe<T> = T | null;
type List<T> = T[];
```

You pass the generic type itself as an argument — a "type function":

```ts
// F is not a concrete type — it's a generic type constructor
type Apply<F extends <T>(arg: T) => any, A> = F<A>;
```

Seen in the wild in React — `ComponentType<P>` accepts any component that takes props `P`, whether class or function.

---

## Generic Inference Tricks

TS infers `T` automatically from arguments — you rarely need to specify it explicitly.

```ts
function wrap<T>(value: T): { value: T } {
  return { value };
}

wrap("hello") // T inferred as string
wrap(42)      // T inferred as number
```

**Inferring from multiple arguments:**

```ts
function pair<A, B>(a: A, b: B): [A, B] {
  return [a, b];
}

pair("hello", 42) // A = string, B = number
```

**`extends` to preserve literal types:**

Without a constraint, TS widens inference:

```ts
function identity<T>(value: T): T { return value; }
identity("hello") // T = string — widened
```

With `extends string`, TS preserves the literal:

```ts
function identity<T extends string>(value: T): T { return value; }
identity("hello") // T = "hello" — literal preserved
```

`extends string` signals to TS to keep the literal rather than widen. The constraint allows any string — but inference behaves differently.

**Inferring through callbacks:**

```ts
function transform<T, R>(value: T, fn: (val: T) => R): R {
  return fn(value);
}

transform("hello", (s) => s.length) // T = string, R = number — both inferred
```

TS chains inference — it knows `s` is `string` because `T` was inferred first.

---

## Bounded Polymorphism

Using `extends` to **preserve the exact type** of an argument while still constraining it.

Without it — you lose the specific type:

```ts
function firstElement(arr: any[]): any {
  return arr[0];
}

const result = firstElement(["a", "b"]); // any — lost string
```

With a generic constraint — you preserve it:

```ts
function firstElement<T>(arr: T[]): T {
  return arr[0];
}

const result = firstElement(["a", "b"]); // string — preserved
```

**The key pattern — `keyof` with bounded polymorphism:**

```ts
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const user = { id: 1, name: "Alice" };
const name = getProperty(user, "name"); // string
const id = getProperty(user, "id");     // number
getProperty(user, "age");               // error — not a key of user
```

`K extends keyof T` constrains the key, and `T[K]` gives you the exact return type. This pattern underpins utility types like `Pick` and `Omit`.

**Constraining to object for spreads:**

```ts
function merge<T extends object, U extends object>(a: T, b: U): T & U {
  return { ...a, ...b };
}
```

Both `T` and `U` must extend `object` — the spread operator only works on objects.
