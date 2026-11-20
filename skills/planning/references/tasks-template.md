# Tasks Template

`specs/NNN-slug/tasks.md` is this skill's output — the ordered, PR-sized slices for
**one feature**. It lives in the feature folder. The roadmap (`specs/_roadmap.md`)
owns features and phases; this file owns the tasks inside one feature.

## `specs/NNN-slug/tasks.md`

```markdown
# Tasks — <feature name> (NNN-slug)

Spec: specs/NNN-slug/spec.md — Revision <YYYY-MM-DD> (`status: approved`)
Design: specs/NNN-slug/design.md → docs/architecture/architecture.md (+ docs/design/ when there's a UI) — omit this line when there is no design
Status: planned

## Tasks

- [ ] **T1** (rev 2026-11-17) — <task title>. Outcome: <one reviewable change>.
      Criteria: AC-1. Touches: <files/modules>. Tests: <the test(s) that pin it>.
      depends_on: none. Size: S.
      Verify: <how you'd confirm it>.
- [ ] **T2** (rev 2026-11-17) — <task title>. Outcome: <...>.
      Criteria: AC-2. Touches: <...>. Tests: <...>.
      depends_on: T1. Size: M.
      Verify: <...>.

## Test strategy (set once per feature by `test-strategy`, in the Plan phase)

- Command: <the project's declared test/check command, per `AGENTS.md`>
- Layers: <e.g. domain logic unit; repositories + route handlers integration; no e2e here>
- Shared seams: <what is faked (clock, network) vs. run real (in-memory/test DB), decided once>
- Per task: <which task pins which AC at which layer, e.g. T1→AC-1 unit, T2→AC-2 integration>

## Parallel tracks

- <T-track A: tasks with no shared files / ordering that can run in worktrees>
- <T-track B: ...>

## Coverage

| Acceptance criterion | Task(s) | Test        |
| -------------------- | ------- | ----------- |
| AC-1                 | T1      | <test name> |
| AC-2                 | T2      | <test name> |

## Gaps / risks

- <AC-n not yet covered, or a risk that could change the plan>

## Plan calls (mark for the user to check)

- <technical call made because there was no design step, if any>
```

## Revision delta (when the spec gets a new revision)

Add a section for the new revision's **NEW/CHANGED** requirements only. Unchanged
requirements get no new tasks.

```markdown
## Revision 2026-12-01 — delta

- [ ] **T7** (rev 2026-12-01) — <task for a NEW/CHANGED requirement>. Outcome: <...>.
      Criteria: AC-5. Tests: <...>. depends_on: T2. Size: S. Verify: <...>.
- [ ] ~~T5~~ (rev 2026-11-17) — dropped: superseded by rev 2026-12-01.
```

## Rules

- **One task ≈ one reviewable PR** (~300 lines of diff, per `AGENTS.md`). Not a
  sub-step, not an epic. See `task-sizing.md`.
- Every task names its **outcome** (what's true when done) and its **Verify:**
  line (how you'd confirm it) — a task missing either one isn't sized.
- Every task traces to an **`AC-n`**; every `AC-n` has at
  least one task with a **named test**. The **`## Test strategy`** section (set once
  per phase by `test-strategy`, in the Plan phase) names the command, the layers in
  play, and the shared seams each task's test follows — see `test-strategy`.
- The design, when one exists, sharpens a task's files and boundary but never justifies
  a task on its own. A technical call made without a design is listed under **Plan calls**.
- Task IDs (`T1`...) are stable within the feature; once assigned, don't renumber.
- Each task carries its **spec revision date** and its **`depends_on`**.
- **Superseded tasks are struck through, not deleted** (`~~T5~~ dropped: ...`).
- **`docs/architecture/architecture.md` owns the design**; never restate it here —
  link to it. A short `design.md` in the feature folder may carry the shape of this
  feature's solution when one exists.
