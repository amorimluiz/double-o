---
name: 00-techspec
description: Translate an approved PRD into a technical specification with architecture, interfaces, data models, and test strategy, through codebase exploration and interactive technical grilling. Use when a PRD exists and needs a design, or when the 00-loop SDD pipeline reaches its TechSpec step. Do not use for PRD creation, task breakdown, or implementation.
---

# Create TechSpec

Produce `.sdd/<slug>/techspec.md`: the HOW for the approved PRD. Downstream skills (`00-tasks`, `00-loop`) consume it for component boundaries, interfaces, data models, integration points, and the test strategy.

<HARD-GATE>
- Explore before designing: every TechSpec must be informed by the existing architecture.
- Questions before writing: the user shapes the design by answering technical clarification questions, one at a time. Every TechSpec, however simple.
- Decide, then write: after the grilling converges, write the file directly. The user reviews the generated file and requests changes — no draft-approval loops.
</HARD-GATE>

## Workspace

- Directory: `.sdd/<slug>/` (same slug as the PRD).
- Read first: `prd.md` — the primary input, including its Decisions and Open Questions. When it is missing, stop and ask for the PRD or for a description to work from.
- Artifacts: `prd.md`, `techspec.md` (this skill), `design/`, `tasks.md`, `task-NN.md`, `qa/`, `review/`, `notes/`, `state.yml`. Every run artifact stays under `.sdd/<slug>/` and is never committed (full layout: `00-loop` → Bootstrap).
- Language: reuse the language recorded in `state.yml`; when absent, detect it per `00-prd`.

## State

Update `.sdd/<slug>/state.yml` at every completed step; full schema in the sibling skill `00-loop` (`references/state-schema.md`). This skill owns:

- On start: `step: techspec`, `updated_at`.
- On completion: `artifacts.techspec: done`, `step: design`, `updated_at`.
- On a real blocker: `step: blocked`, append the blocker.

## Workflow

1. Gather context: read `prd.md` and the current `techspec.md` when present (update mode: preserve approved sections). Explore the codebase through the runtime's subagent tool when available: architecture patterns, existing components, dependencies, and the technology stack.
2. Grill the design: walk the decision tree branch by branch — architecture and component boundaries, data models and storage, API design and integration points, testing strategy, performance. Use the same question mechanics as `00-prd/references/question-protocol.md`; never spend a question on what the codebase can answer.
3. Write `techspec.md` using `references/techspec-template.md`; fill every applicable section and note the reason for omissions. Map every PRD goal and user story to a technical component, referencing PRD sections by name instead of restating business context. When the architecture or a flow reads better as a diagram, use the `mermaid-diagrams` skill and inline the Mermaid block.
4. Hand off: report the file path and ask for approval. On approval, point to `00-design` — or continue the `00-loop` pipeline when it invoked this skill.

## Design Minimalism

Design for the complete PRD scope in this single TechSpec. Design size is never a reason to trim or stage scope. Minimalism applies to the design itself: include no component, interface, or abstraction the design does not strictly need, and prefer extending an existing module over proposing new packages or directories.

## Error Handling

- PRD missing: stop; ask for the PRD or a description to proceed.
- Product decisions contradicted by the codebase: document both, recommend one with rationale.
- Conflicting architectural patterns: document both, recommend one with the trade-offs.
- Target directory missing: create it.

Adapted from compozy's `cy-create-techspec` skill (MIT, © 2026 NauckGroup LTDA), simplified for double-o.
