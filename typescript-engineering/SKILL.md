---
name: typescript-engineering
description: Build, change, debug, and review TypeScript with useful static guarantees, precise boundaries, and practical runtime behavior for the project's framework and toolchain.
---

# TypeScript engineering

Use this skill for TypeScript implementation, refactoring, debugging, type design, and code review. Treat the type system as a tool for making valid states easy to represent—not as a reason to add ceremony or encode every possibility in types.

## First inspect the project

- Read `package.json`, `tsconfig` files, source conventions, and existing test/lint/format/build scripts before making design decisions.
- Determine whether the project emits JavaScript, uses a bundler/runtime transpiler, runs types directly, or uses TypeScript only for checking. Respect path aliases, module resolution, ESM/CommonJS settings, declaration output, and package exports.
- Match the project's TypeScript and `typescript-eslint` versions. Do not change compiler options or migrate tooling unless the request calls for it.

## Consistency protocol

TypeScript offers several ways to encode the same idea. Reduce variation with this order of precedence:

1. Match the nearest working code and public API conventions.
2. Match the repository's compiler, lint, runtime, and build configuration.
3. If the repository is silent, use the defaults below and use them consistently throughout the change.
4. Do not perform an unrelated type-style migration just to normalize older code.

When multiple implementations are equally sound, select one rather than making agents or users choose between equivalent patterns. Call out alternatives only when they change the runtime contract, emitted output, API compatibility, or meaningful maintenance cost.

When the repository is silent, prefer:

- Inferred types for local variables; explicit types for exported functions, public data, callback contracts, and non-obvious return values.
- `type` for unions, tuples, mapped/conditional types, and composition; `interface` for object contracts intended to be extended or implemented. Follow local convention when either works.
- String-literal unions over enums unless runtime enum objects, reverse mapping, or an established project convention is actually needed.
- `unknown` at untrusted boundaries, followed by one narrow/parse step; localized `any` only for a documented, unavoidable integration gap.
- `satisfies` for config-like values that need checking while retaining their inferred literals; narrowing or a purpose-built type guard instead of broad `as` assertions.
- Optional properties (`field?: T`) when omission has meaning; `field: T | undefined` when the property is required but its value may be absent.
- A generic only when it represents a real relationship between multiple inputs/outputs or materially improves a reusable API.
- `import type` for type-only imports when the project uses or requires it; do not rewrite imports without a compiler/tooling reason.

## Ecosystem pain-point guardrails

These are common sources of wasted time and “almost correct” agent output:

- Architecture and state: trace ownership and data flow before introducing a store, singleton, event bus, cache, or elaborate generic abstraction. Keep state close to its consumer when possible; do not use types to hide unclear runtime ownership.
- Dependencies: search the existing dependency set and platform APIs first. Add a package only when it solves a meaningful problem the current stack cannot solve simply, and change manifests/lockfiles with the project's package manager.
- Build versus runtime: separate typechecking, transpilation, bundling, and execution in your reasoning. Verify the actual emitted/imported code when module resolution, package `exports`, decorators, JSX, declarations, or runtime loaders are involved. Do not “fix” a runtime problem with `any`, `skipLibCheck`, or a path alias that only works in the editor.
- Compiler performance: avoid giant anonymous inferred types, deeply recursive conditional types, and unnecessary type-aware linting across generated/config files. Add named boundaries or split types when a measured compile, declaration-emit, editor, or lint slowdown justifies it; do not annotate everything preemptively.
- Configuration and upgrades: preserve reproducible lockfiles and existing `tsconfig` inheritance. Enable stricter compiler/lint profiles incrementally; do not enable every rule or flag during an unrelated feature change, especially when the resulting errors cannot be triaged in scope.
- Dates and time: make timezone, locale, precision, and serialization explicit. Do not rely on ambiguous external date parsing or silently treat a local time as UTC. Follow the project's date library or supported platform API.
- Debugging: reproduce the failure, form a narrow hypothesis, inspect source/config/generated output, and make the smallest fix that explains the behavior. Do not change several unrelated variables and call the first green run proof.
- Agent drift: do not copy a remembered framework recipe over the installed version. Do not add `eslint-disable`, `@ts-ignore`, broad assertions, or a new package merely to silence a diagnostic; explain and verify the underlying issue instead.

## Type design

- Prefer inference for obvious local values. Add explicit types where they document or constrain public APIs, exported data, callbacks, complex return values, and important boundaries.
- Model meaningful alternatives with discriminated unions and exhaustiveness checks. Prefer a small precise type over a large hierarchy or a generic that exists only to look reusable.
- Use `unknown` for values whose type is not known, then narrow them. Avoid `any`; if it is genuinely required for an untyped dependency or an unrepresentable edge, keep it local, explain the boundary, and avoid allowing it to spread.
- Use `satisfies` when a value must conform to a type while retaining its useful inferred literal type. Use `as const`, utility types, and generics when they make the contract clearer—not as substitutes for understanding the data flow.
- Avoid broad assertions (`as SomeType`), non-null assertions (`!`), `Function`, boxed primitives, and empty-object types as shortcuts. Replace them with a real invariant, a narrow guard, a better API, or a justified localized escape hatch.
- Prefer readonly types or immutable data only where they communicate an actual ownership/invariant. Do not impose blanket immutability on code that intentionally mutates local state.

## Runtime boundaries and validation

TypeScript types are erased at runtime. Separate compile-time contracts from runtime facts:

