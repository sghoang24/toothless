# The `review` Step

`implement` proves the change runs green; `review` judges it against intent. It is the loop's quality gate, and the controller runs it **per unit** (a task, or a batch) in a **fresh session**. It also works standalone on any completed change.

The reviewer **finds and reports; it never fixes.** Fixes go back to the author so the author stays accountable for the change.

## Inputs

- The diff `BASE..HEAD` (`BASE` recorded before the implementer was dispatched; never `HEAD~1`).
- The unit's brief and report — the claims under review, treated as unverified.
- The feature's **review context pack** (`review-context.md`, see `brief-and-report.md`), not the four separate documents.

Crafted context only — never the author's session history.

## When to Use

- A unit is implemented and its gate is filled, and it has not been reviewed yet.
- Before merging a branch, opening a PR, or moving to the next unit.

**Do NOT use** in the session that wrote the code, as a substitute for the gate (the gate proves it runs green; review judges intent), or as an open-ended audit.

## Right-size the review (read this first)

A review's cost is mostly fixed — the fresh session, the context, the report — so a five-line diff otherwise pays the same as a two-hundred-line one. **Depth is a dial here too**, exactly as in `system-design`'s design tiers. Pick the tier from the diff before reading it, and say which you chose.

| Tier         | Choose when                                                                                          | The review does                                                                                                                     |
| ------------ | ---------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| **Light**    | ≤ ~50 lines, 1–2 files, **no contract change** (no interface, schema, API, config key, or migration) | Diff read once, the four gate rows confirmed or flipped at a glance, **three-line report**: verdict + gate + findings.              |
| **Standard** | Anything that changes a contract, exceeds ~50 lines, or more than 2 files                            | The full pass below, full report shape.                                                                                             |
| **Deep**     | Auth, money, data migration or loss, secrets, concurrency                                            | Standard, plus the failure paths walked explicitly — and note that `security-and-hardening` owns the security audit, not this pass. |

Rules of thumb:

- **No contract change and ≤50 lines is Light** — if the diff adds, removes, or reshapes anything another unit will depend on, it is Standard whatever its size.
- **A batch is judged as a whole.** Six small edits can exceed 50 lines together; judge the batch's total, not each member.
- **When unsure, take the higher tier** and say why — an under-reviewed contract change costs a cycle.
- **Light is not a shortcut past the gate.** Critical/Important still block, and the stop rule still ends it. It only drops the ceremony around a diff that has nothing to hold.

## Core Rules

A review is bounded work with a defined end. A review that keeps growing is broken.

- **Fresh session, not the author's.** If you wrote the code, stop and hand the review to a fresh reviewer subagent.
- **Read-only.** Inspect with `git show`, `git diff`, `git log`. Never mutate the working tree, index, or HEAD. If another revision must run, use a throwaway `git worktree` and remove it afterward.
- **The diff is the subject.** Read each changed file once; read a caller or guard only when a specific finding needs it. Needs code far outside the diff? Mark it "needs author context" and move on.
- **No test runs on this checkout.** The report carries the test evidence; judge its honesty by reading. One focused test or one targeted mutation (throwaway worktree) only when a specific doubt needs it — never the whole suite.
- **No recursive dispatch.** One reviewer, one session.
- **Oversized diff is the finding.** Over ~300 lines, or more than the task → send back to split, do not review it. A batch is judged as a whole against the same bar.
- **Never pre-judge a finding.** No "do not flag" in a prompt. A batch gets the same scrutiny, only fewer dispatches.
- **Stop rule.** Stop when every changed file is read once, all four rows are confirmed or flipped, the tests are read, and each surviving finding has a severity and file:line. Then write the report and end.

## The Four Lens

The implementer filled a gate table with four rows. Your job is to **confirm or flip each row**, with evidence — not to run a second, unrelated review. A line-by-line read that only asks "do the tests pass?" checked none of them.

| Lens         | The question                                                | Look for                                                                                                 |
| ------------ | ----------------------------------------------------------- | -------------------------------------------------------------------------------------------------------- |
| **code**     | Does it do what the task says, in the right layer, cleanly? | Spec match, edge cases, error paths, off-by-one, unintended scope, naming, control flow                  |
| **security** | Does this diff open a hole?                                 | Untrusted input unvalidated, missing authz, a secret in code/logs, unsafe deserialization, wildcard CORS |
| **ui**       | Do the screens match `docs/design/`?                        | Drift from the mock, missing state, wrong tokens/components, layout broken at the edges                  |
| **ux**       | Do flows, states, and messages hold up?                     | Empty/loading/error states, dead ends, unclear copy, destructive action without confirm                  |

Keep it proportional: a one-line fix earns a glance at each lens, not a paragraph.

**Confirming a row:** the evidence holds up → write `confirm` and move on; do not restate it.

**Flipping a row:** mark it `flip` with severity and file:line. Flip an `n/a` when the diff contradicts it — a trust boundary touched under `n/a: no trust boundary` is a finding, and it is the gap a hidden `n/a` exists to cover.

**Security is the diff, not the audit.** Scan only what this unit changed. Threat models, dependencies, and privacy are `security-and-hardening`'s job at a milestone — name it in one line and move on.

**One verdict, the worst row.** There are not four verdicts, and a green code lens does not dilute a Critical security row. The controller routes on that single verdict and increments one cycle.

## Procedure

