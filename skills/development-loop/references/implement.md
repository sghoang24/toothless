# The `implement` Step (and its `verify` evidence)

The `implement` step builds **one unit** and proves it green. It is the workhorse of the loop `[implement -> verify -> review]`, and it carries the `verify` step's fresh-evidence bar with it: the implementer writes the test first, watches it fail, makes it pass, runs the full suite, and puts the evidence in its report. There is no separate verifier subagent — the evidence is held here and read by the controller.

One invocation = **one unit**: usually a single task from `specs/NNN-slug/tasks.md`, or a **batch** of small same-shape tasks dispatched together (see the parent skill's _Batching_). Not a feature, not a phase. The unit's `Verify:` line, or its outcome and acceptance criteria, is the definition of done.

**For a batch:** the brief lists every member — its file and its change. Make every listed change, nothing else, and report per member. A batch is behavior-free by definition, so there is **no new RED**: the existing suite is the evidence that each edit is faithful. If one listed member turns out to need a real behavior change, a judgment call, or its own test, do **not** improvise — report `BLOCKED` for that member; it should have been dispatched on its own.

## Standalone vs orchestrated

- **Do the work yourself.** Never dispatch another implementer. Inside `development-loop` this runs as the `implementer` subagent — dispatching from here would recurse.
- **You do not run the loop.** Review belongs to the controller (`development-loop`). When the task's tests pass and the evidence is recorded, report and stop.
- **Standalone** (called directly): after reporting, say in one line that `review` comes next and ask whether to start. Do not invoke it yourself.
- **Orchestrated**: report and stop; the controller decides what runs next. Do not write `status.md` or tick `tasks.md` unless the controller's brief tells you to; put what it needs in the report.

## Inputs

- The **brief** the controller handed you: the task's requirements, with the exact values to use verbatim. It is your single source of requirements.
- The feature's `## Test strategy` (in `specs/NNN-slug/tasks.md`) for the command, layers, and shared seams.
- The parts of `docs/architecture/architecture.md` and the feature's `design.md` the task touches.
- Interfaces and decisions from earlier tasks that the brief cannot know (the controller supplies these).

If the task has no outcome or nothing to verify against, it is not sized: report it as `BLOCKED` and send it back to `planning`. Do not guess.

## Core Rules

- **Test first, always** (`AGENTS.md`) when the unit changes behavior. RED → GREEN → REFACTOR. Code written before its test is deleted, not kept as reference. A **batch** changes no behavior, so it has no RED of its own: run the existing suite and show it still green.
- **Right test, right layer.** Take the layer and boundary from the task's `Tests:` line and the feature's strategy. If either is missing, say so in your report — the controller settles it with `test-strategy`.
- **Green means the suite, not your file.** Run the project's full declared test command before claiming the task done. Output pristine — a stray warning is a finding.
- **One task = one commit-set on the feature's branch.** The branch is `feat/NNN-slug` — one feature = one branch = one PR (`AGENTS.md`), so reuse it across the feature's tasks. Keep **this task's** diff under about 300 lines; if the work outgrows that, stop and report: it is two tasks.
- **Small Conventional Commits** (`feat:`, `fix:`, `test:`, `chore:`).
- **Business logic in the service layer**, never in the controller or interface. Type hints, no bare `except`, structured logging.
- **Boundaries:** no secrets, no `.env` edits, no edited existing migrations (new ones only), no new dependency without asking.
- **Stay in the task's lane.** Unrelated cleanup is out of scope; note it in your report and move on.
- **Escalate rather than guess.** If the task needs an architectural decision, you cannot find clarity, or it wants to restructure more than the plan anticipated, stop and report `BLOCKED` or `NEEDS_CONTEXT`. Bad work is worse than no work.

## Procedure

1. **Load.** Read the brief and its inputs. If anything needed to start is missing, ask now — before writing code.
2. **Branch.** Reuse the feature's branch `feat/NNN-slug` (one feature = one branch). Create it only if this is the feature's first task. Start from a clean tree and a green baseline; name a red baseline you did not cause in your report.
3. **RED.** Write one minimal test for the behavior the unit names and run it. Watch it fail **for the expected reason** (feature missing, not a typo or an error). A test that passes immediately tests existing behavior: fix the test. Name the production change that would make it fail, before you write it. **For a batch:** skip RED — a batch adds no behavior. Run the existing suite first to confirm the batch starts green; that green is the baseline you must preserve.
4. **GREEN.** Write the simplest code that passes, in the service layer. No extra features, no speculative abstraction. Run the focused test, then the full suite. **For a batch:** make each listed edit exactly as the brief states it, one file at a time, and re-run the suite.
5. **REFACTOR.** Improve names and remove duplication without changing behavior; re-run the suite after each step. New behavior means a new test, back to RED.
6. **Repeat** per behavior (or per member, for a batch) until the unit's acceptance criteria are satisfied. Commit in logical chunks as you go.
7. **Verify the evidence — the `verify` gate.** Before reporting, confirm what you are about to claim with a run in this session, not a memory of green:
   - **Identify** the claim and the exact command that proves it — the project's declared command (`test-strategy` found it, per `AGENTS.md`), plus whatever the unit's `Verify:` line names.
   - **Run** it fresh and in full. Do not narrow scope to the files you touched.
   - **Read** the entire output: exit code, failure count. A single failing test is red, however good the rest looks.
   - **Match** output to claim. If it does not confirm the claim, the claim is false — say the real status, naming what failed.
   - Walk the unit's `Verify:` line item by item. Report gaps; do not report a vibe.
   - **For a batch:** also confirm every listed member is present in the diff, and that nothing outside the list changed.
8. **Report and stop.** Write the full report to the report file the controller named, then reply with the short status contract (below). Include the TDD evidence, the full-suite result, files changed, diff size, anything out of scope you noticed, and any blocker. A design gap goes back to `system-design`, not into your code.

## The verify gate, in one line

**NO COMPLETION CLAIM WITHOUT FRESH VERIFICATION EVIDENCE.** "Should pass", "probably fixed", and "looks right" are not verification. A silently-skipped red is a false report; a red reported is fine.

## Report contract

Write the full report to the report file. It carries:

- What you implemented (or attempted, if blocked).
- **TDD evidence:** RED — the command, the relevant failing output, and why that failure was expected; GREEN — the command and the passing output.
- The full-suite command and result; every failure named, including pre-existing ones.
- Files changed; the diff size.
- Self-review findings (below) and any concerns.

Then reply with **only** (under ~15 lines — the detail lives in the file):

| Field            | Value                                                          |
| ---------------- | -------------------------------------------------------------- |
| **Status**       | `DONE` \| `DONE_WITH_CONCERNS` \| `BLOCKED` \| `NEEDS_CONTEXT` |
| **Commits**      | short SHA + subject                                            |
| **Test summary** | one line, e.g. "14/14 passing, output pristine"                |
| **Concerns**     | optional                                                       |
| **Report file**  | its path                                                       |

Use `DONE_WITH_CONCERNS` if you finished but have doubts about correctness; `BLOCKED` if you cannot complete it; `NEEDS_CONTEXT` if you need information that was not provided. Never silently produce work you are unsure about.

## Self-review before reporting

Read your own diff with fresh eyes. **Completeness:** did I implement everything the brief asked, no more? **Quality:** are names accurate, the code clean? **Discipline:** did I avoid overbuilding and follow existing patterns? **Testing:** do the tests verify behavior (not mocks), cover the edge cases, and is the output pristine? Fix what you find before reporting.

## Red Flags

| Thought                                      | Reality                                                                      |
| -------------------------------------------- | ---------------------------------------------------------------------------- |
| "Too simple to test" / "I'll test after"     | Tests-after pass immediately and prove nothing. Delete the code, start over. |
| "I tested it by hand"                        | Ad-hoc and unrepeatable. Automate it.                                        |
| "It passed a minute ago"                     | Stale. A fresh run in this session is the only evidence.                     |
| "My new test is green, the task is done"     | A green file is not a green suite. Run the full command.                     |
| "One unrelated test was red, not my problem" | Report it by name. You watched it fail; silence is a false report.           |
| "I'll fold in this adjacent cleanup"         | Out of scope. It is another task.                                            |
| "It's a bit over 300 lines, fine"            | It is two tasks. Oversized diffs do not get reviewed properly.               |
| "I'll put the logic in the controller"       | `AGENTS.md`: business logic lives in the service layer.                      |
| "I'll dispatch a reviewer to check my work"  | Review is the controller's job. Report instead.                              |
| "I'll guess at the ambiguity and move on"    | Ask. `NEEDS_CONTEXT` costs a minute; a wrong guess costs a cycle.            |
