---
name: development-loop
description: "Use to build features, from an approved task list to landed, reviewed code. The goal is to own the loop [implement -> review] per unit — usually one task, or a batch of small same-shape tasks: dispatch a fresh implementer subagent (which writes tests first, proves them green, and fills a four-lens gate: code, security, ui, ux), then a fresh reviewer subagent (which confirms or flips each lens), cap the cycles, and run a final whole-branch review. You are not allowed to let a unit skip the gate or the review, to waive a Critical/Important finding, or to exceed the cycle cap without asking the human. Trigger phrases: build the feature, run the loop, implement the tasks, start development, work through tasks.md, dev loop, batch small tasks."
argument-hint: "Point at the feature, e.g. specs/001-auth/ (its tasks.md)"
---

# Development Loop

`planning` cuts a feature into tasks; `test-strategy` settles how they are tested. This skill drives those tasks to landed code — one **unit** at a time, each built test-first and reviewed by a fresh session.

**Two steps: `implement -> review`.** There is no `verify` stage and no verifier subagent — the implementer proves the work green in its own report, and the controller reads that evidence. The proof itself lives in a **four-lens gate** (`code`, `security`, `ui`, `ux`) the implementer fills and the reviewer confirms.

**Self-contained.** Use it to build any feature with an approved task list, whether or not `fight` is driving; `fight` simply calls it when it's active. It is the loop's single source of truth: the cycle cap, the transitions, and the dispatch rules live here, not in `fight` and not in the steps below.

**The loop runs per unit, not per feature.** One cycle = one unit taken through `implement -> review`. A **unit** is what one dispatch covers: a single task, or a batch of small same-shape tasks (see _Batching_). A feature with eight tasks runs the loop up to eight times, fewer when tasks batch.

## The Model

| Thing            | What it is                                                                            | Where it lives                                |
| ---------------- | ------------------------------------------------------------------------------------- | --------------------------------------------- |
| **Task**         | One reviewable change (~300 lines).                                                   | `specs/NNN-slug/tasks.md`                     |
| **Batch**        | Several small same-shape tasks dispatched as one unit.                                | `specs/NNN-slug/tasks.md`                     |
| **Unit**         | What one dispatch covers: a single task, or a batch. The loop's unit.                 | `specs/NNN-slug/status.md` (`current_unit`)   |
| **Cycle**        | One unit's round through `implement -> review`.                                       | `specs/NNN-slug/status.md` (`cycle`)          |
| **Gate**         | Four lens rows (`code`, `security`, `ui`, `ux`) with green/n-a + evidence.            | the report, then the unit review              |
| **Brief**        | The unit's requirements, extracted to a file for the implementer.                     | see `references/brief-and-report.md`          |
| **Report**       | The implementer's full account, written to a file.                                    | see `references/brief-and-report.md`          |
| **Unit review**  | One reviewer's verdict for one unit; never overwritten.                               | `<scratch>/specs-NNN-slug/unit-<N>-review.md` |
| **Context pack** | The feature's review scaffold — spec criteria, interfaces, conventions — in one file. | `<scratch>/specs-NNN-slug/review-context.md`  |
| **Branch**       | One feature = one branch = one PR.                                                    | `feat/NNN-slug`                               |

One unit = **one** implementer dispatch and **one** reviewer dispatch. For a single task that is one task; for a batch it is the whole batch, reviewed as one diff.

### The two steps and the gate

The step names are the shared vocabulary across the pack. Other skills say "back to `implement`" or "`review` judges the diff" — they mean these:

| Step          | What it does                                                      | Reference                 |
| ------------- | ----------------------------------------------------------------- | ------------------------- |
| **implement** | Builds one unit test-first, proves it green, and fills the gate.  | `references/implement.md` |
| **review**    | Confirms or flips each gate lens on the diff, in a fresh session. | `references/review.md`    |

### The four-lens gate

Every unit is judged on four lenses; the implementer fills each row and the reviewer confirms or flips it. This is the whole quality surface.

