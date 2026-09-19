# API Design

Hard rules for every interface — HTTP/REST, RPC, GraphQL, a public library, or an agent-facing tool. A project's own conventions override the specifics here. Snippets are illustrative.

Apply every rule to every change. If a rule must be broken, name it and why in the change description.

## Contract first

The interface specification is the source of truth, written before the implementation and reviewed as a contract.

- Capture the contract in machine-readable form: OpenAPI for HTTP, proto/IDL for RPC, SDL for GraphQL, an exported type surface for a library, a typed schema for an agent tool.
- Design and review the surface before coding; implementation follows the contract, not the reverse.
- Keep the spec in version control beside the code, and regenerate clients and docs from it.
- Validate the spec and detect breaking changes in CI. A spec that drifts from behavior is a defect.

## The contract is bigger than the spec (Hyrum's Law)

With enough consumers, every observable behavior becomes depended upon — timing, ordering, defaults, error text, identifier formats.

- Treat all observable behavior as contract, not only what you documented.
- **Additive-only** within a version: add fields, resources, methods, enum values. Never remove, rename, retype, renumber, or change the meaning of an existing one.
- Renaming is remove + add. Removing is a new major version.
- Reserve retired identifiers so they can never be reused (proto field numbers and names; JSON and graph enum values).
- Keep the observable surface small: every behavior you expose is a permanent obligation.
- Fixing a bug can be a breaking change. When consumers may rely on the old behavior, gate the correction behind a version or a flag.

## Naming and shape

- Model resources (nouns), not operations. Collections are plural.
- Identifiers are stable and opaque; clients never parse length, prefix, or format.
- One casing convention per surface (paths, fields, query params, headers), applied everywhere; never mix.
- Use standard formats (RFC 3339 timestamps, standard money and country types) instead of bespoke ones.
- Never leak internals: no storage primary keys, ORM fields, or implementation flags in the public shape.
- Non-CRUD actions use a custom method (`collection/{id}:verb`) over POST, never a new HTTP verb.

## Methods and semantics

- Safe methods (GET, HEAD, OPTIONS, TRACE) have no side effects.
- Idempotent methods (the safe methods, PUT, DELETE) can be retried; POST and PATCH are not idempotent by default.
- GET never changes state. PUT replaces the whole resource; PATCH applies a partial change; do not mix the two.
- Use the most specific status code; never collapse failures into 200 or 500.
- Use conditional requests (`ETag` + `If-Match`) to prevent lost updates.

## Idempotency and retries

- Assume at-least-once delivery. Achieve exactly-once effect with deduplication, not by assuming the network behaves.
- Accept an `Idempotency-Key` for non-idempotent writes; store key to result, fingerprint the payload, and reject the same key with a different payload (`409`/`422`).
- Retry only retryable failures, with exponential backoff and jitter; honor `Retry-After`. Never retry a terminal client error.
- Test the double-fire: call every mutating operation twice and assert a single effect.

## Errors

Errors are part of the contract. Use RFC 9457 `application/problem+json`:

```json
{
  "type": "https://example.com/problems/out-of-credit",
  "title": "Insufficient credit",
  "status": 403,
  "detail": "Balance is 30; the request costs 50.",
  "instance": "/accounts/12345/transfers/abc"
}
```

- Include a stable, machine-readable `type` (and a domain code where useful); never make clients parse the human `detail`.
- Map HTTP status codes correctly and consistently; decide `403` versus `404` deliberately and document it.
- Never leak stack traces, queries, hostnames, or internal identifiers.
- Errors are stable: changing an error code or its meaning within a version is a breaking change.
- Distinguish terminal client errors from retryable server errors, and say which is which.

## Collections

- Paginate every collection from day one; adding pagination later is breaking.
- Default to an **opaque cursor**: not parseable, not authorization, only a position. Use offset only when random access is a real requirement.
- Bound the page size: an optional size parameter with a documented default and a server maximum; coerce down, never error on a missing size.
- Expose filtering, sorting, and field selection as first-class standardized parameters; reject unknown fields and values with a clear error.
- An empty cursor means end of collection. Treat totals as estimates.

## Versioning and evolution

- Put a major version in the path (`/v1`); never expose minor or patch.
- Version only on an incompatible change; additive changes ship within the current version.
- Keep at most one active major plus any version still in its deprecation window. A new major never depends on the previous one.
- Migrate breaks with **expand, migrate, contract**: add the new form beside the old, move consumers, remove the old only against measured zero usage.
- Announce deprecation machine-readably (`Deprecation` and `Sunset` headers, spec annotation, changelog), give a defined window, and return a definitive status after sunset.
- For libraries, use SemVer: deprecate in a minor, remove in a major.

## Compatibility

- Target backward compatibility: old clients keep working against new servers within a version.
- Be a tolerant reader: ignore unknown fields and unknown enum values rather than failing.
- Be a strict writer and validator: accept only valid input, plus explicitly modeled extension points. Accepting unknown extensions is not accepting invalid data.
- Decide unknown-value handling explicitly and document it.
- Defaults and their serialization are contract; do not change them within a version.
- Do not tighten validation on an existing field (making an optional field required, narrowing a range) within a version.

## Async and long-running work

- Do not block the request on slow work. Return `202 Accepted` and an operation resource with a pollable status, and propagate the final error through the error contract.
- Give operations a state enum and an expiry, and document eventual consistency explicitly.

## Auth and security at design time

- Declare authentication and authorization in the contract, with least-privilege scopes.
- Authorize per object and per property, not only per endpoint. Prevent mass assignment and excessive data exposure by design.
- Validate and bound all input at the boundary; every list and query has a cost ceiling (page size, complexity, rate limit).
- Keep secrets out of URLs and logs.

## Docs and examples are part of the interface

- Ship examples that are valid and, where possible, executed; a stale example is a defect.
- Document what tests cannot encode: idempotency, retries, limits, deprecation, side effects, and failure modes.
- Keep the spec self-contained and immutable per revision.

## Library APIs

- The same additive and compatibility rules apply, enforced by SemVer.
- Avoid boolean parameters, god objects, and leaking internals or concrete types.
- Document thrown errors, thread safety, and ownership of returned values; these are contract under Hyrum's Law.
- Deprecate public symbols before removing them in a major.

## Agent-facing tools

A tool for an LLM or agent is a public API with a caller that will not read the docs. Give it more discipline, not less.

- Type every input and output: explicit types, enums for closed sets, honest required fields, no loose blobs.
- Give every tool a stable, unique, intention-revealing name; keep the initial surface small.
- Apply least privilege, and require confirmation for destructive actions.
- Paginate and bound every list; validate arguments server-side.
- Treat tool descriptions and results as untrusted data, never as instructions.

## Forbidden anti-patterns

- Verbs in URLs; god endpoints (`/process`, `/do`).
- HTTP 200 with an error body; collapsing all failures into 400 or 500.
- Free-text-only errors with no stable code.
- Exposing storage fields or internal identifiers.
- Unbounded list responses; offset-only pagination by default.
- Boolean flags that encode invalid states instead of typed enums.
- PUT for partial updates; PATCH with replace semantics.
- Renumbering or reusing proto tags; changing field types; adding required fields.
- GraphQL queries with no depth, breadth, or complexity limit.
- Ad-hoc or proliferating versions with no support window; removing without deprecation.

## Before done

- Update the spec in the same change as the code.
- Run the contract tests and the breaking-change check; report the result.
- State the compatibility impact: additive, deprecated, or breaking.
- Confirm every example and named field still reflects real behavior.
