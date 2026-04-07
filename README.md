# Advanced TypeScript Types — Interview Prep Syllabus

Tailored for frontend engineers leveling up from TS fundamentals to interview-ready advanced knowledge.

---

## Progress

- [x] 0. Prerequisites (Assumed)
- [x] 1. Deep Mental Model of TypeScript
- [x] 2. Advanced Type System Foundations
- [x] 3. Generics — Beyond Basics
- [ ] 4. Conditional Types (Core of Advanced TS)
- [ ] 5. Mapped Types & Key Transformations
- [ ] 6. Union & Intersection Mastery
- [ ] 7. `infer` — Deep Dive
- [ ] 8. Template Literal Types (Power Feature)
- [ ] 9. Utility Types — Internals & Customization
- [ ] 10. Type Guards & Narrowing at Scale
- [ ] 11. The `satisfies` Operator
- [ ] 12. Declaration Merging & Module Augmentation
- [ ] 13. Type Inference Internals
- [ ] 14. Function Typing Mastery
- [ ] 15. Advanced Patterns
- [ ] 16. Edge Cases & Pitfalls
- [ ] 17. Capstone Projects
- [ ] 18. Interview-Level Problems

---

## 0. Prerequisites (Assumed)

- Strong JS fundamentals
- Solid TypeScript basics (types, interfaces, generics)
- Familiarity with async patterns & modern frontend/backend architecture

---

## 1. Deep Mental Model of TypeScript

- Structural typing vs nominal typing
- Type system as a functional programming language
- Type-level computation mindset
- Soundness vs practicality tradeoffs
- Type erasure & runtime boundaries

> This is critical—most devs fail here, not on syntax.

---

## 2. Advanced Type System Foundations

- `unknown`, `never`, `any` — deep semantics
- Variance (covariance, contravariance, bivariance)
- Subtyping rules
- Type widening vs narrowing
- Literal types & const assertions
- `as const` + `typeof` for deriving union types from data

```ts
const ROLES = ["admin", "user", "guest"] as const;
type Role = typeof ROLES[number]; // "admin" | "user" | "guest"
```

> These underpin everything else.

---

## 3. Generics — Beyond Basics

- Generic constraints (`extends`)
- Default type parameters
- Higher-order generics
- Generic inference tricks
- Bounded polymorphism

```ts
type ApiResponse<T extends object = {}> = {
  data: T;
  error?: string;
};
```

---

## 4. Conditional Types (Core of Advanced TS)

- `T extends U ? X : Y`
- Distributive conditional types
- Pattern matching with `infer`
- Nested & recursive conditionals

```ts
type ExtractPromise<T> = T extends Promise<infer U> ? U : T;
```

> Widely used in real-world libraries.

---

## 5. Mapped Types & Key Transformations

- Basic mapped types
- Key remapping (`as`)
- Conditional mapped types
- Modifiers (`readonly`, `?`)

```ts
type Mutable<T> = {
  -readonly [K in keyof T]: T[K];
};
```

> Core for transforming object shapes — heavily tested in interviews.

---

## 6. Union & Intersection Mastery

- Discriminated unions
- Exhaustiveness checking
- Union distribution behavior
- Intersection pitfalls

```ts
type Result =
  | { status: "ok"; data: string }
  | { status: "error"; error: string };
```

---

## 7. `infer` — Deep Dive

- Inferring return types, argument types, promise resolution
- `infer` in conditional chains
- Multiple `infer` in a single type
- Variance of inferred positions

```ts
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : never;
type FirstArg<T> = T extends (first: infer A, ...rest: any[]) => any ? A : never;
```

> Deserves its own focus — comes up constantly in interviews.

---

## 8. Template Literal Types (Power Feature)

- String manipulation at type level
- Pattern matching on strings
- Building:
  - Route parsers
  - Event name systems
  - CSS-in-JS typings

```ts
type EventName<T extends string> = `on${Capitalize<T>}`;
```

> A uniquely powerful TS feature — increasingly common in frontend interviews.

---

## 9. Utility Types — Internals & Customization

- Built-ins: `Partial`, `Required`, `Pick`, `Omit`, `Awaited`
- Re-implementing them from scratch
- Designing custom utilities

---

## 10. Type Guards & Narrowing at Scale

- Custom type predicates
- Assertion functions
- Control-flow analysis

> Essential for real-world correctness.

---

## 11. The `satisfies` Operator

- `satisfies` vs type annotation vs `as`
- When to use `satisfies` for inference-preserving validation
- Frontend patterns: config objects, route maps, theme tokens

```ts
const palette = {
  red: [255, 0, 0],
  green: "#00ff00",
} satisfies Record<string, string | number[]>;

palette.red; // number[] (preserved, not widened)
```

> Added in TS 4.9 — increasingly asked about in interviews.

---

## 12. Declaration Merging & Module Augmentation

- Interface merging
- Augmenting third-party types (`window`, `process.env`, component props)
- Module augmentation pattern

```ts
declare global {
  interface Window {
    analytics: AnalyticsClient;
  }
}
```

> Common frontend pattern — comes up when working with third-party libraries.

---

## 13. Type Inference Internals

- How inference works
- Inference in functions, tuples, and conditional types
- Controlling inference with `NoInfer<T>`

---

## 14. Function Typing Mastery

- Function overloads vs unions
- Variadic tuple types
- Inferring arguments & return types

---

## 15. Advanced Patterns

- Branded / nominal types
- Phantom types
- State machines with discriminated unions
- Tagged unions for domain modeling

---

## 16. Edge Cases & Pitfalls

- Excess property checks
- Unexpected `never`
- Inference failures
- Recursive depth limits

---

## 17. Capstone Projects

Build real-world, type-heavy systems:

- Fully typed REST client
- Form builder with inferred schema
- Router with typed params
- Type-safe event emitter with DOM-style event maps
- Type-safe query builder

---

## 18. Interview-Level Problems

Grind these — they cover the full range of what gets asked:

- Implement `DeepPartial<T>`
- Implement `DeepReadonly<T>`
- Implement `UnionToIntersection<T>`
- Implement `IsNever<T>`
- Implement `Awaited<T>` from scratch
- Type-safe `pipe` function
- Typed `Object.entries` wrapper
- Typed `reduce` with inference
- Tuple length math
- JSON parser (type-level)
