---
name: planning
description: "Use after a feature's spec (and design, when there is one) is approved and before any implementation, to turn that one feature into an ordered list of PR-sized tasks in specs/NNN-slug/tasks.md, each traced to acceptance criteria and carrying its own tests. Also use when a spec has a new revision (plan only the new/changed work) or when review sends the feature back (fix the task list, keep what is done). Triggers: plan this feature, break into tasks, task breakdown, spec changed. Not for deciding what to build (`brainstorming`), slicing a project into features and phases (`roadmap`), or choosing the shape of the solution (`system-design`)."
argument-hint: "Optional: the feature folder, e.g. specs/003-upload-invoice. Defaults to current_feature in specs/_state.md."
---

# Planning

Plans **one feature at a time**. `brainstorming` decided what and why, `roadmap` decided which features and in what phases, `system-design` decided the shape of the solution. Planning decides **in what order, in what slices**: a task list a builder can execute without re-reading the conversation.

Workflow position: `brainstorming` → `roadmap` → `system-design` → **`planning`** (with `test-strategy`) → `development-loop` (`implement -> verify -> review`, per unit). It also works standalone on any approved spec; it never depends on the other skills being present.

What this skill does **not** do: group features into phases (that is `roadmap`), make design decisions (that is `system-design`), or write code.

## Step 0: Find the inputs and the mode

Read, in parallel:

- the feature folder: `status.md`, `spec.md` (the current spec — `status: approved` required), `tasks.md`, `review.md` and any `design.md` if present
- `docs/design/` for the agreed UI, `docs/architecture/architecture.md` for the system design and relevant ADRs; `AGENTS.md` (PR size, branch naming, worktree flow); the code the feature touches
- `specs/_state.md` for `current_feature`, when no feature was named

Pick the mode:

| Mode                     | When                                                                     | What you do                             |
| ------------------------ | ------------------------------------------------------------------------ | --------------------------------------- |
| **First plan**           | No `tasks.md`                                                            | Full task list                          |
| **Revision delta**       | `spec.md`'s `revision` is newer than the one the tasks were planned from | Plan only NEW/CHANGED requirements      |
| **Re-entry from review** | `review.md` says the task list was wrong or incomplete                   | Fix the list; never redo finished tasks |

## Preconditions (route back, do not improvise)

- No `specs/NNN-slug/spec.md`, or its frontmatter reads `status: spec` → gate two has not passed. Send it back to `roadmap` (which re-dispatches the spec writer and re-runs its Lavish review). Do not plan against an unapproved spec.
- An approved spec but no testable acceptance criteria → the same route back to `roadmap`.
- Standalone with no roadmap (no `specs/_state.md`): a spec written by `brainstorming` feature mode is acceptable input.
- A design is needed but missing (new interface, data model change, UI) → `system-design`. If the feature records "design: skipped" with a reason, or the design is not needed at all, continue.
- If this project has no design step at all (standalone use), make the smallest technical calls you need, and mark each as a **plan call** in `tasks.md` for the user to check.

For the checks above and for any question worth asking before or during planning, use `references/question-bank.md`.

## Core Rules

