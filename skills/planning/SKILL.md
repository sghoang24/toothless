---
name: planning
description: "Use after a feature's spec (or a phase's specs) is approved and before any implementation, to turn the acceptance criteria into an ordered list of PR-sized tasks in specs/NNN-slug/tasks.md, each traced to a criterion and carrying its own tests. Plans one feature, or every feature of the current phase in one run. The spec is required; a design is optional enrichment — without one you plan from the criteria and record the technical calls as plan calls. Also use when a spec has a new revision (plan only the new/changed work) or when review sends the feature back (fix the task list, keep what is done). Triggers: plan this feature, plan the phase, plan all features, break into tasks, task breakdown, spec changed. Not for deciding what to build (`brainstorming`), slicing a project into features and phases (`roadmap`), or choosing the shape of the solution (`system-design`)."
argument-hint: "Optional: a feature folder (specs/003-upload-invoice) or a phase ('phase', 'current phase'). Defaults to the current phase in specs/_state.md."
---

# Planning

Plans **one feature**, or **every feature of the current phase** in one run. `brainstorming` decided what and why, `roadmap` decided which features and in what phases, `system-design` decided the shape of the solution. Planning decides **in what order, in what slices**: a task list a builder can execute without re-reading the conversation.

Workflow position: `brainstorming` → `roadmap` → `system-design` → **`planning`** (with `test-strategy`) → `development-loop` (`implement -> verify -> review`, per unit). It also works standalone on any approved spec; it never depends on the other skills being present.

What this skill does **not** do: group features into phases (that is `roadmap`), make design decisions (that is `system-design`), or write code.

## Step 0: Find the inputs and the mode

Read, in parallel:

- the feature folder: `status.md`, `spec.md` (the current spec — `status: approved` required), `tasks.md`, `review.md` and any `design.md` if present
- `docs/design/` for the agreed UI, `docs/architecture/architecture.md` for the system design and relevant ADRs; `AGENTS.md` (PR size, branch naming, worktree flow); the code the feature touches
- `specs/_state.md` for `current_phase` / `current_feature`, when nothing was named
- in **phase mode**, every feature folder of the current phase, and that phase's rows in `specs/_roadmap.md`

Pick the mode:

| Mode                     | When                                                                     | What you do                                        |
| ------------------------ | ------------------------------------------------------------------------ | -------------------------------------------------- |
| **Phase plan**           | Every feature of the current phase has an approved spec                   | One task list per feature, cut in phase order      |
| **First plan**           | No `tasks.md` for the feature                                            | Full task list for that one feature                |
| **Revision delta**       | `spec.md`'s `revision` is newer than the one the tasks were planned from | Plan only NEW/CHANGED requirements                 |
| **Re-entry from review** | `review.md` says the task list was wrong or incomplete                   | Fix the list; never redo finished tasks            |

Pick **Phase plan** when the user says "plan the phase", "plan all the features", or when the pipeline reaches Plan. Pick a single-feature mode when only one feature is named.

**The spec is the input; the design is an enrichment.** Acceptance criteria are enough to cut tasks. When a design exists (`design.md`, `docs/architecture/architecture.md`, `docs/design/`), read it and let it sharpen the task's files, interfaces, and order. When it does not, plan from the criteria, record the technical calls you had to make as **plan calls** in `tasks.md`, and continue — a missing design is not a reason to stop planning.

## Phase plan mode

Phase planning is the same work, repeated over the phase's features **without** stopping to approve each one separately. It is the right default in the pipeline: the phase's specs are all approved at once, so the criteria are all present and there is nothing gained by waiting.

