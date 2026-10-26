---
name: 00-design
description: Generate the user-interface prototypes for a feature — capture the app's real screens, tokens, and components as a fidelity reference, grill the interface, commission OpenDesign with the PRD, TechSpec, DESIGN.md, and the capture injected as context, then write self-contained interactive HTML prototypes to .sdd/<slug>/design/ behind a fidelity check and an approval gate. Use when a TechSpec exists and the feature has a user-facing interface, or when the 00-loop SDD pipeline reaches its Design step. Do not use for PRD, technical design, task breakdown, or implementation.
---

# Create Interface Prototypes

Produce `.sdd/<slug>/design/`: self-contained interactive HTML prototypes, one per screen, that pin the interface before any production code exists. This is the interface contract `00-tasks` decomposes and the execution loop implements against.

<HARD-GATE>
- Applicability first: a feature with no user-facing surface records a skip and returns to the pipeline. Never fabricate prototypes for internal-only work.
- Ground before generating: capture the app's real screens, computed styles, and components into `design/reference/` before composing the brief (`references/fidelity-capture.md`). Extend the existing visual language; invent nothing.
- Explore before asking: read the PRD, TechSpec, existing screens, the captured reference, `DESIGN.md`, and the `ui-ux-pro-max` design system before spending a question.
- Questions before generating: the user shapes the interface by answering interface questions, one at a time. Use the mechanics in `00-prd/references/question-protocol.md`.
- Fidelity before approval: every prototype conforms to the capture and the token/component allowlist; a genuinely new component needs explicit user approval.
- Approval before advancing: prototypes are a gate. Apply requested adjustments and re-gate; advance only on explicit approval.
</HARD-GATE>

## Workspace

- Directory: `.sdd/<slug>/design/` (same slug as the PRD).
- Read first: `techspec.md` (primary), `prd.md`, `DESIGN.md` at the repo root, and the captured reference in `design/reference/` when it exists. When the TechSpec is missing, stop and point to `00-techspec`.
- `DESIGN.md` absent: author it at the repo root in the Google Labs format before generating; seed its tokens from the captured reference first, then the `ui-ux-pro-max` design-system output, and follow `references/opendesign-integration.md`.
- Artifacts: `design/NN-<screen>.html` plus `design/index.html` (links the screens); `design/reference/` (the fidelity capture, per `references/fidelity-capture.md`); the OpenDesign source files alongside them.
- Language: reuse the language recorded in `state.yml`.

## State

Update `.sdd/<slug>/state.yml`; full schema in the sibling skill `00-loop` (`references/state-schema.md`). This skill owns:

- On start: `step: design`, `updated_at`.
- On completion: `artifacts.design: done`, `step: tasks`, `updated_at`.
- On non-UI skip: `artifacts.design: skipped`, `step: tasks`, `updated_at`, with the reason in `next`.
- On a real blocker: `step: blocked`, append the blocker.

## Workflow

1. Applicability: decide from the PRD and TechSpec whether the feature exposes a surface a user sees — a screen, view, dialog, or TUI/CLI affordance. No such surface: record the skip and hand off to `00-tasks`. Otherwise continue.
2. Capture the existing interface: read `references/fidelity-capture.md` and reach the running app. Capture the in-scope screens, their computed styles, and the component inventory into `design/reference/`. When the app is unreachable, capture from static sources, label it degraded, and say so at the gate.
3. Explore before asking: the captured reference, the existing screens, components, and tokens in the codebase, the design system in `DESIGN.md` plus any OpenDesign design system (`od://design-systems/<id>/DESIGN.md`), and one `ui-ux-pro-max` design-system query for the product type (`references/design-intelligence.md`). Present the merged findings, 3-5 bullets.
4. Grill the interface: read `references/interface-grilling.md` and walk its decision tree branch by branch, one question at a time, until every load-bearing interface decision is confirmed or parked with the user's consent.
5. Write the fidelity contract: `design/reference/fidelity.md` — the allowed tokens and components, what is forbidden, the extend-only rule, and the components that need explicit approval. Per `references/fidelity-capture.md`.
6. Generate: commission OpenDesign per `references/opendesign-integration.md` — compose the brief from the approved answers plus the captured reference, the fidelity contract, PRD, TechSpec, and `DESIGN.md`; start the run, poll it, and persist the returned prototypes into `design/`. Let the run finish; never hand-write files in place of a run that is still in flight.
7. Fidelity gate: compare each prototype against the capture and the contract, and iterate until it conforms — no invented tokens, components reused, layout and hierarchy matching the reference. Report the comparison.
8. Gate: report the prototype paths, the screen inventory, and the fidelity result, then ask for approval through the interactive question tool. On requested adjustments, refine through OpenDesign and re-gate. On approval, set state and hand off to `00-tasks` — or continue the `00-loop` pipeline when it invoked this skill.

## Prototype Standard

The definition of done for `design/`. Every prototype:

- One self-contained HTML file per screen — inline CSS/JS, no network, no build step — that opens directly in a browser.
- Covers its real states per screen: loading, empty, error, and success, plus narrow and wide viewports.
- Reads its colors, spacing, type, and components from the captured reference (`design/reference/`) and `DESIGN.md`; invents none, and extends the existing visual language rather than replacing it.
- Meets the accessibility bar in `rules/frontend.md` at prototype fidelity: semantic structure, keyboard order, visible focus, contrast, and accessible names.
- Uses real copy and realistic data; no placeholder text, dead links, or TODO markup.

## Error Handling

- TechSpec missing: stop; point to `00-techspec`.
- No user-facing surface: record the skip and advance to `tasks`; do not spend interface questions.
- Existing UI unreachable: capture from static sources per `references/fidelity-capture.md`, label the capture degraded, and say so at the gate; never present it as the live interface.
- `DESIGN.md` absent: create it at the repo root, seeded from the capture, then generate against it.
- OpenDesign unavailable (MCP disconnected or app not running): record the blocker and report the fix from `SETUP.md`; write prototypes as local HTML only on explicit instruction.
- Run fails with `failureAction: recharge`: show the recharge URL, and after the user confirms the top-up resume the same run with its original request id.
