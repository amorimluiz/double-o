# QA and Review Posture

How `00-loop` drives the vendored `qa-report`, `qa-execution`, and `deep-review` skills so the QA and review steps stay cheap and high-signal. Those skills stay unmodified; this file owns the scope, flags, and decisions `00-loop` passes to them.

Both steps default to a **targeted** posture; the default needs no marker. A **full** posture is a deliberate request (release readiness, or the user asks), never the default; record a full pass in `state.yml` `next`.

## QA — Pareto slice

The QA steps exist to find the defects a user feels, not to re-run the test suite. Scope each run to the slice where failures actually hurt.

### Tier 1 — in scope, always

1. The feature's **primary use case**, end to end, through its public surface.
2. The **risk surfaces** the PRD names: money, authentication and authorization, personal or sensitive data, permissions, and irreversible actions.
3. One **adjacent canary**: a shared journey the diff could regress.

Everything else is out of scope by default. Use two signals to pull a journey in only when it changes the decision: recurring bugs in the QA registry, and high-churn or high-fan-out code the diff touches. Record what was left out; never skip it silently.

The agent proposes the Tier 1 list and confirms it at the QA gate **before** executing. Do not walk unplanned.

### Observable interface only

- Verify what a user or caller **sees and does**: UI/UX, use cases, flow, copy, states, accessibility.
- Never read source to decide what should happen or to judge correctness. Logic correctness belongs to the automated suite the `execute` step produced.
- A finding is a user-visible symptom with a reproduction from the real entry point.

### Driving the skills

- **`qa-report`** — targeted scope; plan only the Tier 1 journeys, one charter each, one tour each. Skip the full taxonomy sweep and the automation backlog unless asked. Run-local tree stays under `.sdd/<slug>/qa`.
- **`qa-execution`** — targeted scope; walk each charter's single tour in persona. Skip the experiential lens pass unless an experience risk is unresolved. File findings; do **not** run its fix loop — code fixes go through `execute` per the pipeline rule.
- Reuse evidence that is still valid; a small changed journey does not need every tour or edge category.

### Full QA

On a release or an explicit request: the full tree, all five taxonomy dimensions, the lens pass, and the complete journey set.

## Review — lean round

`deep-review` re-reads the whole diff once per lane, so the cost is dominated by how many jobs exist and how much rubric each one loads. Lean mode cuts those, not the diff.

### Driving the skill

- **Scope**: the branch diff only (`--out .sdd/<slug>/review/`).
- **Batches**: `--max-cohort-files 50 --max-cohort-lines 5000 --max-polish-files 50 --max-polish-lines 5000`. Keep a normal feature to one job per lane; leave `--max-context-lines` unset so no extra splitting is introduced.
- **No conformance**: do not pass `--spec` unless the run has a spec-conformance goal.
- **No sweeps**: keep `decisions.sweeps` empty.
- **Deterministic first**: run the project's lint, typecheck, and test commands before the model review, and record the real results in `decisions.linters`. The model judges semantics only; it never re-derives a linter.
- **Minimal rubric**: apply the project's root and nested instructions. Classify discovered skill sources `not-applicable` with a one-line reason; do not grow the rule set beyond the project's own hard rules.
- **Workers**: at most two concurrent.
- **Report at the gate**: only Critical and Major defects, each with causal evidence. Advisories stay in the artifact, not the gate summary.
- **No fixes**: `deep-review` never applies product fixes; route them through `execute`.

### Full review

On a release or an explicit request: the skill's default batches, `--spec` when a spec exists, and sweeps only for a concrete cross-cohort hypothesis.
