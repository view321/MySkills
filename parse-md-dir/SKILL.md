---
name: parse-md-dir
description: Workflow to translate a markdown directory of implementation-ready .md file specifications into a real codebase
---

# The workflow

This skill translates a directory of markdown specifications into working code.

The markdown directory is the source of truth. It should contain `STACK.md`, project-level reference files, and one markdown specification per intended code file. Each file specification should describe the file, its exports, internal definitions, invariants, preconditions, postconditions, tests, and edge cases.

This skill may be used in two modes:

1. **Fresh start**: create a new codebase from a markdown directory.
2. **Existing project update**: update an existing codebase so it matches the markdown directory.

The skill must keep the project API reference synchronized with the markdown file specifications, implement or update the code, run tests, and fix failures until the implementation matches the specs.

---

# Inputs

The user should provide the path to a markdown directory.

The user may also provide:

- The target codebase directory.
- Whether the target project already exists.
- The maximum number of concurrent subagents.
- Whether subagents should be disabled.
- A preferred test command.
- A preferred package manager command.
- A list of files to include or exclude.
- A request to update only specific files.

If the target codebase directory is not specified, infer it from the markdown directory name or use:

```text
project
```

If the markdown directory does not contain enough information to infer the target codebase safely, ask the user for clarification unless the current task requires best-effort progress. In best-effort mode, make conservative assumptions and document them in the final report.

---

# User-configurable subagent controls

The user can control subagent usage with any clear wording, for example:

```text
Use at most 4 concurrent subagents.
Disable subagents.
Run sequentially.
Use one subagent at a time.
Use up to 10 workers.
```

Normalize this into the following settings:

```yaml
subagents_enabled: true
max_concurrent_subagents: 4
```

## Defaults

If the user does not specify subagent settings:

```yaml
subagents_enabled: true
max_concurrent_subagents: 4
```

For small projects with 1-3 target files, subagents are optional. For larger projects, use subagents when available.

## Disable rules

Subagents are disabled when the user says any of:

```text
disable subagents
no subagents
run sequentially
single agent only
```

or when:

```yaml
max_concurrent_subagents: 0
```

When subagents are disabled, perform all file implementation work sequentially in the main agent.

## Concurrency rules

When subagents are enabled:

- Never run more than `max_concurrent_subagents` subagents at the same time.
- Spawn at most one active subagent per target code file.
- A subagent may only modify the one code file assigned to it.
- A subagent must not modify shared files, package manifests, lockfiles, API references, test configs, or unrelated code files unless explicitly assigned that exact file.
- The main agent is responsible for scaffolding, dependency setup, API reference updates, test execution, integration fixes, and final validation.
- If two files are tightly coupled, implement the dependency first or run those files sequentially.

---

# Expected markdown directory structure

Prefer this structure:

```text
markdown-directory/
  STACK.md
  PROJECT.md
  API_REFERENCE.md
  FILE_TREE.md
  TEST_PLAN.md
  IMPLEMENTATION_NOTES.md
  files/
    <intended-code-path>.md
```

Examples:

```text
files/src/index.ts.md
files/src/auth/session.ts.md
files/tests/auth/session.test.ts.md
```

The markdown path should map to the intended code file by removing the leading `files/` segment and the final `.md` extension:

```text
files/src/auth/session.ts.md -> src/auth/session.ts
```

If a per-file markdown spec contains a top-level heading like:

```markdown
# `src/auth/session.ts`
```

that heading may be used to confirm the target code path.

If the heading and path mapping conflict, stop and resolve the conflict before implementing that file. If best-effort progress is required, prefer the explicit heading and report the conflict.

---

# Contract-style documentation format

Every per-file markdown specification should use contract-style documentation for each important file, class, function, method, variable, constant, type, interface, endpoint, command, schema, model, component, hook, and event.

The implementation must preserve these contracts.

## File-level template

Each per-file markdown file should include sections like:

```markdown
# `<intended-code-file-path>`

## Purpose

## Responsibilities

## Non-responsibilities

## Invariants
- List conditions that must always hold for this file or module.

## Exports

## Internal definitions

## Dependencies

## Detailed behavior

## Error handling

## Side effects

## Data structures

## Configuration

## Testing requirements

## Edge cases

## Notes for implementation agent
```

## Class template

Each class should be documented like:

```markdown
## Class: UserRepository

### Purpose

### Invariants
- Deleted users are never returned by normal query methods.
- Password hashes are never exposed in returned DTOs.
- User IDs are immutable after creation.
- Email addresses are stored normalized to lowercase.

### Constructor

#### Preconditions
- Required dependencies are provided.
- Dependencies satisfy the interfaces described in `API_REFERENCE.md`.

#### Postconditions
- The instance is ready to serve method calls.
- No external side effects occur unless explicitly documented.

### Method: createUser(input)

#### Preconditions
- `input.email` is present and valid.
- `input.email` is not already used by another active user.
- `input.passwordHash` is already hashed.
- Caller has already validated request permissions.

#### Postconditions
- A new user record exists in storage.
- The returned user has an assigned `id`.
- The returned user does not include `passwordHash`.
- The stored email is lowercase.
- A `UserCreated` event is emitted.

### Method: findByEmail(email)

#### Preconditions
- `email` is non-empty.
- `email` can be normalized to a valid email address.

#### Postconditions
- Returns the matching active user, or `null`.
- Never returns soft-deleted users.
- Returned object does not include password hash.
```

## Function template

Each standalone function should be documented like:

```markdown
## Function: normalizeEmail(email)

### Purpose

### Preconditions
- `email` is a non-empty string.
- `email` is expected to represent an email address.

### Postconditions
- Returns a lowercase, trimmed email address.
- Throws or returns a validation failure for invalid email input, according to the file spec.
- Does not mutate external state.

### Edge cases
- Leading and trailing whitespace.
- Mixed-case domains.
- Invalid email formats.
```

## Variable or constant template

Each important variable or constant should be documented like:

```markdown
## Constant: DEFAULT_SESSION_TTL_MS

### Purpose

### Invariants
- Value is a positive integer.
- Value is expressed in milliseconds.
- Value is used only as a fallback when no explicit configuration is provided.

### Usage
- Used by `createSession` when session TTL is absent from configuration.
```

## Type, interface, schema, or model template

Each type, interface, schema, or model should be documented like:

```markdown
## Interface: CreateUserInput

### Purpose

### Fields
- `email`: user email before normalization.
- `passwordHash`: already-hashed password.

### Invariants
- `passwordHash` is never a plaintext password.
- `email` is normalized before persistence.

### Validation rules
- `email` must be syntactically valid.
- `passwordHash` must be non-empty.
```

## Endpoint or command template

Each endpoint or command should be documented like:

```markdown
## Endpoint: POST /users

### Purpose

### Preconditions
- Request body satisfies the create-user schema.
- Caller is authorized to create users.

### Postconditions
- A new user is persisted.
- Response body excludes secret fields.
- Returns the documented HTTP status code.

### Error cases
- Invalid input returns `400`.
- Duplicate active email returns `409`.
- Unauthorized caller returns `401` or `403`, as specified.
```

---

# Contract implementation rules

Contract sections are normative.

When translating markdown into code:

- Implement behavior so all listed invariants remain true.
- Enforce preconditions at the correct boundary.
- Guarantee postconditions after successful execution.
- Add runtime validation when the spec requires the code to reject invalid input.
- Do not expose sensitive fields if an invariant or postcondition forbids it.
- Normalize, transform, persist, emit events, log, or perform side effects exactly where the spec says to do so.
- Do not add undocumented side effects.
- Do not silently ignore documented error cases.
- Do not broaden public APIs beyond what the spec requires unless necessary for the stack or tests.
- Prefer simple, direct implementations over speculative abstractions.

If a precondition says the caller has already validated something, do not duplicate heavy validation unless the API reference, file spec, or stack conventions require defensive validation at that layer.

If a precondition describes what this function itself must verify, implement the validation in that function.

If the distinction is unclear, choose the safer boundary, document the assumption in the final report, and ensure tests cover the behavior.

---

# Source-of-truth precedence

Use this precedence order:

1. The user’s latest explicit instruction.
2. `STACK.md` for technology choices and tooling.
3. `API_REFERENCE.md` for project-wide API contracts.
4. Per-file markdown specs under `files/` for file-specific implementation details.
5. `PROJECT.md`, `FILE_TREE.md`, `TEST_PLAN.md`, and `IMPLEMENTATION_NOTES.md` for architecture and testing guidance.
6. Existing code, only when updating an existing project and only where the markdown specs are silent.

If `API_REFERENCE.md` conflicts with a per-file markdown spec, reconcile them before implementation.