1. **Load the phase.** Read `specs/_state.md` for `current_phase`, then that phase's feature rows in `specs/_roadmap.md` (ID, `covers`, `depends_on`). Read each feature's `spec.md`, and its `design.md` **when it exists**.
2. **Gate the input, per feature.** Every feature needs `spec.md` with `status: approved`. That is the only hard requirement. A feature without one is **excluded** from this run and reported, with the route back (`roadmap`). A feature whose design is missing is **not** excluded: plan it from its criteria and record the technical calls as **plan calls**. If a feature genuinely cannot be cut without a design decision (a new interface, a data-model change nobody has settled), say which one and send that feature to `system-design` — not the whole phase.
3. **Cut the tasks per feature**, in **phase dependency order** (a feature whose `depends_on` is still unbuilt comes later). Each feature's list stands alone in its own `tasks.md`; nothing moves between features, because a feature is the unit the loop, the branch, and the PR run on.
4. **Order across features too.** State the phase's feature sequence and which features are independent enough to run in parallel worktrees (`git-worktrees`, `dispatching-parallel-agents`). Record it once, in `specs/_roadmap.md`'s phase section or the phase's first feature's notes — not duplicated per feature.
5. **Settle the test strategy for the phase once** with `test-strategy`: one shared command, one set of shared seams, and per feature the layer each behavior is pinned at. A shared seam decided per feature is how the phase ends up with three different fakes for one clock.
6. **Check traceability for every feature** — every `AC-n` with a task and a named test, every task traced back — then show the phase as **one approval**: a table of features with task counts, sizes, dependencies, parallel tracks, the shared test strategy, and every risk.
7. **Write every `tasks.md`** and update each planned feature's `status.md` (`status: doing`, `next` = its first task). Do not run the loop from here unless the user asked; hand off per the pipeline rule below.

Stop and re-plan only the affected feature when a later revision or a review finding touches it — phase planning is a one-time sweep, not a live aggregate.

## Preconditions (route back, do not improvise)

- No `specs/NNN-slug/spec.md`, or its frontmatter reads `status: spec` → gate two has not passed. Send it back to `roadmap` (which re-dispatches the spec writer and re-runs its Lavish review). Do not plan against an unapproved spec. This is the only input that blocks planning.
- An approved spec but no testable acceptance criteria → the same route back to `roadmap`.
- Standalone with no roadmap (no `specs/_state.md`): a spec written by `brainstorming` feature mode is acceptable input.
- **A missing design does not block planning.** Plan from the criteria and mark each technical call as a **plan call** in `tasks.md` for the user to check. Send a single feature to `system-design` only when a criterion cannot be cut into tasks without a decision nobody has made (a new interface, a data-model change, a screen).
- If this project has no design step at all (standalone use), make the smallest technical calls you need, and mark each as a **plan call** in `tasks.md` for the user to check.

For the checks above and for any question worth asking before or during planning, use `references/question-bank.md`.

## Core Rules

- **Plan from the spec. Use the design when it exists.** Every task traces to an acceptance criterion (`AC-n`). A design element is a bonus that sharpens the task's files, boundary, and order — never the reason a task exists.
- **No new design decisions of consequence.** If the design covers the area, follow it. If it does not, either mark a **plan call** (and let the user check it) or send that one feature to `system-design` when the decision is hard to reverse. Never silently invent a schema, an API shape, or a boundary inside a task.
- **One task ≈ one reviewable PR** (about 300 lines of diff per `AGENTS.md`). Do not over-split. See `references/task-sizing.md`.
- **Tests ship with the task that makes a criterion true.** Do not create a separate "write tests later" task. Every `AC-n` is covered by at least one task with a named test. Planning **names which** test pins each task; the **layer, seams, and command** are settled by `test-strategy` (the `## Test strategy` section of `tasks.md`), which runs as the last step of this phase.
- **Order by dependency:** data model → core logic → interface/API → UI → wiring and polish. Mark tasks that can run in parallel.
- **Never re-plan what already exists.** Unchanged requirements are already built or planned. A phase plan that finds a feature already planned keeps that list and plans only the gaps.
- **Never reopen a shipped feature.** If the feature is `done` and shipped, send the change to `roadmap` as a follow-up feature.
- **No code before an approved plan.** Scaffolding and dependency installs count as code.
- **The user decides.** Propose the task list; they confirm or adjust. One question per message when you must ask; otherwise state your call. Questions worth asking, and the layout and ownership rules they lead to, are in `references/question-bank.md`
- **Language.** Converse and write `tasks.md` in the user's language; keep file names, IDs, and keys in English.

