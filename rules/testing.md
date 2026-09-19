# Testing Standards

Hard rules for tests, in every language and stack. A project's tools and conventions shape how tests are written; the obligations below — test-first, the coverage floor, and the ban on weakened tests — hold everywhere. Snippets are illustrative pseudocode: apply the rule in your language's idioms.

Apply every rule to every change. If a rule must be broken, name it and why in the change description.

## Test-first is mandatory for new behavior

Write the test before the production code, and watch it fail before making it pass: **red → green → refactor**.

- Red must be real: the test fails for the missing behavior, not for a typo you did not intend. Confirm the failure matches the behavior.
- Write only enough test to fail, then only enough code to pass, then refactor.
- **Double-entry:** state the expected value independently. Never derive it by calling the code under test — that makes the test agree with the bug.
- Every new feature ships test-first. Every bug fix starts with a failing regression test.
- If a case is genuinely impractical to test first (an exploratory spike, a visual or latency property), it does not ship until tests exist; state the deviation and why.

```
red      -> run the new test; it fails for the right reason
green    -> minimal code; the new test and all prior tests pass
refactor -> improve structure; tests stay green
```

## Coverage floor

- New and changed code carries **at least 80% unit-test coverage**. This is a floor, not a target: below it, work is not done; above it, do not chase the number.
- Measure it from unit tests. Do not count incidental coverage from integration or E2E tests.
- Prefer **branch/decision** coverage over line coverage when both are available.
- Overall coverage must never decrease. A drop is a defect unless the change removes code.
- Test the risk, not the number: a covered line with no meaningful assertion is not covered. Where the project has mutation testing, run it on changed code — a surviving mutant is a test defect.

## Choose the smallest test level that can verify the behavior

- Test each behavior at the lowest level that can meaningfully verify it; do not repeat the same assertion at several levels.
- Prefer unit → integration/component → end-to-end. Avoid the inverted **ice-cream cone** (mostly E2E) and the **hourglass** (unit and E2E, little in between).
- If the system's real complexity lives at service boundaries, integration tests are the high-value layer; that is a valid emphasis, not an excuse to skip unit tests of isolated logic.
- End-to-end tests cover critical journeys and wiring. Keep them few, deterministic, and fast to localize.

## A unit test is defined by isolation, not by file layout

A test is a unit test only if it:

- runs in-process and fast (milliseconds, not seconds);
- needs no network, disk, database, or external service;
- never sleeps or blocks on real time;
- passes in any order and in parallel with any other test;
- imposes its own state and cleans up, sharing no mutable global state.

If it touches any of these, it is an integration test — even inside the unit-test suite. Keep the suites separate so a unit run cannot be slowed or broken by infrastructure.

FIRST: **F**ast, **I**solated, **R**epeatable, **S**elf-validating, **T**imely.

## Test doubles

Prefer real objects. Reach for a double only when the real collaborator is slow, nondeterministic, or outside your control.

- Order of preference: **real implementation → fake → stub → mock**. A fake keeps fidelity to the real contract and has its own tests; verify it with a **contract test** against the real implementation where feasible.
- Mock only at boundaries you do not own: network, third-party APIs, clock, randomness, filesystem/OS, queues.
- Never mock the subject under test. Replacing the code under test proves nothing about it.
- Prefer **state** assertions over **interaction** assertions: verify what the system produced, not which methods it called in which order.
- Do not overspecify incidental arguments or call order; that couples the test to the implementation and makes it a **change detector**.

## Test design

- Arrange–Act–Assert (Given–When–Then). One action under test, one behavior per test.
- Assert on observable behavior, not implementation detail. A refactor that preserves behavior must not break the test; if it does, rewrite the test.
- A failing test must identify exactly what broke. Split tests that verify unrelated behaviors.
- No logic in tests: no `if`, `switch`, or loop that decides whether to assert. Use parameterized tests instead of loops.
- Self-validating: pass/fail needs no human reading output. Name tests by scenario and expected behavior, not by function name.
- Use builders, fixtures, and object mothers to make setup readable. Test code is production code: review it, name it, maintain it.
- Assert the value or shape, not mere existence. `not null`, `not to throw`, `is defined`, `is true` prove almost nothing.

## Determinism

A test that sometimes fails without a code change is a defect — it destroys the signal for every other test.

- **Time:** never read the system clock directly in logic. Inject a clock and freeze it. Cover boundaries: midnight, month end, DST, leap year.
- **Randomness:** seed every generator; log the seed on failure so it reproduces.
- **Async and concurrency:** never sleep to wait. Poll with a bounded timeout, or react to a signal. Control threads and barriers; a flaky concurrency test is often a real bug.
- **State:** start each test from a known state; prefer rebuild over cleanup; reset globals and singletons.
- **Order:** tests pass in any order and in parallel. Randomize order in CI to expose hidden coupling.
- **I/O:** unique temp paths per test; roll back or rebuild databases; clean up.
- Never fix a flake by retrying, widening a timeout, or lowering an assertion. Fix the cause or delete the test.

## Bug fixes

- Begin with a test that reproduces the bug and **fails on the current code**; fix until it passes; keep the test as a permanent regression guard.
- A regression test that passes before the fix proves nothing and must not be merged.
- When a bug escapes, add a test for the class of bug, not only the single report.

## Forbidden test anti-patterns

- **Assertion-free tests:** executing code is not testing it.
- **Tautologies:** asserting a value equals itself, or computing the expected value with the code under test.
- **Mocked subject**, or asserting only on mocks.
- **Weakened assertions:** a broad catch or `raises(Exception)` that any failure satisfies; a softened equality that most wrong values pass.
- **Commented-out, skipped, or deleted tests** to reach green. Every skipped test needs a linked issue and an expiry.
- **Coverage gaming:** trivial tests for getters or framework code, quiet additions to the exclusion list, testing code you did not write, snapshot dumps.
- **Snapshot abuse:** a snapshot never substitutes for an assertion; keep snapshots small, deterministic, and reviewed, and update them individually — never regenerate wholesale to pass.
- **Test logic in production:** no test-only flags, branches, or methods in shipped code.
- **Mystery data:** fixtures or external files the test does not make visible.
- **Slow suite:** tests too slow to run on every change will not be run.

## AI-generated tests

Generated tests earn their place the same way humans' do. The coverage floor and every rule above apply without exception.

Accept a generated test only if all four hold:

1. It compiles or builds.
2. It passes reliably (run repeatedly; no flakes).
3. It increases coverage of the changed code.
4. It contains a real, value-specific assertion.

Then prove it can fail: revert or remove the production change and confirm the test goes **red**. A test that passes on the pre-change code is worthless.

- Never accept a test because it passes on the current implementation — if the implementation is buggy, that freezes the bug.
- Verify every generated function, method, flag, and dependency against real docs or source before use.
- Disclose material AI assistance in the change description.

## Proof before done

- Report the actual command and its output: tests, coverage, linter, build.
- State the unit coverage of the changed code against the 80% floor.
- A change is done when its behavior is proven by a test that can fail, not when the suite is green.