1. **Check size first.** Oversized or out of scope: stop and send it back (see Core Rules). Then set the tier (Light / Standard / Deep) and say which.
2. **Read the claims, not the trust.** The brief says what was requested; the report says what the implementer believes it built. Both are claims. Judge the diff on its merits — a stated rationale ("kept it simple per YAGNI") never downgrades a finding's severity. For a batch, also check **uniformity**: every member's change should be the same shape. One member that quietly did something different is a finding — a batch is the one place a scope slip hides well.
3. **Tests first, then the diff.** Tests reveal intent: do they test behavior rather than mocks, cover the edge cases, and would they catch a regression? Then walk the diff and confirm or flip each of the four gate rows, checking it satisfies the task's criteria and nothing more.
4. **Substantiate, then report.**
   - Check each candidate against the specific caller, guard, or test it needs; drop it if that rules it out.
   - Separate a defect already in the base from a regression this change introduces.
   - For behavior the spec is silent on, judge by what a reasonable user would expect; silence is not permission.
   - A structural finding names the move in one line (split the file, extract the helper); it does not design the fix.
   - Zero findings is a valid result. Never invent one to look thorough.

## Severity and report

- **Critical:** breaks correctness, data, or the spec. Blocks merge.
- **Important:** a real defect or a broken convention. Fix before proceeding.
- **Minor:** style, naming, small improvements, non-blocking. Note, do not block.

Write the report to `<scratch>/specs-NNN-slug/unit-<N>-review.md` (the same scratch dir as the brief and report; or reply with it if standalone), ordered by leverage, not discovery order. **One file per unit** — a later unit must never overwrite an earlier unit's verdict, because the final review reads them all.

```markdown
**Verdict:** Ready | Not ready | Ready with fixes — <one or two sentences; the worst lens>

### Gate

| Lens     | Status          | Evidence / reason                   |
| -------- | --------------- | ----------------------------------- |
| code     | confirm \| flip | <file:line, or the flipped finding> |
| security | confirm \| flip | ...                                 |
| ui       | confirm \| flip | ...                                 |
| ux       | confirm \| flip | ...                                 |

### Findings

**Critical**

1. <title> — <file:line>: <what is wrong>; <why it matters>; <the move, if structural>
   **Important** ...
   **Minor** ...

### Declined to judge

<behaviors set aside as out of scope, one line each with the reason; "nothing" if none>
```

The Gate table is the report's core — a reader should see all four lens statuses without reading the findings. Strengths and Recommendations are **optional**: include a Strength only when it explains why a risk was accepted, and drop the section otherwise.

**Light tier** drops the template to three lines — the tier exists so a small diff doesn't pay Standard's price:

```markdown
**Verdict:** Ready | Not ready — <one sentence; the worst lens>
**Gate:** code <confirm|flip> · security <confirm|flip|n/a> · ui <confirm|flip|n/a> · ux <confirm|flip|n/a>
**Findings:** <none, or "Minor — <file:line>: <what>" per line; Critical/Important use the full form>
```

## Route and record

The controller routes; the reviewer only names where the problem lives: code → `implement`, task list → `planning`, design → `system-design`, requirement → `roadmap` (delta revision, then gate two), PRD → `brainstorming`.

- **Sending back:** increment `cycle` (only a send-back increments it), set `next`, add a one-line note. At `cycle_cap`, park with a ruling and ask the human.
- **Clean or minor-only:** the unit is done. The branch lands once per feature, after the final review.
- **Re-review** after fixes is a fresh, bounded review of the new diff — it cannot wander onto untouched code.

## Red Flags

| Thought                                                          | Reality                                                             |
| ---------------------------------------------------------------- | ------------------------------------------------------------------- |
| "I wrote it, I'll review my own diff"                            | Fresh session. Your blind spots are the point.                      |
| "The report says it's clean, so it's clean"                      | The report is a claim. Judge the diff.                              |
| "Tests pass, so it's fine"                                       | Tests cover the code lens. Three rows still need an answer.         |
| "ui/ux don't apply here, so I'll skip the rows"                  | Confirm `n/a` or flip it with a reason. Skipping hides a gap.       |
| "The extra cleanup is nice, I'll allow it"                       | Unintended scope is a finding.                                      |
| "Let me read everything that calls this" / "just one more thing" | Bounded. Read what a finding needs, then apply the stop rule.       |
| "I'll edit the file and restore it, or run the suite"            | Read-only. Mutate only in a throwaway worktree, once.               |
| "I'll dispatch a sub-reviewer to double-check"                   | No recursive dispatch.                                              |
| "No findings, so I'll add a few nits"                            | Zero findings is valid. Inventing findings is dishonest.            |
| "The spec doesn't mention this case"                             | Judge silent behavior by what a reasonable user expects.            |
| "The plan chose this, so it's fine"                              | The plan does not grade its own work. Report it; the human decides. |
| "A 20-line diff needs the full report to be safe"                | Depth is a dial. Light still blocks on Critical/Important.          |
| "It's small, so I'll skip picking a tier"                        | State the tier. Undeclared depth is how reviews drift.              |

## Completion Check

- [ ] Ran in a fresh session, read-only, with no further dispatch.
- [ ] The tier was chosen and stated; a Light diff got the three-line report, not the full template.
- [ ] The reviewer read the feature's context pack, not the separate spec/design/architecture files.
- [ ] Diff-scoped: each changed file read once, ended on the stop rule; no suite run on this checkout.
- [ ] Oversized or out-of-scope diff was sent back, not reviewed.
- [ ] Checked against the task's criteria, the current spec, and the design; all four gate rows confirmed or flipped with evidence; tests read first.
- [ ] The report and its rationales were treated as claims, not evidence; no finding was pre-judged.
- [ ] Findings substantiated, each with severity, file:line, and impact; report ordered by leverage.
- [ ] `unit-<N>-review.md` written; the controller updated `status.md` (`next`, `notes`, and `cycle` only on a send-back).

If a Critical or Important issue is open, the task does not proceed.
