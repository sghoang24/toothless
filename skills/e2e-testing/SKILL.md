---
name: e2e-testing
description: "Use ONLY when the user explicitly asks to test the whole project end to end (e2e tests, end-to-end run, smoke test of the full product). Runs the critical user journeys against the real assembled system and triages the failures. It is an on-demand skill: not part of the implement/verify/review loop, not a per-task or per-feature step, never started automatically by another skill, and not a gate for any other step. It does not replace unit or integration tests, and it never re-runs a flaky journey until it turns green."
argument-hint: "What to test, e.g. 'the whole project' or 'signup and checkout journeys'."
disable-model-invocation: true
---

# E2E Testing

The loop's `verify` step proves each task works in isolation. It does not prove the pieces work **together**: that the click reaches the API, the API writes the row, the row shows up on the next screen. This skill is the run that proves the assembled product works as a whole.

It is **on demand**. Use it when the user asks for it, whenever they ask. It is independent of the workflow: no other skill calls it, it updates no workflow state (`_state.md`, `_roadmap.md`, `status.md`), and it does not decide whether anything ships.

## The model

| Thing       | What it is                                                | Where it lives          |
| ----------- | --------------------------------------------------------- | ----------------------- |
| **Journey** | One user-visible path through the assembled system.       | named in the e2e suite  |
| **Suite**   | The set of journeys for this run.                         | `e2e/` (`AGENTS.md`)    |
| **System**  | The real app: real boundaries, real store, real services. | the running environment |

E2E is the only layer that exercises the whole path. Unit and integration tests stay the bulk of coverage; e2e proves the seams between them hold.

## Do not use

- To replace unit or integration tests, or to cover every edge case (that is the task-level tests' job).
- Per task or per feature, or as a node of the implement/verify/review loop.
- Automatically, because a phase ended or a release is near. If the user wants it then, they ask.

## Core rules

- **Journey-based, not page-based.** Test what a user is trying to do ("a new member joins and sees the board"), not "the settings page renders".
- **The critical few.** The core flow, sign-in, payment, and data-loss paths, and the seams between components. A huge, slow suite gets skipped.
- **Against the real system.** Real boundaries, real store, real services. Mocking the seams defeats the purpose.
- **Deterministic and independent.** Each journey sets up and cleans up its own data and does not depend on run order. No bare sleeps; wait for a real condition.
- **Stable selectors.** Roles, user-visible text, or test ids the app commits to. Never brittle CSS paths or nth-child.
- **Flaky is broken.** Fix the cause, or quarantine with a tracked issue. Never leave it red and move on, and never re-run until green.
- **A red journey is a finding.** Report it with the failing step and evidence. Do not explain it away, and do not fix product code yourself.

## Procedure

1. **Scope with the user.** "Test the whole project" means the project's critical journeys, not every path. Take them from the PRD and the specs' acceptance criteria, rank them (core flow, then money, security, and data-loss paths, then risky seams), and write each down with `references/journey-template.md`. If the scope, the environment, or the existing e2e tool is unclear, ask one question at a time; otherwise state your call and continue.
2. **Stand up the real environment.** Use the project's own dev or compose command (`AGENTS.md`), seeded with representative data. Record which environment (local, preview, staging) so a failure is reproducible.
3. **Write the journeys** in `e2e/`, with the project's existing e2e tool and conventions. Do not introduce a second framework.
4. **Run the full suite**, not just the journey you wrote. A headless run is preferred; a local run is fine for an on-demand check. If the suite is meant to be kept, offer to wire it into CI.
5. **Triage every failure:**
   - **Product bug:** the assembled system is broken. Report it; the user decides whether to hand it to `implement`.
   - **Test bug:** wrong selector, brittle wait. Fix the test.
   - **Flaky:** find the cause; quarantine with a tracked issue only if the root cause needs its own work.
6. **Report the evidence:** command, environment, and pass or fail per journey, including the failures. Do not summarize them away. What happens next is the user's call; start no other skill.

## Red flags

| Thought                                      | Reality                                                             |
| -------------------------------------------- | ------------------------------------------------------------------- |
| "Unit tests pass, so the product works"      | Units pass in isolation; e2e proves they work together.             |
| "Let's e2e every edge case"                  | That is the unit suite's job. Cover the critical few.               |
| "Just mock the API, e2e is slow"             | Mocking the seams tests nothing.                                    |
| "Add a sleep, it's flaky"                    | A sleep hides the race. Wait for the real condition.                |
| "It fails sometimes, re-run it"              | Flaky is a defect. Re-running until green is how bugs ship.         |
| "A phase just ended, I'll run e2e"           | On demand only. Wait for the user to ask.                           |
| "I'll fix the product bug the journey found" | Report it. Fixing goes through `implement`, by the user's decision. |

## Completion check

- [ ] The user asked for this run, and the scope was agreed or stated.
- [ ] Journeys came from the PRD and acceptance criteria: the critical few.
- [ ] Tests ran against the real assembled system, deterministic and independent, with stable selectors and no bare sleeps.
- [ ] The full suite ran, and every failure was triaged as product bug, test bug, or flaky.
- [ ] The report states command, environment, and per-journey results; no other skill was started and no workflow state changed.