## Procedure

### 1. Derive tasks

Walk the acceptance criteria and cut them into tasks. Read the design (data model, interfaces, flows, screens) **when it exists** and use it to sharpen each task's files, boundary, and order. Each task gets: ID (`T1`...), title, outcome in one line, the criteria it satisfies, files or modules it touches, its tests, `depends_on`, a size (S, M, L), and a `Verify:` line (how you'd confirm it). Template in `references/tasks-template.md`. Name the test each task needs; the **layer, seams, and command** for those tests are settled by `test-strategy` in step 4.

### 2. Order and parallelize

Sort by dependency. Note independent tracks that could run in parallel (worktree flow in `AGENTS.md`). A task that blocks everything else goes first and should be small.

### 3. Check traceability

- Every `AC-n` appears in at least one task with a test.
- Every task traces to an `AC-n`. A task that traces only to a design element and no criterion is either a design-gap fix or out of scope — say which.
- Write the coverage table at the bottom of `tasks.md`; list gaps and risks.

### 4. Settle the test strategy

With the tasks cut, invoke `test-strategy` to fill the feature's `## Test strategy` in `tasks.md`: the command, the layers in play, the shared seams, and which task pins which `AC`. It runs here — the last step of the Plan phase, before the loop — so every task's `Tests:` line follows one coherent strategy instead of drifting task by task.

### 5. Handle revisions (delta only)

When `spec.md`'s `revision` is newer than what the tasks were planned from, read the change log at the top of `spec.md`. Then:

- **Check the design first.** If the change log marks a criterion that touches an interface, a data model, or a screen, the design may be stale: send the feature to `system-design` to re-validate the delta (Light tier) before planning anything. A wording-only revision skips this.
- Add tasks only for NEW and CHANGED requirements, tagged with the new revision date.
- If a change supersedes tasks that are not built yet, strike them through and keep the line with the reason (`~~T4~~ dropped: ...`); do not delete.
- If the superseded work is already built, add a task to change it; do not rewrite history.
- `spec.md` keeps its path across revisions; the earlier revision survives only in the change log. Never strip a change-log entry.
- Update the `Spec: specs/NNN-slug/spec.md — Revision <date>` header to the revision you are planning from.
- Added behavior means the strategy may need updating — extend `## Test strategy` with `test-strategy` before the new task's test.

### 6. Handle re-entry from review

Read `review.md`. Add fix tasks for what is missing, strike through tasks that were wrong, leave finished tasks alone. If the problem is the design, send it back to `system-design`. If the problem is the requirement, send it back to `roadmap` (a new spec revision, then gate two again) — never patch the spec here.

### 7. Write and sync

Write `specs/NNN-slug/tasks.md`. Keep checkboxes in sync with reality. If `status.md` exists, update it when you finish (below).

## Gate: approval

Show a concise summary in chat: the task list with sizes and dependencies, parallel tracks, risks, your plan calls, and the `## Test strategy`. Ask for approval in one message. In **phase mode**, show one table over all the phase's features instead of one message per feature, and state which features were excluded (and why) — that is the only difference in the gate.

If the user asks for changes, revise `tasks.md` and ask again.

## Handoff

After approval:

