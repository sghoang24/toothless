# Roadmap templates

`_roadmap.md`, `_state.md`, and every `status.md` are written entirely in English.

## specs/\_roadmap.md

```markdown
---
status: draft # draft | approved
updated: YYYY-MM-DD
prd: docs/product/PRD.md
---

# Roadmap

## Phase 0: Foundation [status: in-progress] # planned | in-progress | done

Goal: <one sentence>
Exit criteria:

- <verifiable, e.g. "CI is green and an empty build deploys to staging">
  Release gate (end of phase, needs explicit user confirmation):
- [ ] CI green, migrations run, secrets and env vars in place, rollback plan exists
- [ ] Smoke test passes after deploy
      Features:
- 001-scaffold-ci (enabler) — <one line> — covers: NFR-2 — depends_on: none
- 002-auth-oidc (enabler) — <one line> — covers: FR-1, NFR-1 — depends_on: 001

## Phase 1: Walking skeleton [status: planned]

Goal: <one sentence>
Exit criteria:

- <verifiable>
  Release gate:
- [ ] <checks for this phase>
              Features (one line each; detail comes at replan):
- 003-<slug> — <one line> — covers: FR-2 — depends_on: 002

## Phase 2: Core features [status: planned]

Sketch: <short description only; no feature IDs yet if unclear>

## Backlog (unscheduled)

- <idea> — covers: FR-n or none

## Coverage

| Requirement | Covered by    | Note               |
| ----------- | ------------- | ------------------ |
| FR-1        | 002-auth-oidc |                    |
| FR-5        | —             | deferred: <reason> |

## Calls I made

- <decision stated by the assistant, so the user can reject it>

## Phase log

- Phase 0 (YYYY-MM-DD): shipped <...>; cut <...>; learned <...>
```

Phase status values: `planned | in-progress | done`.

## specs/\_state.md

```markdown
current_phase: 0
phase_status: in-progress # planned | in-progress | done
current_feature: 001-scaffold-ci
updated: YYYY-MM-DD
```

Only `roadmap` writes this file. Every other skill only reads it.

## specs/\_status.md (project PR ledger)

Status **has no stage** — `_state.md` already holds the current phase and feature. This file is a
plain ledger of what has landed: one row per merged PR/commit, tagged with its phase, feature, and
task. It is **short — a ledger, not a report.** No evidence, no findings, no "known gaps" (those
belong in the feature's `status.md` notes, `review.md`, or the phase log in `_roadmap.md`).

```markdown
# Project status

Updated: YYYY-MM-DD

| PR  | Phase | Feature              | Task | Note                          |
| --- | ----- | -------------------- | ---- | ----------------------------- |
| #1  | P1    | 001-foundation       | T1   | seeded scaffold + CI          |
| #2  | P1    | 002-manual-portfolio | T2   | add-holding form + service    |
| #3  | P1    | 003-price-alerts     | T3   | Binance ingestion + watchlist |
```

- **Created by `roadmap`** (header + empty table), **appended by `finishing-a-development-branch`**
  when a branch lands, and by `deploy` when a phase ships.
- One row per landing. If a feature spans several PRs, it gets several rows; if several tasks land
  in one PR, list the tasks in the Note.
- **Do not record stage, cycle, the current feature, or a narrative here** — `_state.md` holds the
  position. If you are tempted to add a paragraph, it goes in the phase log (`_roadmap.md`) or the
  feature's `status.md` instead.

## specs/NNN-slug/status.md

```markdown
feature: NNN-slug
phase: 0
covers: [FR-1, NFR-1]
depends_on: []
status: todo # todo | doing | blocked | done
current_unit: null # e.g. T3, or T4-T6 for a batch — what the loop is building
cycle: 0 # times THIS unit was sent back (only the loop increments it)
cycle_cap: 3 # per unit (a task, or a batch); a blocked unit is parked and the loop moves on
next: design # one-line pointer, e.g. "design", "implement T3", "fix review blocker B1"
notes: |
<short, newest first>
```

Created by `roadmap` (`status: todo`, `cycle: 0`, `next: design`). Updated by `system-design`, `planning`, and `development-loop`.

## Who writes what

| File          | Created by | Updated by                                                                                                                                                       |
| ------------- | ---------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `_roadmap.md` | `roadmap`  | `roadmap` (replan); the user approves                                                                                                                            |
| `_state.md`   | `roadmap`  | `roadmap`                                                                                                                                                        |
| `_status.md`  | `roadmap`  | `finishing-a-development-branch` (each landing), `deploy` (phase ship)                                                                                           |
| `status.md`   | `roadmap`  | `system-design`, `planning`, `development-loop` (`status`, `current_unit`, `next`, `notes`); only the loop increments `cycle`, and only when a unit is sent back |

## Status meanings

- `todo`: not started.
- `doing`: started. The step is not recorded; infer it from the feature folder (`spec.md` → spec done, `design.md` → design done, ticked boxes in `tasks.md` → implementation progress, `review.md` with open blockers → in review).
- `blocked`: stopped, needs the user (for example a task parked past `cycle_cap`, or an unmet dependency). The loop keeps going with the next task when only one is stuck.
- `done`: merged and reviewed. Deploy happens at the phase release gate.

## Choosing the next feature

The first feature in the current phase, in roadmap order, whose `depends_on` features are all `status: done`. If none is ready and the phase is not finished, set `blocked` on the feature that is holding things up and ask the user.

## Release gate

When every feature in the phase is `done`, `roadmap` runs replan: verify the exit criteria, go through the release gate checklist with the user, and only then mark the phase `done` and advance. Never deploy or tick the gate without explicit confirmation.

## Sizing check for a feature

Split it if any of these is true:

- it needs more than about 8 acceptance criteria;
- it spans several unrelated user-visible behaviors;
- its plan would touch most modules of the project;
- you cannot describe what the user can do after it ships in one sentence.

Merge it with a neighbor if it delivers nothing observable on its own and is not an enabler.
