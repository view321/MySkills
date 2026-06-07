---
name: edit-docs
description: Interview the user about documentation changes, descend from high-level decisions to details, and update Markdown docs.
---

# edit-docs

Use this skill when the user wants to change existing documentation, specifications, API references, or project behavior descriptions.

The goal is to update documentation safely before implementation. Do not change code while using this skill.

## Core Workflow

1. Read the existing documentation.
2. Restate the requested documentation change.
3. Identify which part of the documentation tree is affected.
4. Ask the most important question first.
5. After the user answers, descend into the next level of detail.
6. Continue until the change is specific enough to safely edit the docs.
7. Explain each proposed documentation change and why it belongs there.
8. Ask for approval before making large structural documentation changes.
9. Update only the affected Markdown sections.
10. List any software changes that may be required later.

## Interview Rules

- Ask questions from broad to narrow.
- Do not use a single large questionnaire unless the user asks for one.
- Start with decisions that affect goals, architecture, public APIs, data models, security, compatibility, or user-visible behavior.
- Preserve existing approved decisions unless the user explicitly changes them.
- When the user changes a high-level decision, revisit affected lower-level sections.
- If the user asks for a quick edit, make the smallest safe documentation change.
- If the user says “decide for me,” make a reasonable choice and record it under `Assumptions`.
- Do not invent requirements silently.
- Do not change code in this skill.

## Documentation Tree

Use this tree to decide where to ask questions and where to edit:

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

Do not ask about low-level details until the affected higher-level decision is clear.

## Editing Rules

- Update only relevant sections.
- Keep terminology consistent with the existing docs.
- Keep API references synchronized with behavioral descriptions.
- Mark unresolved questions clearly.
- Add implementation follow-up notes when doc changes imply code changes.
- If existing docs are inconsistent, report the inconsistency before making large edits.

## Output Format

After editing, report:

```md
## Documentation Updated

- ...

## Important Decisions

- ...

## Open Questions

- ...

## Possible Follow-Up Implementation Work

- ...
```

## Completion Criteria

This skill is complete when:

- the requested documentation change is reflected,
- related documentation sections remain consistent,
- unresolved questions are listed,
- assumptions are marked clearly,
- and any required implementation follow-up is identified.