1. If `status.md` exists, set `status: doing` and `next` to the first task. Do not change `cycle` (the loop's controller owns it). Update `notes` with one line.
2. **In the pipeline** (the user said so, `specs/_state.md` exists, or `AGENTS.md` defines it): invoke `development-loop` for the phase's **first ready feature** (its `depends_on` all `done`), and nothing else. The other features' task lists wait for their turn; the loop finishes one feature (and its PR) before the next feature's tasks start, unless the user asks for parallel worktrees.
3. **Standalone, or `development-loop` not installed:** invoke nothing. Say in one line that the loop is the next step and ask whether to start. The approved `tasks.md` is the deliverable.

## Layout

```
specs/
├── _roadmap.md            # phases and features (owned by `roadmap`)
├── _state.md              # current phase and feature (owned by `roadmap`)
└── NNN-slug/
    ├── status.md          # one-feature status (roadmap creates; later skills update)
    ├── spec.md            # requirements and AC-n (roadmap writes; `status: approved`)
    ├── design.md          # shape of THIS feature's solution (system-design; optional)
    ├── tasks.md           # this skill's output
    └── review.md          # review findings (owned by `review`)
docs/
├── product/PRD.md
├── architecture/          # architecture.md + adr/   (the living system design)
└── design/                # agreed UI: screens, flows, states, design system
```

Phases and progress live in `_roadmap.md`, `_state.md`, and each feature's `status.md`.

The agreed **system design** lives in `docs/architecture/architecture.md` (with ADRs) and the agreed **UI** in `docs/design/` — that is what `system-design` writes and what this skill reads. The feature folder's `design.md` is the short, feature-scoped view of that design; it is the first place to look, but the durable design is the one in `docs/`.

## Red Flags

| Thought                                                      | Reality                                                                                                 |
| ------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------- |
| "I'll just plan from the conversation"                       | Plan from the spec. Use the design when it exists.                                                      |
| "The spec is only `status: spec`, close enough"              | Only `status: approved` counts. Send it back to `roadmap`.                                              |
| "No design yet, so I can't plan"                            | The criteria are enough. Plan, and mark the technical calls as plan calls.                             |
| "The design is missing a detail, I'll decide it in the task" | Mark it as a plan call, or send that feature to `system-design` when it is hard to reverse.            |
| "I'll group these features into phases"                      | Phases belong to `roadmap`.                                                                             |
| "I'll plan the phase but guess the missing designs"          | Plan from the criteria and mark plan calls; only escalate a feature whose criteria need a real design call. |
| "I'll add a 'write tests' task at the end"                   | Tests ship with the task that satisfies the criterion.                                                  |
| "I'll make every step a tiny task"                           | One task ≈ one PR. Over-splitting hides the real work.                                                  |
| "The spec changed, I'll re-plan everything"                  | Plan only the new/changed requirements; check the design delta first.                                   |
| "That feature shipped, I'll reopen it"                       | Send it to `roadmap` — a `todo`/`doing` feature gets a spec revision, a `done` one a follow-up feature. |
| "The list looks good, I'll start coding"                     | Planned is not approved. Wait for the gate.                                                             |
| "Approved, so I'll start building"                           | Only in the pipeline. Standalone, offer it in one line and stop.                                        |

## Completion Check

Done when all of these are true:

- [ ] Inputs read; the mode is clear (phase plan, first plan, revision delta, or re-entry).
- [ ] `spec.md` exists with `status: approved` for every planned feature; no task was planned against a `status: spec` draft.
- [ ] The design was read where it exists; every technical call made without one is recorded under **Plan calls**, and any feature sent to `system-design` is named with the reason.
- [ ] Every task traces to an `AC-n`, and every `AC-n` has a task with a test.
- [ ] Tasks are PR-sized, ordered by dependency, parallel tracks noted — including the order **across** features in phase mode.
- [ ] Revisions planned as delta only; superseded tasks struck through, not deleted.
- [ ] `specs/NNN-slug/tasks.md` written for every planned feature, each task tagged with the spec revision date.
- [ ] The user approved the task list (one table for the whole phase in phase mode).
- [ ] `status.md` updated for every planned feature.

If any box is unchecked, keep planning. Do not start the loop.
