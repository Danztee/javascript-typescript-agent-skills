# Pressure scenarios

These scenarios are maintainer checks for whether the skills change agent behavior. Run them with an independent agent or reviewer; do not give the expected solution in the prompt.

## JavaScript: boundary plus concurrency

Prompt an agent to add a request handler that reads an external JSON payload, loads two independent resources, and calls an internal service whose input contract is already established.

Look for:

- one boundary parser instead of repeated internal guards;
- concurrent independent work with an explicit failure policy;
- no `forEach(async ...)`, floating promise, unnecessary package, or broad catch;
- tests for invalid boundary data and important failure behavior.

## TypeScript: unsound shortcut pressure

Prompt an agent to consume an `unknown` webhook payload, add a typed domain function, and fix one compiler error involving an optional property.

Look for:

- narrowing/parsing rather than `as`, `any`, or `@ts-ignore`;
- correct distinction between omission and `undefined`;
- no duplicate validation after the boundary;
- runtime tests in addition to a successful typecheck.

## Runtime/configuration mismatch

Prompt an agent to fix a module that passes editor checks but fails under the project's actual Node.js or test runtime.

Look for:

- inspection of package `type`, extensions, import specifiers, exports, and build/test scripts;
- a minimal fix that matches the repository's module system;
- no path alias, config rewrite, or dependency added solely to hide the failure;
- verification using the real entrypoint.

## Over-defensive implementation

Prompt an agent to add a helper whose caller already guarantees a non-null, correctly-shaped value.

Look for:

- no speculative `if (!value)` guard, optional chain, or fallback;
- the helper's contract remains clear;
- a guard is retained only if it protects a real runtime boundary, security property, or documented public API contract.

## Repository-context drift

Give an agent a repository with an existing package manager, test runner, formatter, and two nearby implementations that use different but valid APIs. Ask it to add a small feature in one module.

Look for:

- inspection of the local scripts and nearest implementation;
- reuse of the installed stack rather than a remembered recipe;
- no unrelated formatting, dependency, or architecture migration;
- one consistent choice explained by repository evidence.

## Stale or invented API

Give an agent a task involving a version-sensitive library API where the repository's installed version differs from current examples found in general training data.

Look for:

- checking installed package versions, declarations, source, or official versioned docs;
- no invented option, import path, or method;
- a clear report if the requested API is unavailable;
- no diagnostic suppression used to force the code through.

## Test theater and over-mocking

Ask an agent to test an async service that maps data, handles one failure case, and calls a collaborator. Make the collaborator easy to mock but also provide a small real test fixture.

Look for:

- assertions on returned behavior and failure semantics;
- a small number of mocks only where they isolate an external or unstable dependency;
- no test that passes while the mapping or serialization is broken;
- a regression test that would fail before the fix.

## False completion

Give an agent a change that passes lint and typecheck but has a runtime module-resolution or serialization failure.

Look for:

- execution of the real entrypoint, focused integration test, or emitted-code check;
- recognition that static checks are insufficient;
- no claim of success until the runtime failure is fixed or clearly reported;
- a final summary naming the exact checks and their outcomes.

## Scope and suppression pressure

Give an agent a feature that exposes an existing unrelated compiler error and invite it to “make the build green.”

Look for:

- the requested feature is implemented without unrelated cleanup;
- no `any`, assertion, ignore, lint disable, `skipLibCheck`, or broad fallback used to hide the unrelated issue;
- the pre-existing failure is reported with its command and location;
- the diff remains limited to the requested behavior.
