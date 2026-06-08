---
name: make-api-with-tests
description: Make program's API with tests
---

# make-api-with-tests

Create a clear API design for the requested program by building its test suite one approved test at a time.

## Goal
Define the program’s public interface and expected behavior through tests before implementation. Cover normal behavior, edge cases, invalid inputs, boundary values, failure modes, and any invariants implied by the request.

## Process
1. Restate the target program briefly, including assumptions about language, framework, and test runner. If essential details are missing, choose sensible defaults and state them.
2. Propose the API shape: function names, classes, endpoints, input/output formats, errors, and side effects.
3. Create exactly one test case.
4. Show the test to the user with a short explanation of what behavior it specifies.
5. Stop and wait for explicit approval, rejection, or edits.
6. After approval, create the next test case and repeat.
7. Do not write implementation code in this skill.
8. Do not create multiple tests at once unless the user explicitly asks.

## Test Quality
Each test should be executable, focused, and named clearly. Prefer small tests that isolate one behavior. Include setup, action, and assertion. Avoid brittle assertions unless the behavior requires exact output.

## Completion
When the user says the test suite is complete, summarize the API contract and list the completed tests in order. Make clear that implementation should be handled by the `tests-to-code` skill.
