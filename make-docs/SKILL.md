---
name: make-docs
description: Interview the user about new software, recursively decompose the idea, and produce a Markdown specification with an API reference.
---

# make-docs

Use this skill when the user wants to design a new program, feature, API, app, library, or system before implementation.

The goal is to help the user turn an idea into a clear Markdown specification. Do not write implementation code while using this skill.

## Core Workflow

1. Restate the user's idea in plain language.
2. Split the idea into a small number of major chunks.
3. Explain why this split is useful.
4. Ask the user whether they approve the split.
5. After approval, take each chunk and split it into smaller subchunks.
6. Explain why each split is useful.
7. Ask the user to approve, reject, rename, merge, remove, or add chunks.
8. Repeat until each chunk is specific enough to document:
   - purpose
   - inputs
   - outputs
   - behavior
   - errors
   - dependencies
   - public API surface
9. Create or update a Markdown specification, usually at `docs/spec.md`.

## Interview Rules

- Ask about one level of the tree at a time.
- Always explain what you are proposing and why.
- Always ask for approval before descending to the next level of detail.
- Do not overwhelm the user with a long questionnaire.
- Ask the most important unresolved question first.
- If the user corrects the structure, update the tree before continuing.
- If the user says “decide for me,” make a reasonable choice and record it under `Assumptions`.
- Do not invent requirements silently.
- Do not implement code in this skill.

## Decomposition Tree

Use this tree as a default structure when interviewing the user:

```text
Project
├── Goals
├── Non-Goals
├── Users
├── Workflows
├── Architecture
│   ├── Modules
│   ├── Dependencies
│   └── Boundaries
├── Data
│   ├── Models
│   ├── Validation
│   └── Persistence
├── APIs
│   ├── Public API
│   ├── Internal API
│   └── Errors
├── Runtime
│   ├── Configuration
│   ├── State
│   ├── Logging
│   └── Failure Modes
└── Examples
```

Do not ask about low-level API signatures before the user has approved the higher-level architecture.

## Output File

Create or update:

```text
docs/spec.md
```

Use this structure unless the project already has a better documentation layout:

```md
# Project Specification

## Overview

## Goals

## Non-Goals

## Users

## Workflows

## Architecture

## Modules

## Data Models

## API Reference

### Public Functions / Classes / Endpoints

For each public API item:

- Name
- Purpose
- Signature
- Parameters
- Return value
- Errors
- Side effects
- Example usage

## Error Handling

## Configuration

## Security Considerations

## Examples

## Open Questions

## Assumptions
```

## Completion Criteria

This skill is complete when:

- the user has approved the final decomposition,
- all intended public behavior is documented,
- the API reference is present,
- unresolved questions are listed clearly,
- assumptions are marked explicitly,
- and `docs/spec.md` has been created or updated.