| Lens         | The question                                                              | Evidence the row needs                                |
| ------------ | ------------------------------------------------------------------------- | ----------------------------------------------------- |
| **code**     | Does it do what the task says, cleanly and in the right layer?            | which test covers it, file:line                       |
| **security** | Does the diff open a hole — untrusted input, authz, secrets, deserialize? | the trust boundary it touches, or why none is touched |
| **ui**       | If there are screens, do they match `docs/design/`?                       | the screen/state and its design reference             |
| **ux**       | Do flows, states (empty/loading/error), and messages hold up?             | the flow/state walked                                 |

| Row value | Means                                                                                        |
| --------- | -------------------------------------------------------------------------------------------- |
| `green`   | Done, with the evidence named in the row. A row without evidence is not green.               |
| `n/a`     | Not applicable **and** the reason is stated (`no user-facing surface`, `no trust boundary`). |
| `open`    | Known gap, with the severity and why it is not being fixed here.                             |

The gate is **small on purpose**: a row is one line plus its evidence, not a paragraph. A backend task's `ui`/`ux` rows are `n/a: no user-facing surface` — that is a valid answer, not a failure, and it does not go back to the implementer.

**Security here is the diff, not the audit.** This lens scans what the unit just changed — injection, missing authz, a leaked secret, unsafe deserialization. The deeper work (threat model, dependency tree, privacy) is `security-and-hardening`, a standalone pass at a milestone, not this loop.

**The verdict is the worst lens.** Route on the single worst row across the four; there are not four verdicts. That is what keeps the cycle cap meaningful and the send-back unambiguous.

## When to Use

- An approved task list exists (`specs/NNN-slug/tasks.md`), the design is agreed, and at least one task is unbuilt.

**Do NOT use** to decide what to build (`brainstorming`), slice features into phases (`roadmap`), design the system (`system-design`), cut a feature into tasks (`planning`), settle the test strategy (`test-strategy`), or land the branch (`closing-a-phase` — the loop exits into it). Do not use it for a single task with no feature context; that is still the loop, but say so and run one cycle.

## Core Rules

The loop's hard gates. Detail lives in the references, not here.

- **Cycle cap: 3, per unit.** A unit sent back three times goes to the human — park it with a written ruling and move on. The cap is per unit, so one stuck unit does not starve the feature.
- **Each cycle changes the approach.** Before cycle _n_, name what was wrong with cycle _n−1_ and what is different this time. No new hypothesis means STOP.
- **No unit skips the gate or the review.** All four rows answered, and a fresh session judges the diff. "Trivial" is not an exemption — a run of trivial tasks is a batch.
- **The reviewer is a fresh session** — never the author's, never one the implementer spawned.
- **The references own the detail; do not restate or relax it.** The reviewer's limits, tiers, and report shape are in `references/review.md`; the brief/report/context-pack contract is in `references/brief-and-report.md`.
- **No branch without approval to land.** `closing-a-phase` raises the MR/PR; merging needs explicit human approval.

## Batching

A run of small, same-shape work — the same mechanical edit across files — is **one unit**, not one per file. That is the safe path, not a shortcut.

**Allowed only when every member is all of:** same shape, independent (no `depends_on`, no shared file), behavior-free (no new RED; the existing suite proves it), one review surface, and under ~300 lines together. Put every member in **one brief**, record `BASE` once, dispatch **one** implementer, review the whole diff as one unit.

**Guards:** a member that is big, risky, or ambiguous gets pulled out and dispatched alone — a batch must not let a hard task hide among easy ones. The whole batch goes back together on a Critical/Important finding. The cap is per unit, so batching six tasks buys three cycles for the batch, not eighteen.

Batching is **one** agent on **one** branch; `dispatching-parallel-agents` is several agents on several branches. They compose.

### Reviewing small units together

A review's cost is mostly fixed — the fresh session, the context, the report — so two tiny units reviewed separately pay it twice for almost no extra scrutiny. When the next unit is ready and both are **small** (each ≤ ~50 lines, no contract change) and **adjacent** in the same area, review them as **one dispatch** rather than two.