- If the per-file spec is more specific and does not contradict the project architecture, update `API_REFERENCE.md` to match it.
- If the API reference is clearly the project-wide contract and the per-file spec is stale, update or interpret the per-file implementation work to match `API_REFERENCE.md` and report the inconsistency.
- If the conflict cannot be safely resolved, stop for clarification unless best-effort progress is required.

---

# Step 1: Read the stack

Read `STACK.md` from the markdown directory.

Use it to determine:

- Programming language.
- Runtime.
- Frameworks.
- Package manager.
- Test framework.
- Build tools.
- Linting and formatting tools.
- Database or persistence choices.
- External services.
- Environment variables.
- Deployment assumptions.

If `STACK.md` is missing, infer the stack from the markdown specs and any existing project files. Create or update `STACK.md` only if the user allows documentation updates or if this is necessary to proceed.

Correct spelling and obvious metadata issues when updating generated documentation, for example `preffered` -> `preferred` and `Refernce` -> `Reference`.

---

# Step 2: Scan the markdown directory

Scan all relevant `.md` files in the markdown directory.

Read, when present:

```text
STACK.md
PROJECT.md
API_REFERENCE.md
FILE_TREE.md
TEST_PLAN.md
IMPLEMENTATION_NOTES.md
files/**/*.md
```

Ignore non-source notes unless the user says they are part of the implementation contract.

For each per-file markdown spec, extract:

- Intended code file path.
- Purpose.
- Responsibilities and non-responsibilities.
- Exports.
- Internal definitions.
- Classes.
- Constructors.
- Methods.
- Functions.
- Variables and constants.
- Types and interfaces.
- Schemas and models.
- Components and hooks.
- Endpoints and commands.
- Events.
- Invariants.
- Preconditions.
- Postconditions.
- Error handling.
- Side effects.
- Dependencies.
- Configuration.
- Testing requirements.
- Edge cases.
- Implementation notes.

Build an internal implementation plan from this scan before writing code.

---

# Step 3: Validate and update the API reference

`API_REFERENCE.md` is the project-wide source of truth for public and internal APIs.

## If `API_REFERENCE.md` exists

Update it according to the markdown file specs.

Preserve useful existing information, but correct anything that conflicts with the current per-file specs or the user’s latest instructions.

## If `API_REFERENCE.md` does not exist

Create it from the markdown file specs before implementing code.

The API reference should document every important exported or externally consumed item.

Use this structure:

```markdown
# API Reference

## Modules

## Classes

## Functions

## Methods

## Types and interfaces

## Variables and constants

## Components and hooks

## Commands

## HTTP endpoints

## Events

## Database models and schemas

## Configuration values

## Environment variables

## Errors

## External integrations
```

Each API entry should include contract sections when applicable:

```markdown
## `<name>`

### Kind
Function, class, method, type, component, endpoint, command, constant, schema, event, etc.

### Defined in
Path to the intended code file.

### Purpose

### Signature

### Parameters

### Returns

### Invariants

### Preconditions

### Postconditions

### Behavior

### Errors

### Side effects

### Dependencies

### Used by

### Testing requirements
```

Do not start implementation until the API reference and per-file specs are consistent enough to guide code generation.

---

# Step 4: Determine the target codebase state

Determine whether this is a fresh start or an existing project update.

## Existing project update

Use this mode when the target codebase directory exists and contains relevant project files.

The goal is to update the existing implementation to match the markdown directory.

## Fresh start

Use this mode when the target codebase directory does not exist or is empty.

The goal is to create and implement a new project from the markdown directory.

If uncertain, inspect the target directory.

Never delete an existing project directory just because it does not match the markdown directory.

---

# Step 5: Prepare the target file map

Create a target file map from `FILE_TREE.md` and `files/**/*.md`.

For each target file, record:

```text
markdown spec path
intended code path
whether the code file already exists
primary exports
dependencies on other project files
testing requirements
risk level
```

Skip files only when:

- The user explicitly excluded them.
- The markdown spec marks them as non-implementation documentation.
- They are generated files that should not be manually edited.

If `FILE_TREE.md` is missing or incomplete, reconstruct it from `files/**/*.md` and update `FILE_TREE.md` when documentation updates are allowed.

---

# Step 6: Scaffold the project when needed

In fresh-start mode:

1. Create the target codebase directory.
2. Scaffold the project according to `STACK.md`.
3. Create package manifests, config files, test config, lint config, source directories, and test directories as needed.
4. Do not write business logic during scaffolding.
5. After scaffolding, implement each target file according to its markdown spec.

