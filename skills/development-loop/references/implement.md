# The `implement` Step (with its green evidence)

The `implement` step builds **one unit**, proves it green, and fills the four-lens gate. It is one of the loop's two steps — `implement -> review` — and it carries the whole green-evidence bar with it: the implementer writes the test first, watches it fail, makes it pass, runs the full suite, and puts the evidence in its report and its gate rows. There is no separate verifier stage and no separate subagent.

One invocation = **one unit**: usually a single task from `specs/NNN-slug/tasks.md`, or a **batch** of small same-shape tasks dispatched together (see the parent skill's _Batching_). Not a feature, not a phase. The unit's `Verify:` line, or its outcome and acceptance criteria, is the definition of done.

**For a batch:** the brief lists every member — its file and its change. Make every listed change, nothing else, and report per member. A batch is behavior-free by definition, so there is **no new RED**: the existing suite is the evidence that each edit is faithful. If one listed member turns out to need a real behavior change, a judgment call, or its own test, do **not** improvise — report `BLOCKED` for that member; it should have been dispatched on its own.

## Inputs and stance

- The **brief** is your single source of requirements, with the exact values to use verbatim.
- The feature's `## Test strategy` (command, layers, seams), plus the parts of `docs/architecture/architecture.md` and `design.md` the task touches.
- Interfaces from earlier units that the brief cannot know (the controller supplies them).
- **Do the work yourself** — never dispatch another implementer; inside `development-loop` you _are_ the implementer subagent.
- **You do not run the loop.** Review is the controller's job. Report and stop; do not write `status.md` or tick `tasks.md` unless the brief says to.
- **Standalone** (called directly): after reporting, say in one line that review comes next and ask whether to start.

If the task has no outcome or nothing to verify against, it is not sized: report `BLOCKED` and send it back to `planning`.

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
7. **Prove the evidence.** Before reporting, re-run in this session — not a memory of green — via the project's declared command plus whatever the unit's `Verify:` line names. Run it **fresh and in full**; do not narrow scope to the files you touched. Read the whole output: one failing test is red however good the rest looks. Match output to claim — if it does not confirm, say the real status and name what failed. Walk the `Verify:` line item by item. For a batch, also confirm every listed member is in the diff and nothing else changed.
8. **Report and stop.** Write the full report to the report file, then reply with the short status. A design gap goes back to `system-design`, not into your code.

## The green bar, in one line

**NO COMPLETION CLAIM WITHOUT FRESH VERIFICATION EVIDENCE.** "Should pass", "probably fixed", and "looks right" are not verification. A silently-skipped red is a false report; a red reported is fine.

## Fill the gate, then report

Before you write the report, fill one row per lens with **evidence**:

| Lens     | green means                                                                                      |
| -------- | ------------------------------------------------------------------------------------------------ |
| code     | the test that covers the behavior, and where it lives (file:line)                                |
| security | the trust boundary this diff touches and how it is handled — or `n/a: no trust boundary touched` |
| ui       | the screen/state changed and its `docs/design/` reference — or `n/a: no user-facing surface`     |
| ux       | the flow/state walked and the message shown — or `n/a` with the reason                           |

`n/a` is a complete answer when it names why. A `green` row without evidence is not green — leave it `open` and say what is missing. Do not invent work to fill a row.

## Report contract

Write the full report to the report file. It carries:

- What you implemented (or attempted, if blocked).
- **TDD evidence:** RED — the command, the relevant failing output, and why that failure was expected; GREEN — the command and the passing output.
- The full-suite command and result; every failure named, including pre-existing ones.
- **The four-lens gate table** — one row per lens, `green` with evidence or `n/a` with the reason.
- Files changed; the diff size.
- Self-review findings (below) and any concerns.

Then reply with **only** the short status (under ~15 lines — the detail lives in the file): `DONE` | `DONE_WITH_CONCERNS` | `BLOCKED` | `NEEDS_CONTEXT`, the commits (short SHA + subject), a one-line test summary, any concerns, and the report path. Use `DONE_WITH_CONCERNS` if you finished but have doubts; never silently produce work you are unsure about.

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
| "It's a bit over 300 lines, fine"            | It is two tasks. Oversized diffs do not get reviewed properly.               |
| "I'll put the logic in the controller"       | `AGENTS.md`: business logic lives in the service layer.                      |
| "I'll dispatch a reviewer to check my work"  | Review is the controller's job. Report instead.                              |
| "I'll guess at the ambiguity and move on"    | Ask. `NEEDS_CONTEXT` costs a minute; a wrong guess costs a cycle.            |
