# JavaScript decision rules

Use these examples when a task has more than one reasonable implementation. Follow the repository's existing style first.

## Async work

Independent operations should start together:

```js
// Avoid: three avoidable round trips.
const user = await fetchUser()
const settings = await fetchSettings()
const flags = await fetchFlags()

// Prefer: one failure policy and concurrent work.
const [user, settings, flags] = await Promise.all([
  fetchUser(),
  fetchSettings(),
  fetchFlags(),
])
```

Use `for...of` when each iteration intentionally depends on the previous one. Use `map` plus `Promise.all` when each operation is independent. Do not use `forEach(async ...)` when the caller must wait.

When the collection can be large or externally controlled, bound concurrency with the project's existing limiter or process incrementally. Consider memory, rate limits, failure semantics, and cancellation before starting all operations.

If the operation creates timers, listeners, subscriptions, streams, or clients, pair ownership with cleanup through `finally` or the project's lifecycle hook. Thread an existing `AbortSignal` when cancellation is part of the surrounding contract.

## Boundary validation

Parse an external value once, then pass the trusted representation inward:

```js
const raw = await request.json()
const input = parseCreateUserInput(raw) // shape and business rules live here
return createUser(input) // no duplicate hypothetical checks
```

Do not add `if (!input)` inside `createUser` when its contract and caller already establish that `input` exists. Keep authentication, authorization, resource limits, and injection defenses where the threat requires them.

## Missing values

Use the operator that matches the domain:

```js
const page = options.page ?? 1 // 0 is a valid page value
const label = options.label || 'Unnamed' // every falsy label is intentionally replaced
```

Do not add optional chaining or fallbacks just to suppress a theoretical null case. If absence is invalid, fix the contract or let the failure remain visible.

## Collections and state

Use `find`, `some`, and `every` when their intent is the operation. Use a loop for early exit, sequential async work, or complex control flow. Use shallow spread only when a shallow copy is intended; do not deep-clone by reflex.

Before adding a store, singleton, event bus, or cache, identify who owns the state, its lifetime, and its invalidation policy. Prefer local ownership when shared state is not required.

## Runtime verification

When changing imports, package exports, extensions, or build configuration, run the actual project entrypoint or test runtime. A successful lint or parse does not prove that Node.js or the browser can load the module.
