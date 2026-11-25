# The Final Review (Roll-Up)

Per-unit reviews already judged each diff. The final review does **not** re-review the branch — it
**rolls their verdicts up** and checks the few things only exist at feature scale: coverage,
contradictions, and anything no unit owned.

It is a **read of the unit reviews and the gate tables**, not a fresh diff pass. That is deliberate: a
second full review is the most expensive thing in the loop and the least likely to find something a
unit review missed, because it reads the same code against the same context pack.

Run it once, after the last unit is done, in a fresh session.

## When to Run

- Every unit in `tasks.md` is done or parked-with-ruling.
- Before handing to `closing-a-phase`.

## Inputs (and nothing else)

- Every `unit-<N>-review.md` (see `review.md` → _Severity and report_).
- The gate table from each unit's report, or the rolled-up table in `status.md`.
- The feature's `review-context.md` — for the acceptance criteria, not for re-reading the code.

Do **not** generate the whole-branch diff. Do not open the changed files unless one specific
contradiction needs the line it points at.

## What the Roll-Up Checks

| Check              | Question                                                                                | What a failure looks like                                          |
| ------------------ | --------------------------------------------------------------------------------------- | ------------------------------------------------------------------ |
| **Coverage**       | Is every acceptance criterion in the context pack answered by at least one unit review? | A criterion no unit owned — the classic miss                       |
| **Contradictions** | Do two unit reviews disagree about the same code or interface?                          | Unit 3 says the helper is extracted, unit 5 says it was duplicated |
| **Parked debt**    | Is every `open` gate row and every parked finding accounted for?                        | An `open` row nobody fixed and nobody ruled on                     |
| **Verdicts**       | Does the worst unit verdict still hold?                                                 | A unit marked Ready whose finding was later reopened               |
| **Consistency**    | Did two units solve the same problem two ways?                                          | Two error-handling styles, two names for one concept               |

If all five hold, the roll-up passes. That is the normal case, and it should take minutes, not a
session.

## Procedure

1. **Collect the inputs.** List the unit review files; if one is missing for a done unit, that is a
   finding — the unit is not reviewable after the fact. Say so.
2. **Walk the criteria.** For each acceptance criterion, name the unit review that covers it. A gap goes
   in the report; do not go read the code to fill it.
3. **Diff the verdicts.** Look for contradictions, duplicated solutions, and unresolved `open` rows.
   When two reviews conflict, the line they cite is the one thing worth opening.
4. **Write the roll-up** to `specs/NNN-slug/review.md` (the feature-level file, distinct from the
   per-unit ones).
5. **A single fix wave.** Collect every finding, fix in one pass — not a unit-by-unit re-entry. If the
   roll-up turns up more than a handful, the problem is structural: route it and say so.
6. **Route and hand off**, exactly as a unit review does (code → `implement`, task list → `planning`,
   design → `system-design`, requirement → `roadmap`, PRD → `brainstorming`). When clean (or
   Minor-only), hand to `closing-a-phase`.

## The one exception: a seam needs eyes

A roll-up reads verdicts, so it can miss a seam that **no unit review could see** — two units each
correct, contradictory together. Close reading catches most of these because the reviews cite lines.

If a specific seam is genuinely unclear from the reviews (they never overlapped that area), that is the
**only** time to open the branch: read just those two units' diffs, answer the seam question, and stop.
Not a full pass — a targeted look at one contradiction. Say you did it and why.

## Report

Write it to `specs/NNN-slug/review.md`, short — this is a roll-up, not a third review:

```markdown
**Verdict:** Ready | Ready with fixes | Not ready — <against the feature, one or two sentences>

### Coverage

<criterion → which unit review answered it; list the gaps>

### Conflicts

<none, or the contradiction, both citations, and which line settles it>

### Carried debt

<every `open` row and parked finding, with its ruling; "none" if none>
```

- `Ready` — every criterion covered, no unresolved conflict; hand to landing.
- `Ready with fixes` — fixable in one wave; do so, then land.
- `Not ready` — a structural gap; route it and say so plainly.

## Red Flags

| Thought                                                 | Reality                                                                  |
| ------------------------------------------------------- | ------------------------------------------------------------------------ |
| "I'll just re-review the whole branch to be safe"       | That is the expensive pass this skill replaced. Roll up the units.       |
| "Every unit passed, so the roll-up can't find anything" | Coverage and contradictions are exactly what per-unit review cannot see. |
| "I'll open the files to double-check the review"        | Only for one contradiction the reviews cannot settle.                    |
| "It found thirty things, so we loop unit by unit again" | A single fix wave. Thirty at this stage means a structural gap.          |
| "I'll re-run the suite to confirm"                      | The gate tables carry the evidence. Do not re-run the loop.              |

## Completion Check

- [ ] Ran once, after the last unit, from the unit reviews and context pack — not a fresh diff pass.
- [ ] Every acceptance criterion traced to a unit review; gaps named.
- [ ] Contradictions, duplicated solutions, and unresolved `open` rows checked.
- [ ] Any code opened was only to settle one specific seam, and that was said.
- [ ] Findings fixed in a single wave or parked-with-ruling; `review.md` written; handed to `closing-a-phase` only when clean.
