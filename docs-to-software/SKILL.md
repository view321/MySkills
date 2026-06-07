---
name: docs-to-software
description: Implement or edit software according to existing Markdown documentation or specifications.
---

# docs-to-software

Use this skill when the user wants to implement, modify, or fix software based on existing documentation created by `make-docs`, `software-to-docs`, or `edit-docs`.

The documentation describes intended behavior. The code should be changed to match it.

## Core Workflow

1. Locate and read the relevant documentation.
2. Summarize the required behavior before editing code.
3. Identify affected files, modules, APIs, tests, commands, or endpoints.
4. Compare the current code with the documented behavior.
5. Report major doc/code conflicts before editing.
6. Make the smallest reasonable implementation plan.
7. Implement the required change.
8. Add or update tests where appropriate.
9. Run relevant checks if available.
10. Update documentation only if implementation reveals a necessary clarification.

## Implementation Rules

- Treat docs as the source of truth for intended behavior.
- Do not ignore the API reference.
- Do not change public behavior beyond what the docs require unless the user explicitly approves.
- Do not remove documented public APIs unless the docs require it.
- Preserve backwards compatibility unless the docs explicitly say otherwise.
- Prefer small, reviewable changes over large rewrites.
- If the docs are ambiguous and the answer affects architecture, public APIs, security, data models, compatibility, or user-visible behavior, ask the most important clarifying question first.
- If the ambiguity is minor, make a reasonable assumption and record it.
- Do not silently invent requirements.

## Conflict Handling

When docs and code disagree:

1. State the conflict clearly.
2. Explain whether the docs or code appear newer, if this can be inferred.
3. Ask the user before resolving major conflicts.
4. For minor conflicts, follow the docs and record the assumption.

## Testing Rules

- Add tests for new or changed behavior when a test framework exists.
- Update existing tests when documented behavior intentionally changes.
- Do not delete failing tests unless they conflict with approved documentation.
- Run the narrowest relevant test command first.
- If checks cannot be run, say so and explain why.

## Output Format

At the end, report:

```md
## Implemented

- ...

## Files Changed

- ...

## Tests / Checks

- ...

## Documentation Notes

- ...

## Remaining Questions

- ...
```

## Completion Criteria

This skill is complete when:

- the code matches the relevant documentation,
- public APIs match the documented API reference,
- important behavior is tested or manually verified,
- doc/code conflicts have been reported or resolved,
- and unresolved ambiguity is documented.
