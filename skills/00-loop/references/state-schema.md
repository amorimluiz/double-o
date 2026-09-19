# state.yml — Schema and Update Protocol

`.sdd/<slug>/state.yml` is the run's snapshot: one glance answers where the run is, what is left, and what is blocked. Keep it small, current, and honest.

## Fields

| Field | Type | Meaning |
| --- | --- | --- |
| `slug` | string | Directory name under `.sdd/`. |
| `goal` | string | One-line summary of the user's prompt. |
| `language` | `en` \| `pt` | Language of the generated artifacts. |
| `step` | `prd` \| `techspec` \| `tasks` \| `execute` \| `done` \| `blocked` | Current stage. Extensible: new steps plug in between `execute` and `done`. |
| `iteration` | int ≥ 0 | Completed execution iterations. |
| `created_at` / `updated_at` | RFC3339 UTC | First write / last write. |
| `artifacts.prd` / `.techspec` / `.tasks` | `pending` \| `done` | Artifact status. |
| `tasks.total` | int | Task count from `tasks.md`. |
| `tasks.done` | int | Completed tasks. |
| `tasks.current` | string \| null | Task being executed right now. |
| `tasks.pending` | int | Remaining tasks. |
| `last_verify` | `PASS` \| `FAIL` \| null | Result of the latest completed iteration's verification. |
| `last_commit` | short sha \| null | Commit of the last completed task. |
| `blockers` | list[string] | Real external blockers, with evidence. Empty when none. |
| `next` | string | Human-readable next action. |

## Example

```yaml
slug: photo-albums
goal: "Albums with upload, tags, and sharing"
language: en
step: execute
iteration: 7
created_at: 2026-09-19T18:00:00Z
updated_at: 2026-09-19T18:20:00Z
artifacts: {prd: done, techspec: done, tasks: done}
tasks: {total: 8, done: 2, current: task-03, pending: 5}
last_verify: PASS
last_commit: 3b194bd
blockers: []
next: "task-03 — upload endpoint"
```

## Update Protocol

Every writer updates on a checkpoint, never mid-task:

| Writer | Writes |
| --- | --- |
| `00-prd` | `artifacts.prd`, `step`, `updated_at` |
| `00-techspec` | `artifacts.techspec`, `step`, `updated_at` |
| `00-tasks` | `artifacts.tasks`, `tasks.total`, `tasks.pending`, `step`, `updated_at` |
| `00-loop` | `iteration`, `tasks.current/done/pending`, `last_verify`, `last_commit`, `blockers`, `step`, `updated_at` |

Invariants:

1. `step` only moves forward, except `blocked` and its resume back to the stage it interrupted.
2. `tasks.done + tasks.pending == tasks.total` at every write.
3. `last_verify: FAIL` is never written for an intermediate failure inside an iteration — only the iteration's final result or a proven blocker.
4. The file is a snapshot, not a log: history lives in git and in the task files. When `state.yml` conflicts with the filesystem, the filesystem wins and the state is repaired to match.
5. `done` is the only successful completion; it requires every task completed and verified.
