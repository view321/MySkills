---
name: make-md-dir
description: Workflow to create or update a markdown directory that describes a codebase before it is implemented or updated by parse-md-dir
---

# `make-md-dir` skill

This skill creates a **markdown directory**: a directory of `.md` files that describes a complete codebase in enough detail for another skill, such as `parse-md-dir`, to translate the markdown files into real code.

The markdown directory is the source of truth for the future implementation.

This skill should not implement production code. It should only create or update markdown files that describe the intended codebase.

---

# Inputs

The user may provide one or more of the following:

- An existing codebase directory
- A product idea
- An app description
- A feature request
- A technical specification
- A README
- An existing markdown directory that needs to be updated
- Existing API documentation
- Existing tests or test requirements
- Architecture preferences
- A desired technology stack
- A desired output directory for the markdown directory
- A desired output directory for the eventual codebase

If the user does not specify an output directory, create a directory named:

```text
markdown-directory
```

If the project has a clear name, use:

```text
<project-name>-md
```

---

# Output directory structure

Create or update the markdown directory with this structure:

```text
markdown-directory/
  STACK.md
  PROJECT.md
  API_REFERENCE.md
  FILE_TREE.md
  TEST_PLAN.md
  IMPLEMENTATION_NOTES.md
  files/
    <mirrored-code-path>.md
```

For every intended code file, create one corresponding markdown file.

Example mapping:

```text
src/auth/session.ts -> files/src/auth/session.ts.md
tests/auth/session.test.ts -> files/tests/auth/session.test.ts.md
app/api/users/route.ts -> files/app/api/users/route.ts.md
```

The `files/` directory should mirror the future codebase directory structure exactly, except each intended code file has `.md` appended.

---

# Core rules

## Do not write implementation code

This skill creates markdown descriptions only.

Do not create production code files such as:

```text
.ts
.js
.py
.go
.rs
.java
.cs
.php
.rb
.swift
.kt
.html
.css
```

Small code-like examples are allowed inside markdown when useful, but the output of this skill is documentation, not implementation.

## The markdown directory must be implementation-ready

The markdown files should be detailed enough that another agent can implement the code without making major architecture decisions.

Every important file, class, function, method, variable, type, endpoint, command, component, schema, model, and test must be described.

## Use contract-style documentation everywhere

Every per-file markdown document must describe code using explicit contracts.

For each file, class, function, method, variable, constant, type, interface, component, hook, command, endpoint, schema, model, and event, document:

- Invariants: facts that must always remain true.
- Preconditions: facts that must be true before the item is used, called, read, written, initialized, rendered, invoked, or persisted.
- Postconditions: facts that must be true after the item is used, called, read, written, initialized, rendered, invoked, or persisted.
- Error conditions: what must happen when preconditions are violated or expected operations fail.
- Side effects: storage writes, network calls, emitted events, logs, cache changes, process exits, state mutations, or other external effects.

Do not limit contract documentation to public APIs. Internal helpers, private methods, module-level variables, constants, and local state that affect behavior also need contracts.

## Prefer explicitness over brevity

Avoid vague instructions such as:

```text
Implement authentication logic.
```

Instead, write specific instructions such as:

```text
This file exports a function named `createSession`.
It accepts a user ID and returns a signed session token.
It validates that the user ID is non-empty.
It uses the configured session secret from `src/config/env.ts`.
It throws `SessionCreationError` if token signing fails.
The returned token is never empty.
The token expiration timestamp is always later than the issue timestamp.
```

## Keep one source of truth

`API_REFERENCE.md` is the project-wide source of truth.

Each per-file markdown file must agree with `API_REFERENCE.md`.

If there is a conflict, update the markdown directory so the conflict is resolved.

---

# Workflow

## Step 1: Determine the source mode

Identify which mode applies.

### Mode A: Existing codebase to markdown directory

Use this mode when the user provides an existing codebase.

The goal is to reverse-engineer the current codebase into a clean markdown directory.

### Mode B: Product or specification to markdown directory

Use this mode when the user provides a product idea, README, feature request, or architecture description, but no existing codebase.

The goal is to design a future codebase in markdown form.

### Mode C: Existing markdown directory update