- **Plan from the spec and design, not from memory.** Every task traces to an acceptance criterion (`AC-n`) or a design element. If it is in neither, it is not in the plan.
- **No new design decisions.** If `docs/architecture/architecture.md` (or the feature's `design.md`) has a gap, return to `system-design` rather than deciding it silently inside a task.
- **One task ≈ one reviewable PR** (about 300 lines of diff per `AGENTS.md`). Do not over-split. See `references/task-sizing.md`.
- **Tests ship with the task that makes a criterion true.** Do not create a separate "write tests later" task. Every `AC-n` is covered by at least one task with a named test. Planning **names which** test pins each task; the **layer, seams, and command** are settled by `test-strategy` (the `## Test strategy` section of `tasks.md`), which runs as the last step of this phase.
- **Order by dependency:** data model → core logic → interface/API → UI → wiring and polish. Mark tasks that can run in parallel.
- **Never re-plan what already exists.** Unchanged requirements are already built or planned.
- **Never reopen a shipped feature.** If the feature is `done` and shipped, send the change to `roadmap` as a follow-up feature.
- **No code before an approved plan.** Scaffolding and dependency installs count as code.
- **The user decides.** Propose the task list; they confirm or adjust. One question per message when you must ask; otherwise state your call. Questions worth asking, and the layout and ownership rules they lead to, are in `references/question-bank.md`
- **Language.** Converse and write `tasks.md` in the user's language; keep file names, IDs, and keys in English.

## Procedure

### 1. Derive tasks

Walk the acceptance criteria and the design (data model, interfaces, flows, screens) and cut them into tasks. Each task gets: ID (`T1`...), title, outcome in one line, the criteria it satisfies, files or modules it touches, its tests, `depends_on`, a size (S, M, L), and a `Verify:` line (how you'd confirm it). Template in `references/tasks-template.md`. Name the test each task needs; the **layer, seams, and command** for those tests are settled by `test-strategy` in step 4.

### 2. Order and parallelize

Sort by dependency. Note independent tracks that could run in parallel (worktree flow in `AGENTS.md`). A task that blocks everything else goes first and should be small.

### 3. Check traceability

- Every `AC-n` appears in at least one task with a test.
- Every task traces to an `AC-n` or a design element.
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

Show a concise summary in chat: the task list with sizes and dependencies, parallel tracks, risks, your plan calls, and the `## Test strategy`. Ask for approval in one message. For a very small feature (three tasks or fewer) one message is enough. Approval means an explicit yes to the task list you showed.

If the user asks for changes, revise `tasks.md` and ask again.

## Handoff

After approval:

1. If `status.md` exists, set `status: doing` and `next` to the first task. Do not change `cycle` (the loop's controller owns it). Update `notes` with one line.
2. **In the pipeline** (the user said so, `specs/_state.md` exists, or `AGENTS.md` defines it): invoke `development-loop` for the first ready task, and nothing else.
3. **Standalone, or `development-loop` not installed:** invoke nothing. Say in one line that the loop is the next step and ask whether to start. The approved `tasks.md` is the deliverable.

## Layout

```
specs/
├── _roadmap.md            # phases and features (owned by `roadmap`)
├── _state.md              # current phase and feature (owned by `roadmap`)
└── NNN-slug/
    ├── status.md          # one-feature status (roadmap creates; later skills update)
    ├── spec.md            # requirements and AC-n (roadmap writes; `status: approved`)
    ├── design.md          # shape of THIS feature's solution (system-design)
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
| "I'll just plan from the conversation"                       | Plan from the spec and design.                                                                          |
| "The spec is only `status: spec`, close enough"              | Only `status: approved` counts. Send it back to `roadmap`.                                              |
| "The design is missing a detail, I'll decide it in the task" | That is `system-design`'s job. Go back, or mark a plan call if there is no design step.                 |
| "I'll group these features into phases"                      | Phases belong to `roadmap`.                                                                             |
| "I'll add a 'write tests' task at the end"                   | Tests ship with the task that satisfies the criterion.                                                  |
| "I'll make every step a tiny task"                           | One task ≈ one PR. Over-splitting hides the real work.                                                  |
| "The spec changed, I'll re-plan everything"                  | Plan only the new/changed requirements; check the design delta first.                                   |
| "That feature shipped, I'll reopen it"                       | Send it to `roadmap` — a `todo`/`doing` feature gets a spec revision, a `done` one a follow-up feature. |
| "The list looks good, I'll start coding"                     | Planned is not approved. Wait for the gate.                                                             |
| "Approved, so I'll start building"                           | Only in the pipeline. Standalone, offer it in one line and stop.                                        |

## Completion Check

Done when all of these are true:

- [ ] Inputs read; the mode is clear (first plan, revision delta, or re-entry).
- [ ] `spec.md` exists with `status: approved`; no task was planned against a `status: spec` draft.
- [ ] Every task traces to an `AC-n` or a design element, and every `AC-n` has a task with a test.
- [ ] Tasks are PR-sized, ordered by dependency, parallel tracks noted.
- [ ] Revisions planned as delta only; superseded tasks struck through, not deleted.
- [ ] `specs/NNN-slug/tasks.md` written, each task tagged with the spec revision date.
- [ ] The user approved the task list.
- [ ] `status.md` updated when it exists.

If any box is unchecked, keep planning. Do not start the loop.
