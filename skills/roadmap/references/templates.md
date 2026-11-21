# Roadmap templates

`_roadmap.md`, `_state.md`, `spec.md`, and every `status.md` are written entirely in English.

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
                      Features:
- 003-<slug> — <one line> — covers: FR-2 — depends_on: 002
- 004-<slug> — <one line> — covers: FR-3 — depends_on: 003

## Phase 4: Expansion [status: planned]

Goal: <one sentence>
Exit criteria:

- <verifiable>
      Features: # cut now — their specs are written in step 6; only the prose detail is lazy
- 0NN-<slug> — <one line> — covers: FR-9, NFR-4 — depends_on: 00M

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

**Every phase lists its features with IDs, `covers`, and `depends_on`** — the spec writer needs them,
and step 6 specs every phase in one run. "Phase detail is lazy" limits **prose**, not feature rows:
the current phase may carry extra notes, later phases carry the rows only.

A phase whose features genuinely cannot be named yet (the PRD is vague about it) is written as:

```markdown
## Phase 3: Hardening [status: planned]

Goal: <one sentence>. Features to be cut at replan before this phase's specs are written.
```

That phase gets no specs in step 6; replan cuts its features and dispatches its spec writer. Do not
guess feature IDs to fill the gap — a made-up feature produces a made-up spec.

## specs/NNN-slug/spec.md

One per feature, for **every** feature of **every** phase. Written by the spec-writer subagent
([spec-writer-prompt.md](./spec-writer-prompt.md)), checked and approved by `roadmap`.

```markdown
---
feature: NNN-slug
phase: 0
status: spec # spec | approved — `roadmap` sets `approved` after gate two
revision: YYYY-MM-DD
covers: [FR-1, NFR-1] # verbatim from _roadmap.md
depends_on: [] # verbatim from _roadmap.md
---

# <feature name> — spec

Revision: YYYY-MM-DD (initial) # later revisions: "(supersedes YYYY-MM-DD)" + change log

## Intent and why

<2-4 sentences: what the user can do after this ships, and why the phase needs it.>

## Covers

<One line per requirement ID, quoting the PRD's own wording.>

## Goals / Non-goals

## Acceptance criteria

- **AC-1**: Given <state>, when <action>, then <observable result>.
- **AC-2**: ...

## AC → requirement mapping

| AC   | Requirement | Phase exit criterion (if it serves one) |
| ---- | ----------- | --------------------------------------- |
| AC-1 | FR-1        | —                                       |

## Open questions

<None, or only what could not be answered from the PRD, roadmap, or code — with the assumed answer.>

## Left to the builder

<Local choices inside the tasks. Never schema, API shape, or a boundary: those are `system-design`.>
```

Rules for a spec:

- The frontmatter `feature`, `phase`, `covers`, and `depends_on` are **copied from `_roadmap.md`,
  verbatim**. If they disagree, the roadmap wins and the spec is wrong.
- Every `AC-n` traces to a `FR-n`/`NFR-n` or a phase exit criterion, and every one appears in the
  mapping table. An unmapped AC is a defect.
- At most ~8 acceptance criteria. More means the feature should be split — report it, do not split it.
- `status: spec` means written but not approved; `planning` refuses to run against it. `system-design`
  and `planning` only trust `status: approved`.
- A revision is a new `Revision:` line plus a change log at the top; the body stays a complete,
  stand-alone document (never a diff).

## A spec revision (Revise mode)

There is still **one** spec file. `roadmap` bumps the `Revision:` line, sets `status: spec`, and
re-runs gate two. Routing rules: see [change-requests.md](./change-requests.md).

```markdown
---
feature: NNN-slug
phase: 0
status: spec # unapproved again until gate two passes
revision: 2026-11-27
covers: [FR-1, NFR-1]
depends_on: []
---

# <feature name> — spec

Revision: 2026-11-27 (supersedes 2026-11-20)
Changes since 2026-11-20:

- NEW: AC-6 — <new behaviour>
- CHANGED: AC-3 — <what changed, and how>
- UNCHANGED: AC-1, AC-2, AC-4, AC-5 (by name only)
```

Revision rules:

- **AC IDs are stable.** A changed criterion keeps its ID; a new one takes the next free number; a
  retired one stays as `AC-n (removed)` — never renumber, because `tasks.md` and the tests refer to them.
- **The change log is the audit trail.** The body below it is always the full, current requirements.
- **Never create a second dated spec file.** History is the `Revision:` line, the change log, and git
  (`git log -p specs/NNN-slug/spec.md`).
- **`history/` snapshots are opt-in and normally off** — only for a project with no usable version
  control. When enabled, copy the outgoing revision to
  `specs/NNN-slug/history/YYYY-MM-DD-spec.md` **before** rewriting. No skill reads `history/`.
- A `done`/shipped feature is **not** revised: `replan` cuts a follow-up feature (`00N-slug-2`).

## specs/\_state.md

```markdown
current_phase: 0
phase_status: in-progress # planned | in-progress | done
current_feature: 001-scaffold-ci
updated: YYYY-MM-DD
```

