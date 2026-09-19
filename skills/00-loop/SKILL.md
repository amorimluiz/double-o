---
name: 00-loop
description: Run the full spec-driven development loop for a feature — generate PRD, TechSpec, and tasks behind approval gates, then execute tasks one per iteration with TDD, verification, and atomic commits, resuming from state.yml. Use when the user asks to build a feature end-to-end from a prompt, or to resume or continue an SDD run. Do not use for one-off edits or single tasks without a spec.
---

# Agentic SDD Loop

Entry point for the spec-driven pipeline. One prompt describes the goal; this skill drives `.sdd/<slug>/` from an empty directory to executed tasks, stopping at each gate for the user's approval.

<HARD-GATE>
- Never advance past a gate without explicit user approval.
- Never mark a task complete without its tests and verification commands passing on real output.
- Never write `state.yml` outside a checkpoint: stage transitions, task completion, blockers.
</HARD-GATE>

## Bootstrap

1. Resolve the repo root and the slug (kebab-case, from the prompt or the user). Ask once when ambiguous.
2. Directory: `.sdd/<slug>/`. Create it when missing; artifacts are `prd.md`, `techspec.md`, `tasks.md`, `task-NN.md`, `state.yml`.
3. Ensure `.sdd/` is ignored: add the entry to the project's `.gitignore` (create the file when missing) unless already covered. Specs are local by design.
4. `state.yml`: create from `references/state-schema.md` with `slug`, `goal`, `language`, `step`, `created_at`. When the file already exists, resume instead of resetting.

## Pipeline

Read `state.yml` first, then route by `step`:

| Step | Action |
| --- | --- |
| `prd` | Activate the `00-prd` skill; on its completion, gate. |
| `techspec` | Activate the `00-techspec` skill; on its completion, gate. |
| `tasks` | Activate the `00-tasks` skill; on its completion, gate. |
| `execute` | Run the Execution Loop below. |
| `blocked` | Report the recorded blocker and stop. |
| `done` | Report the closed run; ask before re-opening. |

- Skip a step whose artifact is `done`; jump straight to the next pending one.
- Gate: report the artifact path, summarize the top decisions, ask for approval through the interactive question tool, and stop. On approval, advance `step` and continue the same turn.
- A rejected artifact goes back to its skill in update mode; never patch it silently.
- More steps (QA, review, ship) plug in between `execute` and `done`: extend the `step` values in `state.yml` and add the matching route here.

## Execution Loop

One task per iteration. Read `tasks.md` and the task file before starting.

1. Pick the first task whose `depends-on` are all completed; on ties, the one owning the earliest outcome.
2. Checkpoint: set `tasks.current`, `iteration + 1`, `updated_at`; set the task `status: in_progress`.
3. Work test-first: write the task's test cases, watch them fail for the right reason (RED), implement the minimal code (GREEN), refactor with tests staying green. Follow the project's rules and the global testing and code standards. Coverage floor: 80% unit coverage of new and changed code.
4. Verify: run the task's Verify commands plus the project's lint, typecheck, and test commands for the changed surfaces. Report real output. Repair a failing check inside this iteration; never waive it.
5. Track: tick the task's Subtasks, Tests, and Acceptance checkboxes, set `status: completed`, and update its row in `tasks.md`.
6. Commit: one atomic commit per task using the frontmatter's `commit` subject, a body naming the task file, and the `Assisted-by:` trailer required by the git rules. Stage only task-owned paths; never commit unrelated files.
7. Update `state.yml`: `tasks.done + 1`, `tasks.current: null`, `last_verify`, `last_commit`, `updated_at`.
8. Continue with the next iteration in the same turn. Stop only when every task is complete (`step: done`), a real blocker appears (`step: blocked`), or the user interrupts.

## Resume and Drift

On entry with an existing `state.yml`, reconcile before continuing — the files are the source of truth:

- Compare `tasks.done` against the task frontmatter statuses.
- Confirm `last_commit` exists in `git log`; the working tree must be clean or contain only the current task's work.
- Repair the state file to match reality, then continue from the real step.

## Stops and Failures

- Every task completed and verified: set `step: done` and report the summary — tasks executed, commits, verification status, anything left over.
- A real external blocker (missing permission, unavailable dependency, product decision without a safe default): record it in `blockers`, set `step: blocked`, stop, and report. Continue independent work before asking.
- No progress in two consecutive iterations on the same task: treat it as blocked and record the diagnosis.
- Never: weaken or delete a failing test, skip hooks with `--no-verify`, expand scope silently, commit unrelated changes, or rewrite completed history.

## Handoff

Each generation skill keeps its own artifact; this skill consumes them in order and owns `state.yml`. Invoking `00-loop` alone on an existing spec resumes from the current step; the individual skills stay invocable on their own.

Adapted from compozy's `cy-loop-tasks` skill (MIT, © 2026 NauckGroup LTDA), simplified for double-o.
