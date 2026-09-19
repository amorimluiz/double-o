---
name: 00-loop
description: Run the full spec-driven development loop for a feature — generate PRD, TechSpec, and tasks behind approval gates, choose an isolated worktree workspace with per-worktree dependency isolation, execute tasks one per iteration with TDD, verification, and atomic commits, then run the applicable QA steps and a deep review, resuming from state.yml. Use when the user asks to build a feature end-to-end from a prompt, or to resume or continue an SDD run. Do not use for one-off edits or single tasks without a spec.
---

# Agentic SDD Loop

Entry point for the spec-driven pipeline. One prompt describes the goal; this skill drives `.sdd/<slug>/` from an empty directory through execution, QA, and review, stopping at each gate for the user's approval. Every step runs in this session; specialized sub-skills provide the stage procedures.

<HARD-GATE>
- Never advance past a gate without explicit user approval.
- Never mark a task complete without its tests and verification commands passing on real output.
- Never write `state.yml` outside a checkpoint: stage transitions, task completion, blockers.
</HARD-GATE>

## Bootstrap

1. Resolve the repo root and the slug (kebab-case, from the prompt or the user). Ask once when ambiguous.
2. Directory: `.sdd/<slug>/`. Create it when missing; artifacts are `prd.md`, `techspec.md`, `tasks.md`, `task-NN.md`, `state.yml`. QA and review outputs live where their skills define them.
3. Ensure `.sdd/` and `.worktrees/` are ignored: add both entries to the project's `.gitignore` (create the file when missing) unless already covered. Specs and worktrees are local by design.
4. `state.yml`: create from `references/state-schema.md` with `slug`, `goal`, `language`, `step`, `created_at`. When the file already exists, resume instead of resetting.

## Pipeline

Read `state.yml` first, then route by `step`:

| Step | Action |
| --- | --- |
| `prd` | Activate the `00-prd` skill; on its completion, gate. |
| `techspec` | Activate the `00-techspec` skill; on its completion, gate. |
| `tasks` | Activate the `00-tasks` skill; on its completion, gate. |
| `workspace` | Run the Workspace Gate below. It decides the execution workspace and provisions dependency isolation, then sets `step: execute`. |
| `execute` | Run the Execution Loop below. |
| `qa-report` | Run the `qa-report` skill. When it completes, set `artifacts.qa-report: done` and gate. Its applicability rules decide scope: no user-visible surface gets a recorded no-work disposition, not a fabricated run. |
| `qa-execution` | Run the `qa-execution` skill. When it completes, set `artifacts.qa-execution: done` and gate. It owns the live sessions and browser evidence for the journeys `qa-report` planned. |
| `review` | Run the `deep-review` skill over the branch diff. When it completes, set `artifacts.review: done` and gate. |
| `blocked` | Report the recorded blocker and stop. |
| `done` | Report the closed run; ask before re-opening. |

- Skip a step whose artifact is `done` or `skipped`; jump straight to the next pending one. The `workspace` step is a decision, not an artifact: skip it once `workspace.mode` is recorded.
- QA applicability: after `execute`, the QA steps exist only when the delivered change touches an interface a user or caller exercises — UI, HTTP/UDS API, CLI, or SDK. With no such surface, set `artifacts.qa-report` and `artifacts.qa-execution` to `skipped`, state the reason in `next`, and jump to `review`.
- Gate: report the artifact path, summarize the top decisions, ask for approval through the interactive question tool, and stop. On approval, advance `step` and continue the same turn.
- A rejected artifact goes back to its skill (or to the fixes the user requested) before the step re-runs; never patch it silently. Code fixes the QA or review steps surface go through `execute`.
- Further steps (ship, deploy) plug in the same way: extend the `step` values in `state.yml` and add the matching route here.

## Workspace Gate

Runs at the `workspace` step, before the first execution iteration. Its job is to give parallel runs and parallel tasks somewhere isolated to execute. Procedure and commands live in `references/workspace-isolation.md`; this section owns the decisions.