Use this mode when a markdown directory already exists.

The goal is to update the markdown directory according to new requirements or changes in the codebase.

---

## Step 2: Create or update `STACK.md`

Create a `STACK.md` file at the root of the markdown directory.

This file describes the preferred implementation stack.

If the user explicitly gave a stack, use it.

If an existing codebase is provided, infer the stack from files such as:

```text
package.json
tsconfig.json
vite.config.*
next.config.*
pyproject.toml
requirements.txt
Pipfile
poetry.lock
Cargo.toml
go.mod
pom.xml
build.gradle
composer.json
Gemfile
Dockerfile
docker-compose.yml
```

If the stack cannot be fully determined, make reasonable choices and mark uncertain assumptions clearly.

Use this structure:

```markdown
# Stack

## Language

## Runtime

## Frameworks

## Package manager

## Test framework

## Build tools

## Linting and formatting

## Database

## External services

## Environment variables

## Deployment assumptions

## Notes and assumptions
```

---

## Step 3: Create or update `PROJECT.md`

Create a project overview file.

Use this structure:

```markdown
# Project

## Purpose

## Main features

## Users or consumers

## High-level architecture

## Data flow

## Important constraints

## Non-goals

## Glossary
```

The purpose of this file is to help future implementation agents understand the project as a whole.

---

## Step 4: Create or update `FILE_TREE.md`

Create a file tree describing the intended final codebase.

The file tree should include all intended source files, test files, configuration files, scripts, and documentation files that matter to implementation.

Example:

````markdown
# File Tree

```text
src/
  index.ts
  config/
    env.ts
  auth/
    session.ts
    password.ts
  users/
    user.model.ts
    user.service.ts
tests/
  auth/
    session.test.ts
package.json
tsconfig.json
README.md
```

## File descriptions

### `src/index.ts`

Application entry point.

### `src/auth/session.ts`

Session token creation, verification, and expiration handling.
````

---

## Step 5: Create or update `API_REFERENCE.md`

Create a project-wide API reference.

This file is the source of truth for all public and internal APIs that future implementation agents must follow.

Depending on the project, include sections for:

```markdown
# API Reference

## Modules

## Classes

## Functions

## Methods

## Types and interfaces

## Variables and constants

## Components

## Hooks

## Commands

## HTTP endpoints

## Events

## Database models

## Configuration values

## Environment variables

## Errors

## External integrations
```

Each API entry should include contract information.

Use this template:

```markdown
## `<name>`

### Kind

Function, class, method, type, component, endpoint, command, constant, variable, etc.

### Defined in

Path to the intended code file.

### Purpose

What this API does.

### Signature or shape

Expected function signature, method signature, class shape, endpoint route, command syntax, type definition, variable type, or constant value shape.

### Invariants

Facts that must always remain true for this API.

### Preconditions

Facts that must be true before this API is used.

### Postconditions

Facts that must be true after this API is used successfully.

### Parameters

Names, types, required/optional status, and meaning.

### Returns

Return type and meaning.

### Behavior

Detailed behavior.

### Errors

Expected errors, exceptions, rejected promises, HTTP statuses, or failure modes.

### Side effects

File system writes, network calls, database writes, emitted events, logging, state changes, process exits, etc.

### Dependencies

Other project APIs or external libraries used.

### Used by

Known callers or consumers.

### Testing requirements

How this API should be tested, including invariant, precondition, and postcondition tests.
```

When updating an existing API reference, preserve useful existing information but correct anything that conflicts with the current requirements or source files.

---

## Step 6: Create one markdown file per intended code file

For every intended code file, create a corresponding markdown file under `files/`.

The markdown file must describe only that one future code file.

The file must use contract-style documentation for every important definition.

Use this template:

