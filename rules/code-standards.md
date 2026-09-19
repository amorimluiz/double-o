# Code Standards

Hard rules for all code, in every language and stack. A project's own standards override these; when they conflict, the project wins. Snippets are illustrative, not literal — apply the rule in your language's idioms.

Apply every applicable rule to every change. If a rule must be broken, name it and why in the change description.

## Simplicity — YAGNI

Build only what the task needs now.

- No interface, adapter, factory, layer, config flag, or generic parameter without two real implementations or an explicit current requirement.
- No extension points for an unspecified future. "Easy to extend" means clean seams, not pre-built hooks.
- Prefer direct code over metaprogramming and indirection.
- Delete speculative parameters, unused options, and dead branches instead of keeping them "for later".
- Over-engineering is a defect, not a style preference.

## Abstraction and duplication — AHA, Rule of Three

Duplicate knowledge is a defect; duplicate shape is often fine.

- First occurrence: inline. Second: tolerate. Third: extract.
- Extract for real repetition, never for resemblance. Two similar blocks with different reasons to change stay separate.
- Reuse before create: search the project for an existing module or helper before adding one.
- A **hasty abstraction** costs more than the duplication it removes. If an abstraction grows flags or conditionals to serve one caller, inline it back.

## Interfaces and boundaries — SOLID, applied

- Introduce an interface only with two real implementations or at a true boundary: I/O, network, time, OS, external service.
- Inside the domain, depend on concrete types. Apply dependency inversion at boundaries, not on every class.
- Single responsibility by **reason to change** (who asks for the change), not by method count. Do not split a cohesive type into fragments.
- Prefer composition over inheritance; keep hierarchies shallow and substitutable.
- Dependencies flow one way. Lower layers never know higher layers.

## Functions, modules, files

- One responsibility and one level of abstraction per function.
- **Depth over length**: hide more behavior behind a smaller interface. Do not fragment a function into steps that only make sense in sequence just to shorten it.
- No line-count limit. A long function is a signal to inspect, not an automatic defect.
- Files hold one primary concept. When a file grows by accretion, split along responsibility or branch; never let one reach thousands of lines.
- Group by domain or feature, not by technical type.
- Keep the public surface small; hide what callers do not need.

## Naming

- Names reveal intent: what a thing is or does, not how.
- Use the domain's vocabulary. One concept, one name, across the codebase.
- Functions are verbs, types are nouns, booleans are predicates.
- No abbreviations except universally understood ones (`id`, `url`, `http`).
- Generic names are a hard guardrail: `data`, `info`, `manager`, `handler`, `helper`, `util`, `base`, `common`, `shared` — only with a qualifier that names the domain meaning.

## Comments and documentation

- Prefer self-documenting code: rename and restructure before commenting.
- Comment only what code cannot say: intent, a non-obvious constraint, the reason for a choice, an external reference.
- Never narrate what the code plainly does. No commented-out code. No `TODO` without a linked issue.
- Document a public API only when its signature does not already show inputs, outputs, errors, and side effects — then one line.
- A stale comment is a defect. Update or delete it with the code.

## Errors

Classify every failure before handling it.

- **Expected** (part of the contract: not found, invalid input, conflict, rate limit): return it in the type — `Result`, `Either`, typed error, or optional. Exceptions are not for expected flow.
- **Unexpected** (bug, invariant violation, infrastructure): let it propagate to a single top-level handler. Do not catch it locally.
- Never swallow an error. An empty catch or an ignored result is a defect. If you catch, handle it, log it with context, or rethrow.
- Catch the most specific type. A broad catch belongs only in the top-level handler.
- When wrapping, preserve the cause and the original stack.
- Log once, at the boundary that handles the error, with enough context to act: the operation, the constraint, the relevant input.
- Users get a generic message; details go to the log. Never leak paths, stack traces, versions, or secrets.
- Validate preconditions at the function entry (fail fast), before any side effect.

## Validation and trust boundaries

- Classify every input as trusted or untrusted. Validate all untrusted input at the boundary, server-side, before use.
- Allowlist what is permitted; do not enumerate what is forbidden. Validate syntax before semantics.
- Validation is not sanitization: pair it with parameterized queries and context-correct output encoding.
- **Fail closed**: when in doubt, deny.
- Client-side validation is UX only, never a security control.
- Treat data from the network, files, environment, IPC, and the model itself as untrusted.

## AI-specific anti-patterns

Hard guardrails. Override only on explicit instruction.

- **Dependencies:** never add a production dependency without approval. Verify provenance — exact name, real source, maintenance — before using it. Never trust a package name the model produced.
- **Scope:** change only what the task requires. No unrequested refactors, renames, or "while I was here" edits.
- **Invented APIs:** confirm every function, method, flag, and config key against real docs or source before calling it.
- **Placeholders:** never emit `... existing code ...`, stubs, or truncated output. Produce the complete change.
- **Secrets:** never hardcode or log credentials, tokens, or keys.
- **Weakened verification:** never delete, skip, or loosen a test to make a change pass.

## Verification before done

- Run the project's tests, linter, formatter, and build; report the actual output as evidence.
- A change is done when its behavior is proven, not when it looks right.
