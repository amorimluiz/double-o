# Task File Template

Structure for `.sdd/<slug>/task-NN.md`. Write in the project's artifact language and keep each outcome, constraint, and test obligation in one place. Omit sections that would repeat another section.

```markdown
---
status: pending
title: [Task title]
type: [backend | frontend | infra | docs | ...]
complexity: [low | medium | high | critical]
depends-on: []
commit: "[type(scope): imperative subject]"
---

# Task NN: [Title]

## Outcome

[2-3 sentences: what a user, agent, or operator can newly do when this task merges, and why it matters in the context of the project.]

## Verify

- [Exact command or observation that proves the outcome, run from the repo root]

## Requirements

[Task-specific constraints. Link the accepted TechSpec section for shared requirements; do not copy generic coding or testing reminders.]

## Relevant Files

- `path/to/file` — [brief reason this file is relevant]
- `path/to/file` — [brief reason this file is affected]

## Shared Dependencies

[Stateful shared dependencies this task reads or mutates — database, cache, broker, object store, search — with the service name and the mutation (migration, seed, destructive test). State "None" when the task touches no shared stateful dependency. `00-loop` uses this to isolate the dependency per worktree.]

## Subtasks

- [ ] NN.1 [What to accomplish]
- [ ] NN.2 [What to accomplish]

## Tests

Write these cases first: RED before GREEN. Each case states input, condition, and expected result.

- [ ] [UT-NN / case name] — [component or behavior, input, expected result]
- [ ] [IT-NN / case name] — [boundary or flow, input, expected result]
- [ ] [E2E-NN / case name] — [journey, precondition, expected result]

[Reuse existing coverage by path and invariant instead of duplicating it; record the reused evidence when it already proves the behavior.]

## Acceptance

- [ ] Outcome observed through its real entry path
- [ ] Every test above passes
- [ ] Project checks pass for the changed surfaces (lint, types, build as applicable)

## Notes

[Optional: decisions taken during implementation, follow-ups discovered, links to the PRD/TechSpec sections that ground the task.]