```markdown
# `<intended-code-file-path>`

## Purpose

Describe why this file exists.

## Responsibilities

List what this file is responsible for.

## Non-responsibilities

List what this file should not do.

## File invariants

List facts that must always remain true for this file as a module.

Examples:

- This file does not perform direct database writes.
- This file never logs secrets or password hashes.
- Public functions return immutable DTOs.
- Soft-deleted records are excluded from normal query results.

## Exports

Describe every exported class, function, method, type, constant, variable, component, command, or module member.

For each export, include a contract section using the required definition contract template below.

## Internal definitions

Describe important internal helpers, private functions, private methods, private classes, local constants, local variables, module-level state, and internal types.

For each internal definition that affects behavior, include a contract section using the required definition contract template below.

## Dependencies

Describe imports from other project files and external libraries.

For each dependency, explain why it is needed and what contract the current file expects from it.

## Public API reference entries

List the matching entries from `API_REFERENCE.md`.

## Detailed behavior

Explain the logic this file should implement.

## Error handling

Describe expected validation, exceptions, rejected promises, HTTP errors, fallback behavior, or recovery behavior.

## Side effects

Describe database writes, network calls, file system access, logging, process exits, environment reads, cache writes, emitted events, or global state changes.

## Data structures

Describe important objects, types, schemas, models, or interfaces.

For each important data structure, document invariants, valid states, invalid states, and normalization rules.

## State and variables

Describe every meaningful module-level variable, mutable field, cache, constant, configuration object, and shared state value.

For each variable or constant, document:

- Purpose
- Type or shape
- Initial value
- Invariants
- Preconditions for reading
- Preconditions for writing or mutation, if mutable
- Postconditions after initialization
- Postconditions after mutation, if mutable
- Whether the value may contain secrets or sensitive data

## Configuration

Describe environment variables, config values, feature flags, or constants used by this file.

## Testing requirements

Describe the tests that should exist for this file.

Tests must cover invariants, preconditions, postconditions, error conditions, and side effects for important definitions.

## Edge cases

Describe important edge cases.

## Example usage

Include short illustrative examples if helpful.

## Notes for implementation agent

Mention constraints, assumptions, ordering requirements, or things the implementing agent should avoid.

## Notes and uncertainties

Document uncertainty when source behavior is unclear, incomplete, or contradictory.
```

### Required definition contract template

Every class, function, method, variable, constant, type, interface, component, hook, endpoint, command, schema, model, and event described in a per-file markdown file must use this contract format or an equivalent format with the same information.

For a class or module-level object:

```markdown
# `<ClassOrObjectName>`

## Kind

Class, service, repository, module object, model, component, etc.

## Purpose

Describe why this item exists.

## Invariants

- Facts that must always remain true for this class or object.
- Include data visibility guarantees, immutability rules, normalization rules, lifecycle rules, authorization assumptions, and consistency requirements.

## Constructor or initialization

### Preconditions

- Facts that must be true before constructing or initializing this item.

### Postconditions

- Facts that must be true after construction or initialization succeeds.

### Errors

- Errors thrown, returned, or emitted when initialization fails.

## Fields and properties

Document each meaningful field or property using the variable contract template.

## Methods

Document each method using the method contract template.
```

For a function or method:

```markdown
## Method: `<methodName>(<parameters>)`

### Purpose

Describe what the method does.

### Preconditions

- Facts that must be true before the method is called.
- Include validation requirements, authorization assumptions, required state, and dependency availability.

### Postconditions

- Facts that must be true after the method succeeds.
- Include returned value guarantees, persisted state changes, emitted events, normalization, caching, and cleanup.

### Parameters

- `<name>`: type, required/optional, valid values, normalization rules, and meaning.

### Returns

Describe the returned value, including nullability and sensitive-field guarantees.

### Errors

Describe what happens when preconditions fail or dependencies fail.

### Side effects

Describe storage writes, network calls, emitted events, cache updates, logs, metrics, or state mutation.

### Edge cases

Describe important boundary cases.

### Tests

Describe tests that prove the preconditions, postconditions, invariants, errors, and side effects.
```

For a variable, constant, field, or property:

```markdown
## Variable: `<variableName>`

### Purpose

Describe why this value exists.

### Type or shape

Describe the type, schema, or allowed values.

### Initial value

Describe how the value is initialized.

### Invariants

- Facts that must always remain true for this value.

### Preconditions for reading

- Facts that must be true before reading the value.

### Preconditions for writing or mutation

- Facts that must be true before changing the value.
- Use `Not mutable` if the value must never be changed after initialization.

### Postconditions after initialization

- Facts that must be true after the value is initialized.

### Postconditions after writing or mutation

- Facts that must be true after the value changes.
- Use `Not mutable` if the value must never be changed after initialization.

### Sensitive data

State whether this value may contain secrets, credentials, tokens, personal data, password hashes, or other sensitive data.

### Tests

Describe tests or checks that prove the variable contract.
```