1. Read the tasks' shared-dependency notes (`00-tasks` records them per task) and scan for stateful services the run mutates — database, cache, broker, object store, search. This is the `isolation` need.
2. Ask through the interactive question tool: execute in a **worktree** (`.worktrees/<slug>`, own branch) or in the **current checkout**. Recommend the worktree when there is more than one task or any isolation need; it is what makes concurrent runs safe. On a resume where `workspace.mode` is already recorded, skip the question.
3. On worktree: provision per `references/workspace-isolation.md` — create the worktree, copy `.sdd/<slug>/` into it, and when there is an isolation need also assign the compose project, port block, and cloned database, and write the managed env override.
4. On checkout: record `workspace.mode: checkout`; with an isolation need, state that concurrent runs will collide and proceed only on confirmation.
5. Record the outcome in `state.yml` (`workspace` block), set `step: execute`, and continue the same turn.

The worktree is the unit of isolation; dependency isolation exists so two worktrees do not fight over one port, container, or database. Never run an isolated run's migrate, seed, or test against the primary checkout's dependencies.

## Execution Loop

One task per iteration. Read `tasks.md` and the task file before starting. All commands run from the resolved workspace identified in `state.yml` (`workspace.path` when set, otherwise the current checkout), with the workspace's env override applied for any compose, migration, seed, or test command.

1. Pick the first task whose `depends-on` are all completed; on ties, the one owning the earliest outcome.
2. Checkpoint: set `tasks.current`, `iteration + 1`, `updated_at`; set the task `status: in_progress`.
3. Work test-first: write the task's test cases, watch them fail for the right reason (RED), implement the minimal code (GREEN), refactor with tests staying green. Follow the project's rules and the global testing and code standards. Coverage floor: 80% unit coverage of new and changed code.
4. Verify: run the task's Verify commands plus the project's lint, typecheck, and test commands for the changed surfaces. Report real output. Repair a failing check inside this iteration; never waive it.
5. Track: tick the task's Subtasks, Tests, and Acceptance checkboxes, set `status: completed`, and update its row in `tasks.md`.
6. Commit: one atomic commit per task using the frontmatter's `commit` subject, a body naming the task file, and the `Assisted-by:` trailer required by the git rules. Stage only task-owned paths; never commit unrelated files.
7. Update `state.yml`: `tasks.done + 1`, `tasks.current: null`, `last_verify`, `last_commit`, `updated_at`.
8. Continue with the next iteration in the same turn. When every task is complete, advance `step` to `qa-report`; stop earlier only for a real blocker (`step: blocked`) or a user interrupt.

## Resume and Drift

On entry with an existing `state.yml`, reconcile before continuing — the files are the source of truth:

- Compare `tasks.done` against the task frontmatter statuses.
- Confirm `last_commit` exists in `git log`; the working tree must be clean or contain only the current step's work.
- Confirm the recorded workspace exists: `workspace.path` present on disk for a worktree, branch checked out, isolated compose project and database still up. Re-provision missing pieces per `references/workspace-isolation.md`; when a worktree is gone but its branch and commits survive, resume in the checkout and re-record `workspace.mode`.
- Repair the state file to match reality, then continue from the real step.

## Stops and Failures

- Every task completed and verified: advance to `qa-report` (or straight to `review` on a recorded QA skip) and keep the pipeline moving. The run reaches `done` only after the QA steps and the review gate pass; then report the summary — tasks executed, commits, verification status, workspace path and branch, isolated dependencies, anything left over. Offer teardown of the isolated stack and worktree (per `references/workspace-isolation.md`) behind a gate; never auto-delete.
- A real external blocker (missing permission, unavailable dependency, product decision without a safe default): record it in `blockers`, set `step: blocked`, stop, and report. Continue independent work before asking.
- No progress in two consecutive iterations on the same task: treat it as blocked and record the diagnosis.
- Never: weaken or delete a failing test, skip hooks with `--no-verify`, expand scope silently, commit unrelated changes, or rewrite completed history.

## Handoff

Each stage skill keeps its own artifact; this skill consumes them in order and owns `state.yml`. Invoking `00-loop` alone on an existing spec resumes from the current step; the individual skills stay invocable on their own.

Adapted from compozy's `cy-loop-tasks` skill (MIT, © 2026 NauckGroup LTDA), simplified for double-o.
