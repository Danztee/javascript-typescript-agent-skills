# TypeScript decision rules

Use these examples when a task has more than one reasonable implementation. Follow the repository's existing style first.

## Runtime boundaries

Types do not validate JSON or other runtime values. Narrow once at the boundary:

```ts
function parseUser(value: unknown): User {
  if (!isUser(value)) {
    throw new TypeError('Invalid user payload')
  }

  return value
}
```

Do not repeat the same shape checks in every internal function. Do not replace a real parser with `value as User`; an assertion changes the compiler's belief, not the runtime value.

## `satisfies` versus assertions

Use `satisfies` when a value must conform while preserving useful inference:

```ts
const routes = {
  home: '/',
  profile: '/profile',
} satisfies Record<string, `/${string}`>
```

Avoid `as SomeType` when it only silences an error. If the value is genuinely unknown, use `unknown` and narrow it. Keep an unavoidable `any` local to the integration boundary and document why.

## Type shape defaults

- Use inferred types for obvious locals and explicit types for exported APIs and non-obvious contracts.
- Use `type` for unions, tuples, and mapped/conditional composition; use `interface` for extendable object contracts when the project has no stronger convention.
- Prefer string-literal unions over enums unless runtime enum behavior is required.
- Use `field?: T` when omission has meaning; use `field: T | undefined` when the property must exist but its value may be absent.
- Add a generic only when it represents a real relationship between inputs and outputs.

## Async and verification

```ts
const [user, settings] = await Promise.all([
  fetchUser(),
  fetchSettings(),
])
```

Do not use `forEach(async ...)`, leave promises floating, or treat a typecheck as runtime proof. When changing module resolution, package exports, declarations, decorators, JSX, or loaders, verify emitted/imported behavior with the real build or runtime.

## Compiler and lint performance

Do not annotate everything or enable every strict/type-aware lint rule during an unrelated feature. If compilation, editor, declaration emit, or linting is slow, measure first; then add named boundaries, split complex types, or narrow lint scope where that addresses the measured cause.