For a type, interface, schema, DTO, or database model:

```markdown
## Type: `<TypeName>`

### Purpose

Describe why this structure exists.

### Shape

Describe fields, nested fields, optional fields, defaults, and allowed values.

### Invariants

- Facts that must always remain true for valid values of this type.

### Preconditions for creation

- Facts that must be true before constructing, validating, parsing, or persisting this type.

### Postconditions after creation

- Facts that must be true after this type is created, parsed, validated, or persisted.

### Normalization rules

Describe lowercasing, trimming, sorting, deduplication, defaulting, coercion, or canonicalization.

### Invalid states

Describe states that must be rejected or impossible.

### Sensitive data

State whether instances may contain secrets, credentials, tokens, personal data, password hashes, or other sensitive data.

### Tests

Describe tests that prove valid states, invalid states, invariants, and normalization.
```

For an HTTP endpoint or command:

```markdown
## Endpoint or Command: `<name>`

### Purpose

Describe what this endpoint or command does.

### Preconditions

- Required authentication, authorization, input shape, headers, environment, flags, files, external services, or database state.

### Postconditions

- Response guarantees, persisted changes, emitted events, logs, cache updates, files created, or cleanup completed.

### Inputs

Describe route params, body, query params, command flags, stdin, files, or environment values.

### Outputs

Describe HTTP status, response body, stdout, stderr, files, or process exit code.

### Errors

Describe validation failures, authorization failures, dependency failures, and expected error responses.

### Side effects

Describe storage writes, network calls, emitted events, cache updates, logs, metrics, or state mutation.

### Tests

Describe tests that prove the endpoint or command contract.
```

### Example per-file contract style

Use this level of detail when documenting definitions:

```markdown
# `src/users/UserRepository.ts`

## File invariants

- Deleted users are never returned by normal query methods.
- Password hashes are never exposed in returned DTOs.
- User IDs are immutable after creation.
- Email addresses are stored normalized to lowercase.

# `UserRepository`

## Kind

Repository class.

## Purpose

Provides storage access for user records while enforcing user visibility and normalization rules.

## Invariants

- Deleted users are never returned by normal query methods.
- Password hashes are never exposed in returned DTOs.
- User IDs are immutable after creation.
- Email addresses are stored normalized to lowercase.

## Method: `createUser(input)`

### Purpose

Create a new active user record.

### Preconditions

- `input.email` is present and valid.
- `input.email` is not already used by another active user.
- `input.passwordHash` is already hashed.
- Caller has already validated request permissions.

### Postconditions

- A new user record exists in storage.
- The returned user has an assigned `id`.
- The returned user does not include `passwordHash`.
- The stored email is lowercase.
- A `UserCreated` event is emitted.

### Errors

- Rejects or throws a duplicate-email error if an active user already uses the normalized email.
- Rejects or throws a validation error if required input is missing.

### Side effects

- Writes one user record to storage.
- Emits one `UserCreated` event after the record is created.

## Method: `findByEmail(email)`

### Purpose

Find an active user by normalized email address.

### Preconditions

- `email` is non-empty.
- `email` can be normalized to a valid email address.

### Postconditions

- Returns the matching active user, or `null`.
- Never returns soft-deleted users.
- Returned object does not include password hash.

### Errors

- Returns or throws a validation error for invalid email input, according to the project error-handling convention.

### Side effects

- Reads from user storage only.
- Does not write to storage or emit events.
```

---

## Step 7: Handle existing codebases

If the user provides an existing codebase, scan the codebase and convert it into markdown descriptions.

Do not modify the codebase.

Ignore generated, vendored, temporary, dependency, cache, and local editor directories such as:

```text
.git/
node_modules/
vendor/
dist/
build/
coverage/
.cache/
.next/
.nuxt/
target/
venv/
.venv/
__pycache__/
.pytest_cache/
.mypy_cache/
.idea/
.vscode/
```