Examples of scaffolding files include:

```text
package.json
tsconfig.json
vite.config.ts
eslint.config.js
pyproject.toml
requirements.txt
pytest.ini
Cargo.toml
go.mod
Dockerfile
.env.example
```

Only create files that are appropriate for the selected stack and described or implied by the markdown directory.

---

# Step 7: Implement or update code files

For every target code file:

1. Read the full `API_REFERENCE.md`.
2. Read the corresponding per-file markdown spec.
3. Read the existing code file if it exists.
4. Compare existing code to the API reference and per-file spec.
5. Implement or update the code file to satisfy both.
6. Preserve useful existing code when it does not conflict with the markdown specs.
7. Remove or refactor behavior that violates invariants, preconditions, postconditions, or documented side-effect boundaries.
8. Add or update tests described by the spec.

The implementation must document or encode contracts appropriately for the stack.

Depending on language and project style, this may include:

- Runtime validation.
- Type definitions.
- Assertions.
- Guard clauses.
- Error classes.
- Schema validation.
- Access modifiers.
- Unit tests.
- Integration tests.
- Docstrings or comments for non-obvious invariants.

Avoid adding comments that merely repeat obvious code. Do add comments where they preserve important contract reasoning that is not otherwise obvious.

---

# Step 8: Subagent workflow for existing project updates

Use this workflow when subagents are enabled.

Spawn one subagent per target code file, bounded by `max_concurrent_subagents`.

Use this prompt template:

```text
You are a helpful coding assistant who updates one existing code file.

Your source of truth is the project API reference and the markdown specification for the one file assigned to you.

Project stack file:
<STACK.md path>

Project API reference:
<API_REFERENCE.md path>

Markdown file specification:
<markdown-file-path>

Code file to update:
<code-file-path>

Task:
Compare the existing code file against the API reference and the markdown specification. Update only this code file so it satisfies the documented purpose, responsibilities, exports, internal definitions, invariants, preconditions, postconditions, error handling, side effects, dependencies, configuration, edge cases, and testing requirements that apply to this file.

Rules:
- Do not touch any other code file.
- Do not edit API_REFERENCE.md.
- Do not edit STACK.md.
- Do not edit package manifests or lockfiles.
- Do not change public APIs unless the markdown spec or API reference requires it.
- Preserve useful existing implementation details that do not conflict with the spec.
- Treat contract sections as normative.
- If the spec is ambiguous, choose the smallest safe implementation and report the ambiguity.
```

If tests for that file live in a separate test file, assign the test file to its own subagent or handle it in the main agent. Do not let a source-file subagent edit an unassigned test file.

After all subagents complete, the main agent must review the combined diff for cross-file consistency.

---

# Step 9: Subagent workflow for fresh starts

Use this workflow when subagents are enabled.

After scaffolding, spawn one subagent per target code file, bounded by `max_concurrent_subagents`.

Use this prompt template:

```text
You are a helpful coding assistant who builds one code file from a markdown specification.

Your source of truth is the project API reference and the markdown specification for the one file assigned to you.

Project stack file:
<STACK.md path>

Project API reference:
<API_REFERENCE.md path>

Markdown file specification:
<markdown-file-path>

Code file to create:
<code-file-path>

Task:
Translate the markdown specification into real code for this one file. Implement the documented purpose, responsibilities, exports, internal definitions, invariants, preconditions, postconditions, error handling, side effects, dependencies, configuration, edge cases, and testing requirements that apply to this file.

Rules:
- Only create or edit this one code file.
- Do not touch any other code file.
- Do not edit API_REFERENCE.md.
- Do not edit STACK.md.
- Do not edit package manifests or lockfiles.
- Do not invent public APIs that are not in the API reference or markdown spec.
- Treat contract sections as normative.
- If the spec is ambiguous, choose the smallest safe implementation and report the ambiguity.
```

If dependencies require a certain order, schedule dependency files first.

Examples:

- Types before services.
- Config before modules that read config.
- Error classes before modules that throw them.
- Database models before repositories.
- Repositories before services.
- Services before controllers or routes.
- Test utilities before tests.

---

# Step 10: Sequential workflow when subagents are disabled

When subagents are disabled, implement files one by one in dependency order.

For each file, follow the same rules used in the subagent prompt:

- Read `STACK.md`.
- Read `API_REFERENCE.md`.
- Read the per-file markdown spec.
- Read existing code if present.
- Update or create only the current file.
- Satisfy all documented contracts.
- Record ambiguities for the final report.

