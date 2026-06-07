---
name: software-to-docs
description: Create or update Markdown documentation by inspecting existing software code and tests.
---

# software-to-docs

Use this skill when the user wants to create, recover, or update documentation based on existing software.

The code describes current behavior. Document what the software actually does, not what it should do.

## Core Workflow

1. Inspect the repository structure.
2. Identify main entry points, public APIs, modules, commands, endpoints, configuration, and tests.
3. Read the relevant source files.
4. Infer current behavior from code and tests.
5. Create or update Markdown documentation.
6. Clearly separate confirmed behavior from assumptions.
7. List missing tests, unclear behavior, dead code, or undocumented public APIs.
8. Do not modify code unless the user explicitly asks.

## Documentation Rules

- Treat code as the source of truth for current behavior.
- Do not document desired behavior as if it already exists.
- Do not invent APIs, parameters, errors, commands, or examples.
- Prefer public interfaces over private implementation details.
- Include examples only when they can be confidently derived from code, tests, or existing docs.
- If behavior is unclear, mark it as unclear.
- If existing docs disagree with code, report the mismatch.
- Do not change software while using this skill.

## What To Inspect

Prioritize:

1. README files and existing docs.
2. Package manifests and project configuration.
3. Main application entry points.
4. Public modules, exported functions, classes, endpoints, commands, or SDK methods.
5. Tests and fixtures.
6. Error handling and validation logic.
7. Configuration and environment variables.
8. Persistence, network, filesystem, or external service boundaries.

## Output Files

Create or update whichever files fit the project best, commonly:

```text
docs/spec.md
docs/api.md
README.md
```

Recommended structure:

```md
# Software Documentation

## Overview

## Current Architecture

## Modules

## Public API Reference

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

## Commands / Entry Points

## Configuration

## Data Models

## Runtime Behavior

## Error Handling

## Examples

## Known Gaps

## Assumptions

## Doc/Code Mismatches
```

## Completion Criteria

This skill is complete when:

- current software behavior is documented,
- the public API reference is updated,
- assumptions are clearly marked,
- unclear behavior is listed,
- and mismatches between existing docs and code are reported.
