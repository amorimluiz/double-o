# Documentation

Hard rules for documentation, in every language and stack. A project's own conventions override the specifics here. Documentation is written in English by default; use another language only when the repository or product requires it. Snippets are illustrative.

Apply every rule to every change. If a rule must be broken, name it and why in the change description.

## The update test

After changing code, update a document if and only if one of these holds:

- the change alters what an outside consumer can observe or rely on — API, behavior, config, error, operation, security, output; or
- the change makes an existing document or comment false.

Otherwise leave the docs alone. A no-signal change with a doc diff is churn; a signal change with no doc diff is a defect.

Code comments and docstrings follow `rules/code-standards.md`. This file governs prose documentation and when it must change.

## Must update — signal changes

Map each change to the smallest artifact that carries it:

- new or changed public API, parameters, return, or errors → the symbol's doc comment and the reference;
- new or changed configuration, CLI flag, env var, or default → the config/CLI reference;
- changed observable behavior, side effects, limits, ordering, or validation → the doc that states that behavior;
- new or changed error conditions or recovery steps → the reference and the relevant guide;
- new or changed setup, build, install, migration, or deploy step → the README or quickstart;
- security-relevant behavior (auth, permissions, secrets, a patched risk) → the security or usage docs;
- a user-visible feature, fix, or breaking change → the README or guide, and the changelog if the project keeps one;
- a non-obvious constraint, gotcha, or accepted limitation → a doc comment or a doc near the code;
- a decision that is expensive to reverse → its rationale in the commit body;
- a renamed or removed symbol, file, or config that another doc names → every reference to it.

## Must not update — no-signal changes

These are not documentation triggers. Touch a doc only to fix something the change made false:

- internal refactor with unchanged observable behavior;
- private helper change outside any contract;
- tests, fixtures, or CI config;
- formatting, whitespace, or local renames;
- a dependency bump with no behavior, config, or operational impact;
- a bug fix that restores already-documented behavior;
- performance work within the same contract;
- documentation-only or comment-only churn.

The shortcut: does the diff change a fact a consumer could rely on, or falsify an existing doc? If not, ship no doc change.

## README is mandatory

Every repository has a `README.md` at its root. It answers, briefly:

- what the project is and why it exists;
- how to install and run it;
- one usage example that works;
- where to get help and who maintains it.

Keep it short; push depth into dedicated docs and link to them. Do not duplicate the reference here.

## Doc types — one purpose each

Classify every document by the reader's need, and give it exactly one:

- **Tutorial** — teaches by doing, for a beginner.
- **How-to** — solves a specific goal, for a competent user.
- **Reference** — describes the machinery, dry and complete.
- **Explanation** — gives context and rationale, the why.

Do not mix types in one document; a hybrid serves no reader. Name a new document's type before writing it.

## Single source of truth

- Each fact has one canonical home; everywhere else links to it.
- Reference for an API is generated from the source (comments or spec), not hand-copied.
- The README owns entry-point facts; deeper docs expand, never restate.
- The why of a change lives in the commit body (see `rules/git-workflow.md`).
- Delete a document that contradicts reality rather than leaving it in place.

## Reference and public surface

- Document every publicly reachable name: functions, types, endpoints, flags, config keys, events, formats.
- State the contract, not the implementation: purpose, inputs, outputs, errors, side effects, and any safety or concurrency constraint.
- Never restate the signature or the types the type system already declares.
- Prefer a generated reference over a hand-maintained one; when hand-written, change it in the same change as the code.

## Style

- Second person ("you"), active voice, present tense.
- Plain, concise language; one term per concept, used consistently.
- Timeless: avoid "currently", "new", "soon", "in the future". Describe the current state.
- Never pre-announce unshipped features or leave placeholders ("coming soon", "TBD").
- Examples are runnable and shown with their expected output; keep them deterministic.

## Truthfulness

A document that lies is worse than no document.

- Examples that can execute must execute (doctests, tested snippets); a broken example is a defect.
- Every symbol, path, flag, or config key a doc names must resolve in the source.
- Mark obsolete docs obsolete with a pointer to the replacement, or delete them.
- Prefer deleting or superseding stale content over silently editing history.

## AI-generated documentation

Generated docs earn their place like any other, and the update test still governs.

- Ground every claim in the source. Never invent an API, flag, endpoint, or behavior.
- Resolve or abstain: if a referenced symbol or fact cannot be verified, do not publish the claim.
- Do not rewrite a document that is already correct, and do not mass-edit docs; change the smallest artifact the signal requires.
- Skip no-signal changes: no doc diff for internal refactors, formatting, or tests.
- A human verifies non-derivable prose — rationale, trade-offs, gotchas — before it ships.
- Disclose material AI assistance in the change description (see `rules/git-workflow.md`).

## Decision records

Decisions live in the change, not in a committed ADR:

- Put the rationale and the alternatives considered in the commit body.
- Do not add ADRs or decision logs to commits or pull requests.
- If you keep a personal decision log (for example, outside the repository in your own workflow), it stays out of the repository's change set.

## Keep diffs clean

- One concept per file, so changes stay small and conflict-free.
- Separate formatting-only edits from content edits.
- Never mass-rewrite documents; diff the minimum affected artifact.

## Before done

- State whether the change required a doc update, and which artifact changed.
- If no doc changed, state why the update test came out negative.
- Confirm examples still run and every named symbol still resolves.
