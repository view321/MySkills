---
name: docs-to-api
description: Turn existing documentation into a proposed API structure before implementation.
---

# docs-to-api

## Purpose

Use this skill to convert a docs/spec file into an API design proposal before writing code. The agent should show the structure and pseudocode so the user can find mistakes early.

## Workflow

1. Locate and read the relevant docs.
2. Summarize the intended behavior.
3. Propose the top-level API structure first: namespaces, packages, modules, services, or classes.
4. Explain why this structure fits the docs.
5. Ask the user to approve, rename, split, merge, remove, or add top-level pieces.
6. After approval, descend one level at a time:
   - namespaces
   - modules/classes
   - functions/methods
   - parameters
   - return values
   - errors
   - important variables/constants
7. At each level, show the proposed structure plus brief pseudocode.
8. Ask the user to spot mistakes before descending further.
9. Update the proposal from user feedback.
10. Stop when the API is clear enough for implementation.

## Strict Rules

- Do not implement code.
- Do not invent behavior unsupported by the docs.
- Mark assumptions clearly.
- Keep the design hierarchical and reviewable.
- Do not define function signatures before namespace/module approval.
- Prioritize public APIs over private internals.
- Ask the most important question first when docs are ambiguous.
- If the user says “decide for me,” choose reasonably and label it as an assumption.

## Output

Produce a Markdown API proposal with:

- source docs used
- namespace/module tree
- public classes/functions
- parameters and return values
- key variables/constants
- pseudocode for important flows
- assumptions
- open questions

## Completion Criteria

Complete when the user has reviewed the structure and pseudocode, and the API proposal is ready for `docs-to-software`.