Sequential mode should be used when the user asks for deterministic, single-agent editing or when concurrency would create unsafe cross-file conflicts.

---

# Step 11: Implement tests

Use `TEST_PLAN.md`, per-file `Testing requirements`, and API reference `Testing requirements` to create or update tests.

Tests should verify:

- Public APIs exist with the documented names and signatures.
- Invariants hold across normal and edge-case behavior.
- Preconditions are enforced at the correct layer.
- Postconditions hold after successful operations.
- Documented error cases occur as specified.
- Sensitive data is not exposed.
- Side effects happen exactly as documented.
- Undocumented side effects do not occur.
- Integration boundaries behave as expected.

If a markdown spec describes a test file under `files/tests/...`, implement it like any other target code file.

If tests are required but no test file spec exists, create the smallest appropriate test files for the selected stack and update the final report to mention that tests were inferred.

---

# Step 12: Install dependencies when needed

Use the package manager from `STACK.md`.

Examples:

```text
npm install
pnpm install
yarn install
pip install -r requirements.txt
poetry install
cargo fetch
go mod tidy
```

Do not add dependencies unless:

- They are specified in `STACK.md`.
- They are required by the markdown specs.
- They are already part of the existing project.
- They are necessary for the test framework or scaffold selected by the stack.

Prefer standard library functionality when it satisfies the spec.

If a new dependency is necessary, update the appropriate manifest and explain why in the final report.

---

# Step 13: Run validation commands

Run the appropriate commands for the stack.

Prefer commands specified by the user or `STACK.md`.

If no commands are specified, infer common commands from the project files.

Examples:

```text
npm test
npm run test
npm run lint
npm run build
pnpm test
pnpm lint
pnpm build
pytest
go test ./...
cargo test
mvn test
gradle test
```

Run tests after implementation.

When possible, also run type checks, linting, or builds.

---

# Step 14: Fix failures and rerun tests

If tests, type checks, linting, or builds fail:

1. Read the failure output.
2. Identify the smallest fix that satisfies the markdown contracts.
3. Modify only the files needed to fix the failure.
4. Rerun the failing command.
5. Repeat until the issue is resolved or a blocker is reached.

Do not weaken tests to make them pass unless the tests contradict the markdown specs. If a test contradicts the specs, update the test to match the specs and report the change.

Do not remove contract checks to hide failures.

---

# Step 15: Final consistency validation

Before finishing, verify:

- The codebase matches `STACK.md`.
- `API_REFERENCE.md` reflects the markdown specs.
- Every target file from `files/**/*.md` has a corresponding code file unless intentionally skipped.
- Every implemented export appears in the API reference when it is public or cross-file internal API.
- Every documented class, function, method, variable, constant, type, endpoint, command, schema, model, component, hook, and event is implemented or explicitly marked as deferred.
- Invariants, preconditions, and postconditions are implemented or tested.
- Tests exist for important behavior.
- Validation commands have been run.
- Any failed command, skipped file, inferred behavior, or unresolved ambiguity is reported honestly.

---

# Final response format

End with a concise implementation report.

Use this structure:

```markdown
Implemented codebase at: <target-codebase-path>

Markdown source directory: <markdown-directory-path>

Subagent settings:
- subagents_enabled: <true|false>
- max_concurrent_subagents: <number>

Files created or updated:
- <path>
- <path>

API reference:
- Created or updated: <yes|no>
- Path: <path>

Validation commands run:
- `<command>`: passed
- `<command>`: failed: <brief reason>

Contract coverage summary:
- Files implemented from markdown specs: <number>
- Classes documented and implemented: <number>
- Functions/methods documented and implemented: <number>
- Variables/constants/types documented and implemented: <number>
- Tests added or updated: <number>

Assumptions or unresolved issues:
- <item>
```

Do not claim that tests passed unless they were actually run and passed.

Do not claim that every contract was implemented if any contract was skipped or ambiguous.

---

# Quality bar

A successful run of this skill produces a codebase where:

- The implementation follows the selected stack.
- The code files match their markdown specifications.
- The API reference is synchronized with the file specs.
- Public APIs are stable and documented.
- Invariants are preserved.
- Preconditions and postconditions are respected.
- Sensitive fields and forbidden side effects are handled correctly.
- Tests verify the most important contracts.
- The final report clearly states what changed and what was validated.
