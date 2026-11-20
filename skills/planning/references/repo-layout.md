# Repository Layout & Tracking

How requirements, specs, designs, features, tasks, and phases relate. Each fact
lives in exactly one place.

## The model

| Thing                     | What it is                                                                                                                 | Where it lives                                                |
| ------------------------- | -------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------- |
| **Requirement**           | A product-level need with an ID (`FR-n`, `NFR-n`).                                                                         | `docs/product/PRD.md`                                         |
| **Feature**               | The unit of work: a vertical slice with observable behavior. Covers one or more requirements.                              | named in `specs/_roadmap.md`; folder `specs/NNN-slug/`        |
| **Spec**                  | One feature's agreed requirements, with testable acceptance criteria (`AC-n`).                                             | `specs/NNN-slug/spec.md`                                      |
| **Design**                | The shape of the solution for one feature: data model, interfaces, flows, screens. May be recorded as "skipped: <reason>". | `specs/NNN-slug/design.md`                                    |
| **Task**                  | One reviewable PR (≈300 lines). Belongs to a feature.                                                                      | `specs/NNN-slug/tasks.md`                                     |
| **Review**                | Findings for one feature, with the return path.                                                                            | `specs/NNN-slug/review.md`                                    |
| **Phase**                 | A milestone grouping features that together deliver something runnable.                                                    | `specs/_roadmap.md`                                           |
| **System-wide decisions** | Architecture and ADRs shared by many features. A living document that grows with the features.                             | `docs/architecture/architecture.md`, `docs/architecture/adr/` |

Three mappings, never mixed:

- **Requirement → feature:** each feature lists the `FR-n` / `NFR-n` it `covers`.
  `roadmap` checks coverage.
- **Feature → phase:** `roadmap` groups features into phases. Phase membership and
  order live in `specs/_roadmap.md`, not in the feature folder.
- **Feature → tasks:** `planning` turns one feature's acceptance criteria and
  design into tasks in the feature's `tasks.md`. The criteria are the required input;
  the design is an enrichment. A feature with no design is still plannable — the
  technical calls it forces are recorded as **plan calls**.

## The current spec and its revisions

The **current spec** of a feature is always `specs/NNN-slug/spec.md` — one file per
feature, for every feature of every phase, written by the `roadmap` spec writer.
A revision keeps the same path: it bumps the `Revision:` line and adds a change-log
entry at the top, and the earlier revision survives in that log.

Each revision is a **complete, stand-alone document** of the latest requirements
(not a diff). Its frontmatter carries `status: spec | approved` and `revision`; the
body opens with a change log:

```markdown
# <feature> — spec

Revision: 2026-12-01 (supersedes 2026-11-12)
Changes since 2026-11-12:

- NEW: <requirement added since last revision>
- CHANGED: <requirement that changed, and how>
- UNCHANGED: <the rest, by name only>
```

`status: approved` is set only after the Roadmap phase's second Lavish gate. A
`status: spec` draft is not an approved spec: `system-design` and `planning` refuse it
and send the feature back to `roadmap`.

A new revision is new work, but only its delta gets planned:

- `planning` reads the change log and schedules **only the NEW/CHANGED**
  requirements as new tasks.
- **UNCHANGED requirements are not re-planned.** They are already built or planned.
- Superseded tasks are struck through in `tasks.md`, not deleted.
- If the feature is already `done` and shipped, do not reopen it. `roadmap`
  replans and adds the change as a follow-up feature, in the current or a later
  phase.

## Why these files sit where they do

- **`specs/`** holds the working artifacts: the roadmap, the cursor, and one folder
  per feature with its spec, design, tasks, review, and status.
- **`specs/_roadmap.md`** is the only place phases and the feature list live.
- **`specs/_state.md`** is the cursor (current phase and feature). `roadmap` owns
  it; every other skill only reads it.
- **`specs/NNN-slug/status.md`** is the only place one feature's own progress lives.
- **`docs/`** holds system-level reference that changes rarely: the PRD,
  `architecture.md`, and ADRs. Per-feature design does **not** live here; it lives
  in the feature folder. Shared UI foundations (navigation shell, design tokens)
  are built by the first feature that needs them and recorded in `architecture.md`
  or an ADR.

## One roadmap file, not one per phase

`specs/_roadmap.md` holds every phase and the overall feature list. A single file
is easier to scan and keeps cross-phase dependencies visible. Phase prose is
**lazy**: only the current phase is spelled out in detail. Feature **rows** exist
for every phase (`ID`, one line, `covers`, `depends_on`) because their specs are
written from them in the Roadmap phase.

## Linking, not duplicating

- `tasks.md` holds the task text, its spec `revision` date, and its checkbox status.
  It links to `design.md` and never restates it.
- `_roadmap.md` references features by ID and shows phase, order, `covers`, and
  `depends_on`. It carries no task detail.

When two files disagree, the owner wins:

| Question                                      | Owner                               |
| --------------------------------------------- | ----------------------------------- |
| What does the product need?                   | PRD                                 |
| What is "correct" for this feature?           | `spec.md` (its acceptance criteria) |
| Which phase, and in what order?               | the roadmap                         |
| What shape does this feature's solution take? | `design.md`                         |
| What was decided for the whole system?        | `architecture.md` and ADRs          |

A feature design must not contradict `architecture.md` or an ADR. If it has to,
record a new ADR first, then update `architecture.md`.

## Status tracking

- **`_roadmap.md`** carries each phase's goal, verifiable exit criteria, release
  gate checklist, and status (`planned | in-progress | done`).
- **`_state.md`** is the cursor: `current_phase`, `phase_status`, `current_feature`.
- **`specs/NNN-slug/status.md`** is one feature's status: `todo | doing | blocked |
done`, its `current_unit`, `cycle` and `cycle_cap`, and a one-line `next` pointer. The step
  inside a feature is not recorded; infer it from the files in the folder. Only
  `development-loop` increments `cycle`, and only when a unit is sent back (the cap is per unit — a task, or a batch of small same-shape tasks).

See the `roadmap` skill's `templates.md` for the exact formats.

## Alternative organizations (and when to prefer them)

The layout above is the default. Pick differently when:

- **Standalone use (no roadmap):** keep just `specs/NNN-slug/` with `spec.md` and
  `tasks.md`. There is no `status.md` or `_state.md`, so `planning` skips status
  updates and offers the next step in one line. A spec written by `brainstorming`
  (feature mode) is acceptable input there.
- **Single-feature project:** `_roadmap.md` still holds the plan, but it is one
  phase with one feature; skip the ceremony and review in prose.
- **Flat backlog:** if the work is not milestone-shaped, keep a `specs/_backlog.md`
  ordered by priority instead of numbered phases.
- **Kanban-style:** if the team tracks in an external tool, keep `tasks.md` as IDs
  plus status and let the tool own board state; do not duplicate columns here.
