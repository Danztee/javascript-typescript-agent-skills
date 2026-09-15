---
name: javascript-engineering
description: Build, change, debug, and review modern JavaScript for browser, Node.js, and full-stack projects while matching the repository's runtime, module system, and tooling.
---

# JavaScript engineering

Use this skill for JavaScript implementation, refactoring, debugging, and code review. Optimize for code that is clear, idiomatic, observable, and easy to change—not for maximum ceremony.

## First inspect the project

- Read `package.json`, the relevant source files, and the existing test, lint, format, and build scripts before choosing patterns.
- Identify the runtime (browser, Node.js, worker, or framework), package manager, module system, supported language target, and established conventions.
- Preserve local conventions unless the user asks for a migration. Do not introduce a new framework, validator, formatter, or abstraction just because it is popular.

## Consistency protocol

JavaScript has many valid spellings. Reduce unnecessary variation with this order of precedence:

1. Match the nearest working code that solves the same problem.
2. Match the repository's configured tooling and runtime constraints.
3. If the repository is silent, use the defaults below and use them consistently throughout the change.
4. Do not refactor unrelated code just to make the whole repository stylistically uniform.

When several approaches remain reasonable, choose one instead of presenting a menu of equivalent implementations. Mention an alternative only when it changes behavior, compatibility, performance, or maintenance cost.

When the repository is silent, prefer:

- `async`/`await` for promise-based control flow; `Promise.all` with `map` for independent async work; `for...of` for intentionally sequential async work. Never use `forEach(async () => ...)` when the caller must wait.
- One module system per package. Follow existing default-versus-named export conventions; if none exist, prefer named exports for modules with multiple public values.
- `find`, `some`, and `every` for their named intent; `map` for one-to-one transformation; `filter` for selection; a loop when early exit, `await`, or complex control flow is clearer.
- `??` when only `null`/`undefined` mean “missing”; `||` only when every falsy value should use the fallback.
- Shallow object spread for an intentional shallow copy; direct mutation for short-lived, locally owned state. Do not deep-clone by reflex.
- `Error` objects with useful messages and causes, rather than strings or ad hoc result shapes, unless the project already uses a Result-style API.

## Ecosystem pain-point guardrails

These are common sources of wasted time and “almost correct” agent output:

- Architecture and state: trace ownership and data flow before adding a store, singleton, event bus, cache, or utility layer. Keep state close to its consumer when possible, and do not introduce global mutable state to avoid passing a few values.
- Dependencies: search the existing dependency set and platform APIs before adding a package. Add a dependency only when it solves a meaningful problem the current stack cannot solve simply; update manifests and lockfiles with the project's package manager.
- Dates and time: make timezone, locale, precision, and serialization explicit. Do not rely on ambiguous external date parsing or silently treat a local time as UTC. Follow the project's date library or supported platform API.
- Build/runtime behavior: verify the actual entrypoint, package `type`, extensions, import specifiers, exports, bundler transforms, and test runtime. Code that parses or lints successfully is not necessarily runnable code.
- Debugging: reproduce the failure, form a narrow hypothesis, inspect the relevant source/config/generated output, and make the smallest fix that explains the behavior. Do not change several unrelated variables and call the first green run proof.
- Performance: measure before micro-optimizing, but catch obvious accidental costs such as unbounded concurrency, repeated expensive work in loops, accidental quadratic scans, N+1 I/O, and unbounded caches.
- Agent drift: do not copy a remembered framework recipe over the installed version. Do not add `eslint-disable`, `// @ts-ignore`, broad fallbacks, or a new package just to make a task appear complete; explain and verify the underlying issue instead.

## Core coding guidance

- Prefer modern language features that the configured target supports: `const` by default, `let` when reassignment is real, modules, destructuring when it improves readability, and `async`/`await` for promise-based flows.
- Keep functions and modules cohesive. Prefer simple data flow and named helpers over clever chains, premature generalization, or large “utility” layers.
- Use strict equality and intentional coercion. Treat truthiness carefully when `0`, `''`, `false`, or `null` have different meanings.
- Preserve the project's mutability model. Do not ban mutation universally; isolate it when shared state or reasoning complexity makes that useful.
- Avoid `eval`, string-built code execution, accidental global state, and APIs that obscure ownership or cleanup.
- Match the project's formatting and linting configuration. Formatting preferences are not a reason to make unrelated code changes.

