# The Final Review (Whole-Branch)

Per-unit reviews judge each diff in isolation, against its own unit. The **final review** runs once, after the last unit is done, and judges the branch as a whole. It catches what no per-unit review can: the seams between units, the feature against its spec, and the defects that only appear once the pieces sit together.

Run it in a fresh session, like any review. The reviewer reads the **whole branch diff** (the merge base to `HEAD`), not the last unit's diff.

## When to Run

- Every unit in `tasks.md` is done or parked-with-ruling.
- Before handing to `finishing-a-development-branch`.

Not before. A final review over a half-built feature reviews a moving target and re-runs after every unit.

## What It Adds

| Per-unit review                 | Final review                                  |
| ------------------------------- | --------------------------------------------- |
| One unit's diff vs its unit.    | The whole branch vs the feature spec.         |
| Is this change correct?         | Is this feature coherent?                     |
| Local seams: this diff's edges. | Global seams: between units, and to the base. |
| The unit's acceptance criteria. | The feature's stated outcome, end to end.     |

Look specifically for:

- **Seams between units.** Two units each correct, but inconsistent together: duplicated helpers, two ways of doing the same thing, a name that drifted, an interface bent to fit a later unit.
- **A batch that drifted.** A batched run should be uniform. If the members ended up in several shapes — one file renamed properly, another with the old name kept as an alias — that is the seam a per-batch review was too close to see.
- **The feature against its spec.** Every acceptance criterion in the newest spec revision, walked once against the branch as a whole. A criterion no single unit owned is the classic miss.
- **Accumulated drift.** N honest local decisions that sum to a shape nobody chose: a layer that got fat, a pattern used three ways, dead code from a superseded unit.
- **The base's problems.** A defect already in the base is not this branch's regression — name it, but do not block on it. Separate the two.
- **Cross-cutting gaps.** Migration + model + API + docs: did the feature's story get told in all the places it touches? Anything the per-unit view was too narrow to see.

## Procedure

1. **Check the branch size first.** The whole branch may legitimately exceed the per-unit ~300-line rule — a feature is bigger than a unit. Judge the _units_, not the branch, against that rule; a branch that is one giant unit is still the finding.
2. **Generate the whole-branch diff** — from the branch's merge base with the target branch (`git merge-base <target> HEAD`) to `HEAD`.
3. **Dispatch one fresh reviewer** with `references/reviewer-prompt.md`, adapted: the subject is the whole branch, `BASE` is the merge base, and the specification is the feature's `spec.md` + `design.md`, not a single unit.
4. **Walk the axes at feature scale.** Requirements → every acceptance criterion, once, end to end. Architecture → the feature's shape, not one diff's. Clean code → consistency across the whole change.
5. **A single fix wave.** Collect all findings, then fix them in **one** pass — not a unit-by-unit re-entry. A final review that turns into another loop has found a structural problem, not a list of fixes.
6. **Route the outcome**, exactly as a per-unit review does: code → `implement`; task list → `planning`; design → `system-design`; requirement → `system-design` or `brainstorming`.
7. **Hand to `finishing-a-development-branch`** when the final review is clean (or Minor-only).

## Report

Write it to `specs/NNN-slug/review-final.md`, same shape as a unit review (`references/review.md` → _Severity and report_), with the verdict stated against the **feature**, not a unit:

- `Ready` — the feature is coherent and complete; hand to landing.
- `Ready with fixes` — the findings can be fixed in one wave; do so, then land.
- `Not ready` — a structural problem; route it and say so plainly.

## Red Flags

| Thought                                                     | Reality                                                                     |
| ----------------------------------------------------------- | --------------------------------------------------------------------------- |
| "Every unit was reviewed, the final review is redundant"    | Unit reviews are diff-scoped. Seams and spec-vs-feature need the branch.    |
| "I'll run it after each unit to be safe"                    | It judges the whole; run it once, at the end.                               |
| "It found thirty things, so we loop unit by unit again"     | A single fix wave. Thirty findings at this stage means a structural gap.    |
| "A base defect blocks this branch"                          | Name it, don't block on it. It is not this branch's regression.             |
| "The diff is 800 lines, so it fails the size rule"          | A feature is bigger than a unit. Judge the units, not the branch.           |
| "Spec criterion 4 wasn't in any unit, so it's out of scope" | A criterion no unit owned is exactly what the final review exists to catch. |
| "The batch looked uniform close up"                         | Per-batch review is close-up. Drift between members shows here.             |

## Completion Check

- [ ] Ran once, after the last unit, in a fresh session.
- [ ] Judged the **whole branch** (merge base → `HEAD`), not the last diff.
- [ ] Every acceptance criterion in the newest spec walked against the branch.
- [ ] Seams, batch drift, accumulated drift, and cross-cutting gaps checked.
- [ ] Findings fixed in a single wave, or parked-with-ruling and the human told.
- [ ] `review-final.md` written; the branch handed to `finishing-a-development-branch` only when clean.
