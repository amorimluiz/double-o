# OpenDesign Integration

How this skill turns approved interface decisions into prototypes on disk. The MCP server is `open-design`; its tools default to the active project, so always pass the project explicitly.

## Preconditions

- The `open-design` MCP server is connected and the OpenDesign app is running. When it is missing, record the blocker and report the fix in `SETUP.md` (OpenDesign section).
- `design/reference/fidelity.md` exists — the capture from `references/fidelity-capture.md` is complete, or explicitly marked degraded.
- `DESIGN.md` exists at the repo root. When absent, author it first (below).
- The interface grilling has converged; the brief is composed from confirmed answers only.

## DESIGN.md when absent

Author `DESIGN.md` at the repo root in the [Google Labs format](https://github.com/google-labs-code/design.md): YAML front matter with the machine-readable tokens (`colors`, `typography`, `rounded`, `spacing`, `components`), then the markdown body in canonical order — Overview, Colors, Typography, Layout, Elevation & Depth, Shapes, Components, Do's and Don'ts. Seed the tokens from the captured reference (`design/reference/tokens.md`) first, then the existing codebase (CSS variables, Tailwind theme, theme files); when the project has an OpenDesign design system, read `od://design-systems/<id>/DESIGN.md` and adapt it. This file is a project artifact and outlives the feature.

## Compose the brief

One structured prompt carrying, in order:

1. Goal and slug; the screens to produce.
2. The approved interface decisions from grilling: per screen, its information hierarchy, states, layout and responsive behavior, component reuse, interactions, and real copy.
3. PRD business context by section reference — do not restate it.
4. TechSpec interface and component references the prototypes must honor.
5. The captured reference (`design/reference/`): the screenshot paths, `tokens.md`, and `components.md` — the real interface the prototypes must match.
6. The fidelity contract (`design/reference/fidelity.md`): the allowed tokens and components, what is forbidden, the extend-only rule, and the approved new components.
7. The `DESIGN.md` tokens and rules: colors, type, spacing, and components the prototypes must use.
8. The Prototype Standard from `SKILL.md`: self-contained HTML per screen, all states, responsive, accessible, real content.
9. The requested output filenames: `NN-<screen>.html` plus `index.html`.

## Commission the run

1. `create_project` with the feature slug (create it once; reuse it across adjustment rounds).
2. Generate a canonical UUID/ULID `requestId` once for the confirmed action and reuse it verbatim on any retry.
3. `start_run` with the project, the composed prompt, and the `requestId`. Pick a skill or plugin from `list_skills` / `list_plugins` when one fits.
4. Poll `get_run` every 30-60 seconds. Runs take 5-30 minutes; a `running` status with unchanged file timestamps is the inner agent working, not a hang. Report progress to the user between polls and let it finish — never cancel a run to hand-write files as a shortcut.
5. On `succeeded`, use the returned `previewUrl` and output the notice before persisting. On `failed` with `failureAction: recharge`, show the recharge URL and, after the user confirms the top-up, resume the same run with the original `requestId` and `resume: true`.

## Persist into `.sdd/<slug>/design/`

The repo copy is the approved snapshot; the OpenDesign project stays the live source for iteration.

1. `get_artifact` (or `list_files` + `get_file`) to fetch the generated entry file and its siblings.
2. Write each prototype into `.sdd/<slug>/design/` with the local file tools — `NN-<screen>.html` per screen and `index.html` linking them. `open-design_write_file` writes into the OpenDesign store, not the repo, so it is for iterating in OpenDesign, not for this step.
3. `.sdd/` is gitignored by `00-loop`; never move prototypes into source, and never treat them as production code.

## Adjustment rounds

On requested changes, refine in the same OpenDesign project — a new `start_run` with a fresh `requestId` and a prompt naming the adjustments — then re-persist, re-run the fidelity check, and re-gate. Keep the previous HTML until the replacement is approved.

## Degraded mode

When OpenDesign is unavailable, write the prototypes as self-contained HTML with the local file tools, following the Prototype Standard and the fidelity contract in `design/reference/fidelity.md`, only on the user's explicit instruction. State clearly that these bypass OpenDesign, and record in `state.yml` that the prototypes were authored locally.