## Async and errors

- `await` independent operations concurrently with `Promise.all`, `Promise.allSettled`, `Promise.any`, or `Promise.race` according to the required failure semantics. Do not serialize independent work accidentally.
- Handle a promise at the point where its result or failure matters. Do not leave a promise floating unless the fire-and-forget behavior is deliberate and its rejection path is accounted for.
- Catch errors only when you can recover, add useful context, translate to a domain error, or perform required cleanup. Do not catch merely to rethrow, log and rethrow at every layer, or replace a useful error with a generic one.
- Preserve error causes when wrapping (`new Error(message, { cause: error })` where the runtime supports it) and keep user-facing/logging decisions at an appropriate boundary.
- Do not add `try`/`catch` around every `await`. Let errors propagate to the nearest layer that can make a meaningful decision.
- Make resource cleanup explicit with the project's supported mechanism (`finally`, disposers, abort signals, or framework lifecycle hooks).

## Validation and defensive programming

Use the smallest amount of validation that protects a real boundary or contract:

- Validate and normalize data crossing a trust boundary: HTTP requests, CLI arguments, environment variables, files, JSON, databases, browser storage, webhooks, and third-party APIs. Check both shape and business meaning when the operation depends on them.
- Once data has been validated, keep the validated representation and do not re-check the same invariant in every helper.
- For internal functions with a clear contract, do not add defensive branches for hypothetical misuse when the caller and control flow already establish the invariant. Fix the caller or type/contract instead.
- Do not use optional chaining, nullish fallbacks, default values, or `if (!value)` guards by reflex. Add them only when absence is a valid runtime state with intentional behavior.
- Do not add runtime type checks to compensate for a static TypeScript guarantee in code that is actually compiled and controlled by the same project. JavaScript and external data still need runtime checks where the boundary is real.
- Never weaken security or correctness in the name of minimalism. Authentication, authorization, injection prevention, output encoding, resource limits, and input validation remain necessary when the threat or contract requires them.

## Testing and verification

- Follow the existing test runner and test style. Test observable behavior, important error semantics, boundary parsing, and regressions; avoid tests coupled to incidental implementation details.
- Prefer small deterministic tests and real collaborators when practical. Mock only unstable, expensive, unavailable, or externally owned dependencies.
- After changes, run the narrowest relevant tests first, then the repository's type/build/lint checks when available. Report checks that could not be run and why.

## Review checklist

When reviewing JavaScript, prioritize:

1. Incorrect async sequencing, ignored rejections, races, and cleanup failures.
2. Trust-boundary mistakes, injection/encoding issues, authorization gaps, and unsafe evaluation.
3. Incorrect assumptions about `undefined`, `null`, falsy values, mutation, or module loading.
4. Error handling that loses context or silently changes failure into success.
5. Unnecessary guards, duplicate parsing, speculative abstractions, and unrelated churn.
6. `forEach(async ...)`, un-awaited `map(async ...)`, accidental sequential awaits, promise-wrapping of promise APIs, and `||`/`??` mistakes.
7. Inconsistent module/export choices, hidden mutation, `filter(...)[0]` where `find` expresses intent, and optional chaining that masks a broken invariant.
8. Ambiguous dates, dependency churn, global state added without ownership justification, unbounded concurrency, and changes verified only by linting or typechecking.

Do not request a stylistic rewrite when the code is correct, readable, and consistent with the repository.

## Reference material

Use these as grounding, not as a mandatory one-size-fits-all style guide:

- MDN JavaScript Guide: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide
- MDN modules: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules
- MDN promises and async error handling: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Using_promises
- Node.js packages and module systems: https://nodejs.org/api/packages.html and https://nodejs.org/api/esm.html
- Node.js test runner: https://nodejs.org/api/test.html
- ESLint configuration: https://eslint.org/docs/latest/use/configure/
- StandardJS rules: https://github.com/standard/standard/blob/master/RULES.md
- Airbnb JavaScript Style Guide: https://github.com/airbnb/javascript
- OWASP input validation guidance: https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html
- State of JavaScript 2025 pain points: https://2025.stateofjs.com/en-US/usage/
- 2025 Stack Overflow AI/developer workflow findings: https://survey.stackoverflow.co/2025/ai

The style guides are useful evidence about common practice, not authority over a repository's existing design or the user's stated preferences.