Also ignore files that are clearly binary assets unless they are central to the project behavior.

For each source file:

1. Determine its purpose.
2. Identify its file-level invariants.
3. Identify its exports.
4. Identify important internal definitions.
5. Identify imports and dependencies.
6. Identify side effects.
7. Identify error behavior.
8. Identify testing implications.
9. For every class, function, method, variable, constant, type, interface, component, endpoint, command, schema, model, and event, document invariants, preconditions, and postconditions.
10. Create the corresponding `.md` file under `files/`.

If the existing implementation is unclear or appears buggy, document the current behavior and add a note under:

```markdown
## Notes and uncertainties
```

Do not silently invent behavior that contradicts the code.

---

## Step 8: Handle fresh projects

If this is a fresh start from a product idea or specification:

1. Read the user’s requirements.
2. Choose or confirm the technology stack.
3. Design the project structure.
4. Create `STACK.md`.
5. Create `PROJECT.md`.
6. Create `FILE_TREE.md`.
7. Create `API_REFERENCE.md`.
8. Create one markdown file per intended code file.
9. In every per-file markdown file, document every class, function, method, variable, constant, type, interface, component, endpoint, command, schema, model, and event using contract-style sections.
10. Create `TEST_PLAN.md`.
11. Create `IMPLEMENTATION_NOTES.md`.

The design should be practical, minimal, and coherent.

Do not over-engineer unless the user asks for a large or enterprise-grade system.

Prefer a small number of focused files over a large, fragmented architecture.

---

## Step 9: Handle existing markdown directories

If the markdown directory already exists:

1. Read `STACK.md`.
2. Read `PROJECT.md`, if present.
3. Read `API_REFERENCE.md`.
4. Read `FILE_TREE.md`.
5. Scan all files under `files/`.
6. Apply the user’s requested changes.
7. Update affected per-file markdown files.
8. Update `API_REFERENCE.md`.
9. Update `FILE_TREE.md` if files were added, removed, or renamed.
10. Update `TEST_PLAN.md` if behavior changed.
11. Ensure every updated per-file markdown file includes contract-style documentation for every important class, function, method, variable, constant, type, interface, component, endpoint, command, schema, model, and event.
12. Preserve existing useful decisions unless they conflict with the new requirements.

Never update a per-file markdown file without checking whether `API_REFERENCE.md` also needs to change.

Never update `API_REFERENCE.md` without checking whether one or more per-file markdown files also need to change.

---

## Step 10: Create `TEST_PLAN.md`

Create a test plan for the future implementation.

Use this structure:

```markdown
# Test Plan

## Test framework

## Test file structure

## Unit tests

## Integration tests

## End-to-end tests

## Fixtures and test data

## Mocking strategy

## Invariant tests

## Precondition tests

## Postcondition tests

## Error cases

## Edge cases

## Side-effect tests

## Commands to run tests

## Coverage expectations
```

For each important API, function, class, method, component, endpoint, command, type, model, or stateful variable, describe the expected tests.

Tests should prove invariants, preconditions, postconditions, error conditions, edge cases, and side effects.

---

## Step 11: Create `IMPLEMENTATION_NOTES.md`

Create an implementation notes file for future coding agents.

Use this structure:

```markdown
# Implementation Notes

## Important architecture decisions

## Contract documentation rules

## Ordering constraints

## Cross-file dependencies

## Files that should be implemented first

## Files that depend on generated types or schemas

## Known risks

## Assumptions

## Things not to do
```

This file should help the future `parse-md-dir` workflow implement the project in a safe order.

The `Contract documentation rules` section should summarize how implementation agents should use invariants, preconditions, and postconditions from the per-file markdown specs.

---

## Step 12: Validate the markdown directory

Before finishing, validate the markdown directory.

Check that:

- `STACK.md` exists.
- `PROJECT.md` exists.
- `API_REFERENCE.md` exists.
- `FILE_TREE.md` exists.
- `TEST_PLAN.md` exists.
- `IMPLEMENTATION_NOTES.md` exists.
- Every intended code file in `FILE_TREE.md` has a matching markdown file under `files/`.
- Every markdown file under `files/` corresponds to an intended code file in `FILE_TREE.md`.
- Every public export described in a per-file markdown file appears in `API_REFERENCE.md`.
- Every API reference entry points to a real intended file.
- Every important class, function, method, variable, constant, type, interface, component, endpoint, command, schema, model, and event has invariants, preconditions, and postconditions where applicable.
- Variables and constants include type or shape, initialization, mutation rules, and sensitive-data status.
- Tests are described for important behavior.
- Tests cover invariants, preconditions, postconditions, errors, edge cases, and side effects.
- There are no obvious contradictions between files.
- Any uncertainty is explicitly documented.

If validation fails, fix the markdown directory before finishing.

---

## Step 13: Report the result

At the end, summarize:

```markdown
Created markdown directory at: <path>

Files created or updated:

- STACK.md
- PROJECT.md
- API_REFERENCE.md
- FILE_TREE.md
- TEST_PLAN.md
- IMPLEMENTATION_NOTES.md
- files/...

Validation summary:

- Number of intended code files
- Number of markdown file specs
- Number of API reference entries
- Number of documented contract sections
- Known assumptions or uncertainties
```

Do not claim the future codebase has been implemented.

Only claim that the markdown source directory has been created or updated.

---

# Optional subagent workflow

If subagents are available, use them for large projects.

## Existing codebase mode

Spawn one subagent per source file or small group of related files.

Give each subagent this prompt:

```text
You are a helpful coding documentation assistant. Your task is to describe one existing code file as a markdown specification for future reimplementation.

Do not modify the codebase.

Read the code file at:

<code-file-path>

Create or update the markdown file at:

<markdown-file-path>

Use the project-wide stack file at:

<STACK.md path>

Use the project-wide API reference at:

<API_REFERENCE.md path>

Your markdown file must describe the file's purpose, responsibilities, file invariants, exports, internal definitions, dependencies, behavior, errors, side effects, data structures, configuration, tests, edge cases, and implementation notes.

For every class, function, method, variable, constant, type, interface, component, endpoint, command, schema, model, and event in this file, document invariants, preconditions, postconditions, errors, side effects, and tests.

Only document this one code file. Do not document unrelated files.
```

After subagents finish, merge their findings into `API_REFERENCE.md`, `FILE_TREE.md`, and `TEST_PLAN.md`.

## Fresh project mode

Spawn one subagent per intended code file.

Give each subagent this prompt:

```text
You are a helpful software architecture documentation assistant. Your task is to write a markdown specification for one future code file.

Do not implement code.

The project stack is described at:

<STACK.md path>

The project API reference is described at:

<API_REFERENCE.md path>

The intended code file is:

<intended-code-file-path>

The markdown file you are creating is:

<markdown-file-path>

Write a detailed markdown specification for this one future code file. The file should be specific enough that another coding agent can later implement the file without making major architecture decisions.

For every class, function, method, variable, constant, type, interface, component, endpoint, command, schema, model, and event that belongs in this file, document invariants, preconditions, postconditions, errors, side effects, and tests.

Only describe this one file. Do not create or modify real code files.
```

After subagents finish, review all markdown files for consistency.

---

# Recommended naming convention

Use this mapping:

```text
<intended-code-file-path> -> files/<intended-code-file-path>.md
```

Examples:

```text
src/index.ts -> files/src/index.ts.md
src/lib/parser.py -> files/src/lib/parser.py.md
app/api/users/route.ts -> files/app/api/users/route.ts.md
tests/parser.test.ts -> files/tests/parser.test.ts.md
```

Do not flatten paths.

Do not rename files unless necessary.

The markdown path should make it obvious which future code file it describes.

---

# Quality bar

A good markdown directory should let an implementation agent answer these questions for every file:

- Why does this file exist?
- What should it export?
- What should it import?
- What should each function, class, type, method, variable, or component do?
- What invariants must always hold?
- What preconditions must be true before each item is used?
- What postconditions must be true after each item succeeds?
- What errors should it handle?
- What side effects are allowed?
- What data must never be exposed?
- What tests should prove that it works?
- How does it connect to the rest of the project?

If those questions cannot be answered, the markdown directory is not detailed enough.