This is not batching (batching is one _implementer_ dispatch). Each unit is still implemented, verified, and committed on its own; only the **reviewer** sees the two diffs together, as one combined `BASE..HEAD` range, at the higher of the two tiers. Guards:

- **Small and contract-free only.** As soon as one unit changes an interface, schema, API, config key, or migration, review that one alone at Standard.
- **Both findings stay separate.** A Critical/Important on either diff sends **that** unit back; the other keeps its done state and is not re-reviewed.
- **Never skip the roll-up.** Two-unit review is convenience; the roll-up is what catches coverage gaps and contradictions.

## Procedure

### 1. Load the feature and its tasks

Read `specs/_state.md` (current phase/feature) and the feature's `specs/NNN-slug/status.md` (`status`, `current_unit`, `cycle`, `cycle_cap`, `next`, `notes`). If that file is missing, create it from the feature's `_roadmap.md` row before starting, and say so. Read `specs/NNN-slug/tasks.md` — the task list, each task's `Verify:` line, and the feature's `## Test strategy` section. Read the spec (`specs/NNN-slug/spec.md`, `status: approved`) and the parts of `docs/architecture/architecture.md` the tasks touch.

If a task has no outcome or nothing to verify against, it is not sized: send it back to `planning` before building anything.

Then group the pending tasks into units: keep every task that needs its own judgment, tests, or review surface as a unit of one, and batch the runs of small same-shape work (see _Batching_).

Then build the feature's **review context pack** (`review-context.md`, see `references/brief-and-report.md`) once — the spec criteria, the interfaces and boundaries the feature touches, conventions, and the test strategy in one file. Every unit review and the final review read it instead of the separate documents; rebuild it only when the spec revision, design, or strategy changes.

### 2. Set up the branch

Work on `feat/NNN-slug`. Start from a clean tree and a green baseline; if the baseline is red for a reason you did not cause, name it before you begin.

### 3. Run the loop, one unit at a time

For each unit, in `depends_on` order (a batch is ready only when every member is):

1. **Record `BASE`** — `git rev-parse HEAD` — **before** dispatching. The review diff and any fix round need it. Never use `HEAD~1`: it silently drops all but the last commit of a multi-commit unit.
2. **Build the brief and dispatch the implementer.** Extract the unit's requirements to a brief file (see `references/brief-and-report.md`) and dispatch **one** fresh implementer subagent with `references/implementer-prompt.md`. Give it the brief, the interfaces earlier units created, the global constraints, and the report-file path. Never the session's history, never the whole plan.
3. **Read the report and check the gate.** The implementer returns a short status (`DONE`, `DONE_WITH_CONCERNS`, `BLOCKED`, `NEEDS_CONTEXT`) and a report file carrying the four lens rows and the test evidence. On `BLOCKED` or `NEEDS_CONTEXT`, answer the question and re-dispatch rather than guessing. Treat the report as claims: check the diff, re-run anything you doubt. A green row without evidence, a missing row, or an `n/a` with no reason → back to `implement` (cycle).
4. **Review the unit.** Generate the diff for `BASE..HEAD` and dispatch **one** fresh reviewer subagent with `references/reviewer-prompt.md` and `references/review.md`. Give it the feature's **context pack** (`review-context.md`) and the implementer's gate table. The reviewer confirms or flips each of the four rows and returns **one** verdict, the worst lens. For a batch, the reviewer sees every member's files — the review is looser on nothing, only fewer in number. Never the author's session, and each unit writes its **own** `unit-<N>-review.md` — never one shared file, or the roll-up loses the earlier verdicts. When the next unit is small, adjacent, and contract-free, review the two together in one dispatch (see _Reviewing small units together_).
5. **Route the outcome.** Findings in the code → `implement`. Findings in the task list → `planning`. Findings in the design → `system-design`. Findings in the requirement → `roadmap` (a spec revision, delta only, then its Lavish gate again); only if the PRD itself is wrong does it go to `brainstorming`.
6. **Close the unit.** A verdict of clean (or only Minor findings) → the unit is done. Tick **each member's** checkbox in `tasks.md`, record it in `specs/NNN-slug/status.md` (`next`, `notes`, the four gate statuses), and move to the next unit. The **branch** is not landed here — it lands once per feature, after the final review (Procedure step 5).

