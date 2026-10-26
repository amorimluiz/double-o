---
name: 00-tasks
description: Decompose an approved TechSpec into a dependency-ordered task graph with TDD test assignments, writing tasks.md plus one task-NN.md per task. Use when a TechSpec exists and needs task breakdown, or when the 00-loop SDD pipeline reaches its Tasks step. Do not use for design, requirements, or implementation.
---

# Create Tasks

Produce `.sdd/<slug>/tasks.md` and its `task-NN.md` files: the execution graph consumed by `00-loop`. Reuse the TechSpec's decisions and file references; a new task does not require starting research from zero. When approved `design/` prototypes exist, the interface they fix is a contract — assign each screen or component to an owning task and reference the prototype file rather than re-deciding its layout or states.

## Workspace

- Directory: `.sdd/<slug>/` (same slug as the PRD).
- Read first: `techspec.md`, `prd.md`, and the approved `design/` prototypes when they exist. When the TechSpec is missing, stop and point to `00-techspec`.
- Artifacts: `tasks.md` (graph and status table), `task-NN.md` (one per task), `state.yml`.
- Language: reuse the language recorded in `state.yml`.

## State

Update `.sdd/<slug>/state.yml`; full schema in the sibling skill `00-loop` (`references/state-schema.md`). This skill owns:

- On start: `step: tasks`, `updated_at`.
- On completion: `artifacts.tasks: done`, `tasks.total`, `tasks.pending`, `step: workspace`, `updated_at`.
- On a real blocker: `step: blocked`, append the blocker.

## Decomposition Rules

1. Start from the requested scope and the TechSpec's sequencing; expand into code only when an ownership or contract question remains.
2. Put the earliest useful solution to the motivating problem first. A foundation task names its consumer and verifies its own boundary. Resolve a shared contract before dispatching its consumers, or keep the coupled work in one task.
3. Size by risk and ownership. A default of three to seven tasks guides the cut and never caps it; file count does not decide it.
4. Assign every behavior to exactly one owning task. Do not defer an explicit acceptance requirement to another task.
5. Every task carries its own test assignments — the project's TDD rule applies per task, never to a later phase.
6. Flag stateful shared dependencies. When a task touches a database, cache, broker, object store, or search cluster — especially when it mutates one through a migration, seed, fixture, or destructive test — record it in the task's Shared Dependencies section. The `00-loop` Workspace Gate uses these notes to isolate the dependency per worktree; a missed flag is a cross-run collision waiting to happen.

## Task File Rules

Use `references/task-template.md` for both frontmatter and body:

- Frontmatter owns `status`, `title`, `type`, `complexity`, `depends-on`, and the Conventional Commit subject `commit`.
- `depends-on` lists task IDs whose completion unblocks this one; keep it consistent with `tasks.md`.
- Tests list concrete cases — IDs with input, condition, and expected result — so the implementer writes them first (RED before GREEN).
- `Relevant Files` names exact paths with one clause each on why to read them.
- Omit empty or inapplicable sections; never leave placeholder text.

## Workflow

1. Read the inputs and choose outcomes, dependencies, and types.
2. Write one `task-NN.md` per task (zero-padded: `task-01.md`).
3. Write `tasks.md`: the goal, the dependency-ordered table (ID, title, type, complexity, depends-on, status), and the execution order.
4. Check consistency once: dependency cycles, orphan references, behavior ownership, and test coverage of every TechSpec component and boundary. A task with existing sufficient coverage records that evidence instead of inventing cases.
5. Update state, report the paths, and ask for approval. On approval, point to `00-loop` — or continue the `00-loop` pipeline when it invoked this skill.

## Error Handling

- TechSpec missing or an unresolved contract: report it concretely and stop; gather available context before asking.
- Inconsistent graph: repair the files, then re-check. Never silently drop a requested outcome.
- Update mode: regenerate only the tasks affected by changed inputs; preserve completed tasks unless their inputs changed.

Adapted from compozy's `cy-create-tasks` skill (MIT, © 2026 NauckGroup LTDA), simplified for double-o.
