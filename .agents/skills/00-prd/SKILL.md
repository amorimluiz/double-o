---
name: 00-prd
description: Create a Product Requirements Document for a feature through codebase/market research and interactive grilling. Use when starting a new feature or product, defining business requirements, or when the 00-loop SDD pipeline reaches its PRD step. Do not use for technical design, task breakdown, or implementation.
---

# Create PRD

Produce `.sdd/<slug>/prd.md`: the business-facing source of truth for WHAT users need, WHY it matters, and WHO they are. Downstream skills (`00-techspec`, `00-tasks`, `00-loop`) consume it for business rules, domain behavior, and product intent.

<HARD-GATE>
- Research before questions: enrich every PRD with codebase and market context.
- Questions before writing: the user shapes the PRD by answering clarifying questions, one at a time. Every PRD, however simple.
- Decide, then write: after the grilling converges, write the file directly. The user reviews the generated file and requests changes — no draft-approval loops.
</HARD-GATE>

## Workspace

- Repo root: resolve once (the current project's git root).
- Directory: `.sdd/<slug>/` — `<slug>` is kebab-case, derived from the feature name.
- Artifacts: `prd.md` (this skill), `techspec.md`, `design/`, `tasks.md`, `task-NN.md`, `qa/`, `review/`, `notes/`, `state.yml`. Every run artifact stays under `.sdd/<slug>/` and is never committed (full layout: `00-loop` → Bootstrap).
- Language: detect the project's artifact language from `AGENTS.md`, `CLAUDE.md`, or `README.md`; when unclear, ask once and record it. Default: English.

## State

Update `.sdd/<slug>/state.yml` at every completed step; full schema in the sibling skill `00-loop` (`references/state-schema.md`). This skill owns:

- On start: create the file when missing with `slug`, `goal`, `language`, `step: prd`, `created_at`.
- On completion: `artifacts.prd: done`, `step: techspec`, `updated_at`.
- On a real blocker: `step: blocked`, append the blocker.

## Workflow

1. Resolve the workspace and read `prd.md` when present. Update mode: preserve approved sections and mirror changes into Open Questions.
2. Discover context on two tracks before asking anything:
   - Codebase: search files, patterns, data models, and integration points related to the request. For research that spans independent areas, use the `agent-exploration` skill (parallel scoped explorers with written results); otherwise use the runtime's subagent tool or search directly.
   - Market: 3-5 web searches on competing products, conventions, and user expectations. When no web tool exists, state the limitation and proceed with the codebase track.
   Present the merged findings, 3-5 bullets per track, to the user.
3. Grill the requirements: read `references/question-protocol.md` and walk its decision tree branch by branch until every load-bearing product decision is resolved or parked in Open Questions with the user's consent.
4. Write `prd.md` using `references/prd-template.md`; fill every section with confirmed answers. Record significant product decisions in the Decisions section and every gap in Open Questions.
5. Hand off: report the file path and ask for approval. On approval, point to `00-techspec` — or continue the `00-loop` pipeline when it invoked this skill.

## Business Focus

The PRD owns WHAT, WHY, and WHO; HOW belongs to `00-techspec`. Translate technical feature names into user-experience questions:

- WRONG: "Should notifications use WebSockets or polling?"
- RIGHT: "Which events should trigger a notification to the user?"

KPIs, success metrics, timelines, and rollout phases have no downstream consumer — leave them out.

## Full Scope, One PRD

Capture the complete scope the user wants in this single document. Document size is never a reason to trim, defer, or stage anything — `00-tasks` decomposes the work later. A capability leaves the PRD only when the user decides against it, recorded in Non-Goals. YAGNI applies to invention: challenge features the user never asked for; keep every one they did. When the user adds scope mid-conversation, fold it in and continue.

## Error Handling

- Insufficient context for a section: record the gap in Open Questions instead of guessing.
- No web search tool: proceed with codebase findings and state the limitation.
- Target directory cannot be created: stop and report the filesystem error.

Adapted from compozy's `cy-create-prd` skill (MIT, © 2026 NauckGroup LTDA), simplified for double-o.