### 4. Track the cycle

Increment `cycle` in `specs/NNN-slug/status.md` **only when a unit is sent back** (a gate row that is not green with evidence, or a Critical/Important review finding). Set `next` to the return target and add a one-line note. When `cycle` reaches `cycle_cap`:

- Park the open findings with a one-line ruling each (what you decided, why, what it costs if wrong).
- Set the feature's `status` to `blocked` only if the whole feature is stuck; otherwise note the parked unit and move on.
- Tell the human, and stop asking the loop to converge — past the cap, rounds do not converge; the failure is structural.

### 5. Roll up the reviews

After **every** unit is done, fold the unit reviews into one feature verdict (`references/final-review.md`). This is a **read of the unit reviews**, not a second pass over the branch: it checks that every acceptance criterion is covered, that no two units contradict each other, and that every `open` row and parked finding is accounted for. Do not re-review the diff. Fix what it finds in a single wave, then hand to `closing-a-phase`.

### 6. Exit the loop

The loop is done when every unit is done or parked-with-ruling and the roll-up is clean. Then `closing-a-phase` lands the branch: it raises the MR/PR for the human — merging needs explicit human approval — appends the ledger row, and cleans up the worktree. Then `deploy`, at the milestone.

One feature = one branch = one PR, so there is exactly **one** landing per feature, after the final review — not one per unit.

Update the feature's `specs/NNN-slug/status.md` (`next`, `notes`) as you go, and the project ledger `specs/_status.md` on each landing.

## Red Flags

| Thought                                              | Reality                                                                               |
| ---------------------------------------------------- | ------------------------------------------------------------------------------------- |
| "This unit is trivial, skip the gate and review"     | Every unit fills the gate and gets reviewed. Trivial is where the shortcut ships.     |
| "Tests pass, so the unit is done"                    | Green tests are one gate row. The other three still need an answer.                   |
| "ui/ux don't apply, so leave the rows out"           | Write `n/a` with the reason. A missing row is indistinguishable from a skipped one.   |
| "The implementer marked it green"                    | Green needs evidence on the row. A bare green is a claim, not a gate pass.            |
| "I'll skip security, the other skill handles it"     | That skill is the milestone audit. This lens checks only the diff — and it runs here. |
| "Cycle 4 is fine, we're close"                       | The cap is 3 per unit. Park with a ruling and ask the human.                          |
| "Same fix, one more try"                             | Each cycle changes the approach. No new hypothesis means STOP.                        |
| "I'll just tell the reviewer to ignore that one"     | Never pre-judge a finding. Let it surface, then adjudicate.                           |
| "Per-unit reviews caught everything, skip the final" | Units can't see coverage gaps or contradictions between units. Roll them up.          |
| "One unit is stuck, the feature is doomed"           | Park it with a ruling and move to the next unit. The cap is per unit.                 |
| "These 6 tiny tasks need 6 dispatches to be safe"    | Same-shape, independent, behavior-free → one batch. That is the safe path.            |
| "I'll tuck this big task in with the tiny ones"      | Pull it out. A batch must not let a hard task hide among easy ones.                   |

## Completion Check

The loop is trustworthy when ALL are true:

- [ ] Every unit had all four gate rows answered with evidence or a stated `n/a`, and was reviewed in a fresh session.
- [ ] Each review returned **one** verdict (the worst row) and read the feature's `review-context.md`.
- [ ] The cycle cap was honored; any unit past it is parked with a written ruling and the human was told.
- [ ] Briefs and reports were files, never pasted history; any batch met every guard.
- [ ] The final roll-up ran after the last unit, from the unit reviews — not a fresh diff pass — and its findings were fixed or parked.
- [ ] `status.md` reflects the current unit and cycle; the loop exited into `closing-a-phase` once for the feature.
