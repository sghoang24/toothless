---
name: test-strategy
description: "You MUST use this as part of the Plan phase, after the tasks are cut, to settle the feature's test strategy once. The goal is to decide, across the feature's tasks, what to test, at which layer (unit / integration), the mocking boundary, determinism, coverage that matters, and the project's declared test command. E2E belongs to the milestone (`e2e-testing`), not the feature. You are not allowed to treat a flaky test as noise (it's a defect) or to assume the command. Trigger phrases: test strategy, what should I test, unit vs integration, should I mock this, flaky test, test plan."
argument-hint: "Point at the feature, e.g. specs/001-auth/ (its tasks.md), or a behavior you're unsure how to test."
---

# Test Strategy

`planning` slices a feature into tasks; the loop's `implement` writes the test **first** for one behavior and watches it fail; `verify` proves the whole suite is **green** with fresh evidence. None of them owns the question above the feature's tasks: **what deserves a test, at which layer, and against what boundary — for this whole feature.** That is this skill.

It is the strategy layer for the tests of **one feature** — mostly **unit** and **integration** — settled **once, inside the Plan phase, after the tasks are cut and before the `development-loop` begins**. A strategy re-decided on every task drifts and overlaps; settled per feature, the tests stay coherent as a suite. **End-to-end is not a feature concern**; assembled-product journeys are owned by `e2e-testing` and run at a milestone/release, not on every feature. This skill tells you which unit/integration tests each of the feature's tasks must write, so `implement` and `verify` have the right ones; it doesn't gate releases (that's `e2e-testing`).

**Self-contained.** Use it to set a feature's test strategy, whether or not `fight` is driving; `fight`/`planning` simply call it when they're active.

## The Model

| Thing        | What it is                                                          | Where it lives            |
| ------------ | ------------------------------------------------------------------- | ------------------------- |
| **Strategy** | The feature's whole test plan: layers, seams, which task pins what. | `specs/NNN-slug/tasks.md` |
| **Layer**    | Unit, integration, or end-to-end — how much real system runs.       | the project's test tree   |
| **Behavior** | The observable outcome a test asserts, not the internals.           | the test's name           |
| **Boundary** | The seam you mock — or deliberately don't.                          | the test's setup          |
| **Test cmd** | The command that runs the suite. Declared, never assumed.           | `AGENTS.md` / `Makefile`  |
| **Coverage** | Which behaviors are exercised — not a single percentage.            | the suite, at review time |

The layer is chosen by **what the test must prove**, not by habit. The more of the real system a test runs, the more it proves and the slower and more brittle it gets. Pick the cheapest layer that still proves the behavior.

## When to Use

- When **planning finishes cutting tasks** for a feature or for a whole phase — to settle the strategy before the loop starts (this skill is part of the Plan phase). Settle it **once per phase** when the phase's features are planned together: one command, one set of shared seams, so the phase does not end up with three different fakes for one clock.
- When a new task or a spec revision adds behavior the strategy doesn't yet cover.
- When a behavior is hard to test and you're tempted to mock everything.
- When a test is flaky, slow, or tests implementation detail instead of behavior.
- When judging whether a feature is covered enough — at review, or before a release.
- When the project has no declared test command and you need to find it.

**Do NOT use** per task — a strategy re-decided on every task drifts and overlaps; settle it once for the feature. Not as the test-first ritual itself (that's `implement`), as the green-evidence gate (that's `verify`), or for the assembled-product release check (that's `e2e-testing`, run at a milestone). This skill decides the feature's unit/integration strategy; those steps _do_, _prove_, and _release-gate_ it.

## When it runs vs. the loop

| Scope               | What runs                                       | Owned by      | When                                       |
| ------------------- | ----------------------------------------------- | ------------- | ------------------------------------------ |
| **Plan phase**      | The phase's (or feature's) unit + integration strategy | this skill    | once, after tasks are cut, before the loop |
| **Loop (per unit)** | `implement` writes tests; `verify` proves green | the loop      | every unit, in `development-loop`          |
| **Milestone**       | End-to-end journeys on the assembled product    | `e2e-testing` | before a release/deploy                    |

This skill is **set up once, in Plan** — it is not a loop step. The `development-loop` only runs `implement -> verify -> review`; the strategy they follow was settled here. Do not write e2e tests per feature — they're slow, brittle, and belong to the milestone gate. A feature proves its behaviors at the unit/integration layer; the assembled system is proven once, at the milestone.

## The Layers

Choose the lowest layer that proves the behavior. Push a test up a layer only when the thing you're proving only exists up there.

| Layer           | Runs                                           | Prove                                     | Keep it                               |
| --------------- | ---------------------------------------------- | ----------------------------------------- | ------------------------------------- |
| **Unit**        | One function/module, dependencies faked        | Pure logic, branches, edge cases          | Fast, most of the suite               |
| **Integration** | A few real components together (+ real store)  | The seams: DB, queue, HTTP client, wiring | Where units meet                      |
| **End-to-end**  | The whole assembled system (see `e2e-testing`) | Critical user journeys hold together      | The critical few — **milestone only** |

The shape is a pyramid, not a diamond: most tests are unit, some integration, few e2e. An inverted suite (mostly slow e2e) gets skipped and proves little. Per-feature work lives in the top two rows; the bottom row is the milestone gate.

## Core Rules

- **Test behavior, not implementation.** Assert the observable outcome ("rejects an expired token"), not the private calls it made. A test that breaks when you rename a helper was testing the wrong thing.
- **One behavior per test.** A test with "and" in its name is usually two tests, or one that will fail ambiguously. Name it for the outcome it pins.
- **Mock at the boundary you own, not the logic under test.** Fake the network, the clock, the filesystem, the third party — not the function you're testing. Over-mocking proves your mocks agree with you, not that the code works.
- **Prefer the real thing when it's cheap.** An in-memory store or a real (test) DB often beats a mock: it catches the seam you'd otherwise fake away.
- **Deterministic or it doesn't count.** No wall-clock dependence (inject the clock), no unseeded randomness, no fixed ports, no dependence on test order or leftover state. Each test sets up and tears down its own world.
- **Treat flaky as broken.** An intermittently failing test is a defect — a real race, a shared-state leak, or a brittle wait. Fix the cause; never `sleep`, never re-run until green. Quarantine only with a tracked issue.
- **Cover behaviors, not lines.** Cover the branches that matter (boundaries, error paths, the new behavior) — a 100% number over trivial getters proves nothing. Coverage is a question you ask ("is this behavior pinned?"), not a trophy.
- **The command is declared, not assumed — and this is the one place it's settled.** Per `AGENTS.md`, the project names its test/check command. Discover it once here; `implement` and `verify` use the command this step nails down rather than re-deriving it. Never assume `make test` or `npm test`.
- **Each task's test comes from the feature's strategy.** Per `AGENTS.md`, tests come with the behavior that needs them. The strategy names which task writes which test; don't defer tests to a "write the tests" task — that layer-task is an anti-pattern — and don't add a test the strategy doesn't call for, or change a layer, without updating it.

## Procedure

### 1. Find the declared test command

Read `AGENTS.md` for the project's test/check command. If it isn't there, discover it — in order: a `Makefile` target (`check`, `test`), `package.json` scripts, `pyproject.toml`/`tox.ini`, `Cargo.toml`, `go test`, or the project's CI workflow file. If none exists, say so and propose the command to the user before using it. Never invent one silently.

This is the **single source of truth** for how the command is found: `implement` runs it during GREEN, and `verify` runs it for fresh evidence — neither re-derives it. If you find it already recorded (in `AGENTS.md`, a prior status note, or a CI file), reuse that instead of searching again.

### 2. Enumerate the feature's behaviors and map them to tasks

List the behaviors the feature's acceptance criteria (`AC-n`) demand — one line each ("an expired token is rejected"). Map each to the task in `specs/NNN-slug/tasks.md` that must pin it. A behavior with no task is a planning gap — say so.

### 3. Pick the layer for each behavior

For each behavior ask: **what is the cheapest layer that still proves it?** Pure logic → unit. A real seam (stores, transports, wiring) → integration. A user-visible path across components is **e2e and belongs to the milestone** (`e2e-testing`), not the feature. Don't push a unit-shaped behavior up to e2e for comfort. Read the layers over the feature's whole suite, so one seam isn't tested three times.

### 4. Decide the boundaries

List what the tests need that they don't control — network, clock, randomness, filesystem, services. For each: **fake it (deterministically) or run it real (if cheap and hermetic).** Keep the thing under test real. Write the seams down once, so every task reuses them.

### 5. Record the strategy

Write it into each planned feature's `specs/NNN-slug/tasks.md`: a short `## Test strategy` section (the command, the layers in play, the shared seams) and each task's `Tests:` line carrying the behavior, layer, and boundary. One place the whole feature reads from.

When a whole phase was planned in one run, the **command and shared seams are decided once for the phase** and repeated verbatim into every feature's `tasks.md`; only the per-feature layers and task mapping differ. Never let two features in one phase disagree about what is faked.

### 6. Hand the strategy to the loop

The strategy is now written down; the Plan phase is done. In the `development-loop`, `implement` writes each task's test first (RED-GREEN-REFACTOR), `verify` proves the suite green, and `review` judges the diff — all following the strategy recorded here. This skill sets the strategy; it does not write the tests.

### 7. Keep it honest and live

When a test lands, check it is honest: does it fail if the behavior breaks (invert the condition and watch)? Does anything depend on order, clock, or shared state? Would it catch a regression, or just re-state the implementation? When a task or a spec revision adds behavior, update the strategy first, then the test. `review` judges the tests (including the mutation check); `verify` runs the full suite for fresh evidence.

## Red Flags

| Thought                                     | Reality                                                                       |
| ------------------------------------------- | ----------------------------------------------------------------------------- |
| "Mock everything so the test is fast"       | You're testing your mocks. Keep the unit real; fake only the boundary.        |
| "Push it to e2e, it's more realistic"       | Slower, flakier, and proves less about the unit. Cheapest proving layer wins. |
| "I'll add an e2e test for this feature"     | e2e is the milestone gate, not per feature. Prove it at unit/integration.     |
| "I'll assert the private helper was called" | That's implementation detail. Assert the observable outcome.                  |
| "Just add a sleep, it's flaky"              | A sleep hides a race. Inject the clock / wait for the real condition.         |
| "Re-run it, it passes sometimes"            | Flaky is a defect. Re-running until green is how bugs ship.                   |
| "Coverage is 90%, we're fine"               | Coverage counts lines touched, not behaviors pinned. Ask what's untested.     |
| "I'll write the tests in a later task"      | Tests come with the behavior. A "write tests" task is a layer-task.           |
| "I'll settle the strategy task by task"     | Per-task strategies drift and overlap. Settle it once for the feature.        |
| "The command is obviously `make test`"      | It's declared in `AGENTS.md`. Find it; don't assume.                          |
| "Too simple to test"                        | Simple code breaks. The test takes a minute.                                  |

## Completion Check

The feature's strategy is sound when ALL are true (this is the Plan-phase exit, before the loop):

- [ ] The project's test command was found from `AGENTS.md` (or discovered and confirmed), never assumed.
- [ ] Every acceptance criterion's behavior is named and traced to the task that pins it; each layer is the cheapest that proves it (unit/integration; e2e deferred to the milestone).
- [ ] The boundaries are drawn: what's real (the unit) vs. faked (network/clock/randomness), and why — written once for the feature.
- [ ] The strategy is recorded in `specs/NNN-slug/tasks.md` (`## Test strategy` + each task's `Tests:` line).
- [ ] Branches that matter (boundaries, error paths, the new behavior) are covered; coverage is judged by behavior, not a number.

Then the loop takes over, and these hold there:

- [ ] Tests assert observable behavior, not implementation detail; one behavior each.
- [ ] Tests are deterministic and independent — no clock, order, port, or shared-state dependence; no bare sleeps.
- [ ] Flaky tests were treated as defects, not re-run to green.
- [ ] The tests were written test-first by `implement`; `review` and `verify` still gate the feature.