`roadmap` creates it (create / revise / replan). One other skill updates it, and only one thing: when
`closing-a-phase` closes a phase and advances the cursor, it rewrites `current_phase`,
`phase_status`, and `current_feature`. No other skill writes this file.

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

- **Created by `roadmap`** (header + empty table), **appended by `closing-a-phase`**
  when a branch lands, and by `deploy` when a phase ships.
- One row per landing. If a feature spans several PRs, it gets several rows; if several tasks land
  in one PR, list the tasks in the Note.
- **Do not record stage, cycle, the current feature, or a narrative here** — `_state.md` holds the
  position. If you are tempted to add a paragraph, it goes in the phase log (`_roadmap.md`) or the
  feature's `status.md` instead.

## specs/NNN-slug/status.md

A **cursor**, not a record. It answers "where is this feature now" — never "what is this feature".
`feature`, `phase`, `covers`, and `depends_on` live in `_roadmap.md` only; duplicating them here was what
made this file look like a second spec. Six fields, keep them short:

```yaml
status: todo # todo | doing | blocked | done
current_unit: null # e.g. T3, or T4-T6 for a batch — what the loop is building
cycle: 0 # times THIS unit was sent back (only the loop increments it)
cycle_cap: 3 # per unit (a task, or a batch); a blocked unit is parked and the loop moves on
next: design # one-line pointer, e.g. "design", "implement T3", "fix review blocker B1"
notes: |
<short, newest first>
```

Created by `roadmap` at Create mode step 6c (or when a phase advances) for the features of the
**current phase**. Updated by `system-design`, `planning`, and `development-loop`.

**Missing is recoverable, not fatal.** Every field is either derived from `_roadmap.md` or a constant,
so any consumer that needs it can create it from the feature's roadmap row without ambiguity —
see _Ensure, don't assume_ below.

## Who writes what

| File          | Created by                                                                                 | Updated by                                                                                                                                                       |
| ------------- | ------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `_roadmap.md` | `roadmap`                                                                                  | `roadmap` (replan / revise); the user approves                                                                                                                   |
| `_state.md`   | `roadmap`                                                                                  | `roadmap`, and `closing-a-phase` when it closes a phase                                                                                                          |
| `_status.md`  | `roadmap`                                                                                  | `closing-a-phase` (each landing), `deploy` (phase ship)                                                                                                          |
| `spec.md`     | `roadmap` — via the **spec-writer subagent**, one per phase; gate-two approval by the user | `roadmap` (revision: delta only, then gate two again)                                                                                                            |
| `status.md`   | `roadmap`; a consumer may create a missing one (see _Ensure, don't assume_)                | `system-design`, `planning`, `development-loop` (`status`, `current_unit`, `next`, `notes`); only the loop increments `cycle`, and only when a unit is sent back |

## Status meanings

- `todo`: not started.
- `doing`: started. The step is not recorded; infer it from the feature folder (`spec.md` `status: approved` → spec done, `design.md` → design done, ticked boxes in `tasks.md` → implementation progress, `review.md` with open blockers → in review).
- `blocked`: stopped, needs the user (for example a task parked past `cycle_cap`, or an unmet dependency). The loop keeps going with the next task when only one is stuck.
- `done`: merged and reviewed. Deploy happens at the phase release gate.

## Choosing the next feature

The first feature in the current phase, in **roadmap order**, whose `depends_on` features are all
`status: done`. Read the order and the dependencies from `_roadmap.md`; read the `status` field from each
feature's `status.md`. If none is ready and the phase is not finished, set `blocked` on the feature that
is holding things up and ask the user.

## Ensure, don't assume

`status.md` is the one artifact whose absence is not an error to stop on. Before a skill uses a
feature's `status.md`, it ensures one exists: create it from the feature's `_roadmap.md` row
(`status: todo`, `current_unit: null`, `cycle: 0`, `cycle_cap: 3`, `next: design`) and **say it was
missing** — a silent create hides a roadmap run that stopped early.

| Skill                   | Ensures for                                      |
| ----------------------- | ------------------------------------------------ |
| `planning` (phase mode) | every feature of the phase, before cutting tasks |
| `development-loop`      | the feature it is about to build                 |
| `closing-a-phase`       | every feature of the phase it just opened        |

Creating one is always safe: the content is derived, never invented, so two skills creating the same
file produce the same file.

## Release gate

When every feature in the phase is `done`, the phase is closed by
**`closing-a-phase`** — the skill that lands the last feature's branch already holds the
evidence, so it is the only place a phase becomes `done`. It verifies the exit criteria against that
evidence, walks this checklist with the human, records the Phase log, flips the phase status, creates
the next phase's `status.md` files, and updates `_state.md`. Never deploy or tick the gate without
explicit confirmation; never close a phase from `roadmap`, `deploy`, or `fight`.

## Sizing check for a feature

Split it if any of these is true:

- it needs more than about 8 acceptance criteria;
- it spans several unrelated user-visible behaviors;
- its plan would touch most modules of the project;
- you cannot describe what the user can do after it ships in one sentence.

Merge it with a neighbor if it delivers nothing observable on its own and is not an enabler.
