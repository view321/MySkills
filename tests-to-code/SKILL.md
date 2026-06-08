---
name: tests-to-code
description: Convert tests to code
---

# tests-to-code

Implement production code that satisfies the test suite created by the `make-api-with-tests` skill.

## Goal
Write the smallest clear implementation that passes the approved tests while preserving the API contract those tests define.

## Process
1. Read the approved tests and identify the required public API, inputs, outputs, errors, and side effects.
2. Implement only the code needed to satisfy the tests. Do not change approved tests unless the user explicitly approves a correction.
3. Keep behavior aligned with the tests rather than adding speculative features.
4. Run or mentally validate the tests after each meaningful change.
5. If a test exposes an ambiguity, explain the ambiguity and choose the behavior most consistent with the existing tests.
6. Refactor after tests pass, keeping the code simple and readable.

## Implementation Standards
Use clear names, minimal dependencies, and idiomatic patterns for the chosen language. Validate inputs where tests require it. Handle edge cases exactly as specified by the tests. Prefer deterministic behavior and avoid hidden global state unless the API contract requires it.

## Output
Show the implementation code and note which approved tests it satisfies. If execution is possible, report the test results. If execution is not possible, state what was checked and any uncertainty.
