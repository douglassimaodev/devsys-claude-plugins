---
name: create-unit-tests
description: Use whenever adding or improving tests in a .NET/Angular project — xUnit, NSubstitute, FluentAssertions on the backend, Vitest on the frontend, deterministic and complete scenarios.
---

# Skill: create-unit-tests

## Backend Test Stack
- xUnit
- NSubstitute
- FluentAssertions

## Frontend Test Stack
- Vitest (`npm test -- --watch=false`)

## Rules
- Prefer `async Task` over `void`.
- Create all relevant tests for the implemented behavior.
- Cover success path.
- Cover validation failures.
- Cover authorization- and ownership-sensitive logic when applicable.
- Cover edge cases; for money, availability/overlap and status logic, cover boundaries explicitly (rounding, overlapping dates, invalid transitions, duplicate webhooks).
- Verify important dependency calls with `Received()` / `DidNotReceive()` when applicable.
- Do not write placeholder comments.
- Do not say "add more tests later".
- Tests must be deterministic.

## Test Naming
`MethodName_StateUnderTest_ExpectedBehavior`

## Output
1. Test scenarios
2. Test files created or changed
3. Important assertions
4. Mocked dependencies
5. How to run tests
