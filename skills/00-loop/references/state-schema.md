# state.yml — Schema and Update Protocol

`.sdd/<slug>/state.yml` is the run's snapshot: one glance answers where the run is, what is left, and what is blocked. Keep it small, current, and honest.

## Fields

| Field | Type | Meaning |
| --- | --- | --- |
| `slug` | string | Directory name under `.sdd/`. |
| `goal` | string | One-line summary of the user's prompt. |
| `language` | `en` \| `pt` | Language of the generated artifacts. |
| `step` | `prd` \| `techspec` \| `tasks` \| `workspace` \| `execute` \| `qa-report` \| `qa-execution` \| `review` \| `done` \| `blocked` | Current stage, in this order. Extensible: further steps plug in between `execute` and `done`. |
| `iteration` | int ≥ 0 | Completed execution iterations. |
| `created_at` / `updated_at` | RFC3339 UTC | First write / last write. |
| `artifacts.*` | `pending` \| `done` \| `skipped` | Artifact status. `skipped` is valid only for `qa-report` and `qa-execution`, and only on a recorded QA applicability decision. |
| `tasks.total` | int | Task count from `tasks.md`. |
| `tasks.done` | int | Completed tasks. |
| `tasks.current` | string \| null | Task being executed right now. |
| `tasks.pending` | int | Remaining tasks. |
| `workspace.mode` | `checkout` \| `worktree` | Where the run executes. Defaults to `checkout`; set at the Workspace Gate. |
| `workspace.path` | string \| null | Worktree path (`.worktrees/<slug>`) when `mode: worktree`. |
| `workspace.branch` | string \| null | Branch checked out in the worktree. |
| `workspace.isolation` | map \| null | Resolved dependency isolation: `compose_project`, `port_base`, `database`. Null when no stateful dependency needs isolating. |
| `workspace.isolated` | list[string] | Stateful dependencies isolated for this worktree (services, databases), with evidence. Empty when none. |
| `last_verify` | `PASS` \| `FAIL` \| null | Result of the latest completed iteration's verification. |
| `last_commit` | short sha \| null | Commit of the last completed task. |
| `blockers` | list[string] | Real external blockers, with evidence. Empty when none. |
| `next` | string | Human-readable next action. |

## Example

```yaml
slug: photo-albums
goal: "Albums with upload, tags, and sharing"
language: en
step: qa-execution
iteration: 7
created_at: 2026-09-19T18:00:00Z
updated_at: 2026-09-19T18:20:00Z
artifacts: {prd: done, techspec: done, tasks: done, qa-report: done, qa-execution: pending, review: pending}
tasks: {total: 8, done: 8, current: null, pending: 0}
workspace:
  mode: worktree
  path: .worktrees/photo-albums
  branch: feat/photo-albums
  isolation: {compose_project: double-o-photo-albums, port_base: 21230, database: app_photo_albums}
  isolated: ["postgres: app_photo_albums", "redis: separate logical DB"]
last_verify: PASS
last_commit: 3b194bd
blockers: []
next: "qa-execution — walk the album creation journey"
```

## Update Protocol

Every writer updates on a checkpoint, never mid-task:

| Writer | Writes |
| --- | --- |
| `00-prd` | `artifacts.prd`, `step`, `updated_at` |
| `00-techspec` | `artifacts.techspec`, `step`, `updated_at` |
| `00-tasks` | `artifacts.tasks`, `tasks.total`, `tasks.pending`, `step`, `updated_at` |
| `00-loop` | `iteration`, `tasks.current/done/pending`, `workspace`, `last_verify`, `last_commit`, `blockers`, `step`, `updated_at` |
| `00-loop` on behalf of `qa-report`, `qa-execution`, `deep-review` | `artifacts.qa-report` / `.qa-execution` / `.review` (`skipped` allowed for QA), `step`, `updated_at` |

The vendored stage skills do not know this schema; `00-loop` applies their state writes when each step completes.

Invariants:

1. `step` only moves forward in the order above. The only backward moves are `blocked` resuming the stage it interrupted and a rejected gate re-running its stage after the requested fixes — record the reason in `next`.
2. `tasks.done + tasks.pending == tasks.total` at every write.
3. `last_verify: FAIL` is never written for an intermediate failure inside an iteration — only the iteration's final result or a proven blocker.
4. The file is a snapshot, not a log: history lives in git and in the task files. When `state.yml` conflicts with the filesystem, the filesystem wins and the state is repaired to match.
5. `done` is the only successful completion; it requires every task completed and verified, both QA artifacts `done` or `skipped`, and the review artifact `done`.
6. `skipped` QA requires the applicability check: the executed change added no interface a user or caller exercises (UI, HTTP/UDS API, CLI, SDK). The reason is recorded in `next` at the skip.
7. `workspace.mode: worktree` requires `path` and `branch`; a stateful dependency the run mutates requires a matching `workspace.isolation` entry and a name in `workspace.isolated`. The workspace is never removed or torn down without the user's approval.