- Parse, validate, and normalize values entering from HTTP, CLI, environment, files, JSON, databases, browser storage, webhooks, and third-party packages. Use the project's existing schema/parser library when one exists.
- Perform boundary validation once and pass the resulting typed representation inward. Do not duplicate the same checks in every internal function.
- For internal functions whose callers and types establish a non-null, correctly-shaped value, do not add speculative guards, fallback values, or branches for impossible states. If the invariant is wrong, fix the type or caller and let the failure remain visible.
- Treat `unknown` as a prompt to narrow at the boundary, not as a reason to sprinkle `typeof` checks throughout trusted business logic.
- Be precise about absence. `undefined`, `null`, an omitted optional property, an empty collection, and a falsy value are not interchangeable unless the domain says they are.
- Keep security controls regardless of type safety: authorization, input constraints, injection prevention, output encoding, secret handling, and resource limits are runtime concerns.

## Compiler and linting posture

- For new projects, prefer a strict compiler posture when compatible with the runtime and migration plan. For existing projects, make incremental, local improvements instead of turning on every strictness flag opportunistically.
- Consider `strict`, `useUnknownInCatchVariables`, `noUncheckedIndexedAccess`, and `exactOptionalPropertyTypes` based on the codebase's actual data model and tolerance for migration noise. Explain the tradeoff before changing them.
- Let the TypeScript compiler own checks it can prove. Avoid duplicating compiler diagnostics as manual runtime conditions or redundant ESLint rules.
- If typed linting is already configured, use its diagnostics to catch issues such as floating promises, unsafe `any` flow, and unnecessary conditions. Do not enable expensive type-aware linting blindly on every generated, config, or JavaScript file.
- Use Prettier or the repository's formatter for formatting; use ESLint for correctness and maintainability rules rather than arguing about whitespace in review.

## Async, errors, and APIs

- Give exported async APIs accurate `Promise<T>` behavior through inference or explicit return types. Handle independent operations concurrently with the appropriate `Promise` combinator.
- Do not leave promises unhandled. An intentional fire-and-forget call should have an explicit, repository-appropriate convention and a known rejection strategy; `void` alone documents intent but does not handle a rejection.
- Catch `unknown` errors safely at the layer that can recover, translate, clean up, or report. Preserve causes and useful domain context; do not swallow errors or log the same failure at every layer.
- Keep runtime API contracts aligned with the types. If a function can return `undefined`, make callers handle that; if it cannot, do not add a meaningless fallback just to silence a concern.

## Testing and verification

- Test behavior and contracts, not TypeScript syntax or implementation trivia. Include boundary parsing, discriminant branches, important failure behavior, and regression cases.
- Keep type-level tests only for public type APIs or inference guarantees that users rely on; keep them small and use the project's established tool.
- After changes, run the relevant tests plus the repository's typecheck/build/lint commands when available. Check emitted/runtime behavior when module resolution, declarations, decorators, or build transforms are involved.

## Review checklist

Prioritize:

1. Type/runtime mismatches, unsound assertions, leaked `any`, and incorrect optionality.
2. Missing validation at real external boundaries and duplicated validation inside trusted code.
3. Async errors, floating promises, race conditions, and cleanup behavior.
4. Public API compatibility, module-resolution/package-export mistakes, and generated declaration quality.
5. Type complexity, speculative generics, unnecessary compiler/lint churn, and defensive branches that obscure the real invariant.
6. `as` used to silence an error, non-null assertions, leaked `any`, dumping unknown data into `Record<string, unknown>`, and `Partial<T>` used where the API really requires a complete value.
7. `forEach(async ...)`, un-awaited `map(async ...)`, accidental sequential awaits, inconsistent `type`/`interface` or enum/union choices, and runtime checks duplicated after parsing.
8. Date/time ambiguity, dependency churn, typechecking-only verification, runtime/module-resolution mismatches, unexplained compiler or typed-lint slowdowns, and configuration changes unrelated to the requested behavior.

Do not recommend a type-level abstraction, compiler flag, or validation layer merely because it exists. Recommend it when it prevents a demonstrated class of bugs or materially clarifies the contract.

## Reference material

- TypeScript Handbook: https://www.typescriptlang.org/docs/handbook/
- TypeScript narrowing: https://www.typescriptlang.org/docs/handbook/2/narrowing.html
- `strict`: https://www.typescriptlang.org/tsconfig/strict.html
- `noUncheckedIndexedAccess`: https://www.typescriptlang.org/tsconfig/noUncheckedIndexedAccess.html
- `exactOptionalPropertyTypes`: https://www.typescriptlang.org/tsconfig/exactOptionalPropertyTypes.html
- `useUnknownInCatchVariables`: https://www.typescriptlang.org/tsconfig/useUnknownInCatchVariables.html
- TypeScript utility types: https://www.typescriptlang.org/docs/handbook/utility-types.html
- typescript-eslint shared configs: https://typescript-eslint.io/users/configs/
- `no-explicit-any`: https://typescript-eslint.io/rules/no-explicit-any/
- `no-floating-promises`: https://typescript-eslint.io/rules/no-floating-promises/
- `no-unnecessary-condition`: https://typescript-eslint.io/rules/no-unnecessary-condition/
- TypeScript compiler performance guidance: https://github.com/microsoft/TypeScript/wiki/Performance
- TypeScript configuration/upgrade tradeoffs: https://github.com/microsoft/TypeScript/issues/50997
- OWASP input validation guidance: https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html
- State of JavaScript 2025 pain points: https://2025.stateofjs.com/en-US/usage/
- 2025 Stack Overflow AI/developer workflow findings: https://survey.stackoverflow.co/2025/ai

These references inform decisions; they do not override project constraints or the user's preference for proportionate validation and low ceremony.
