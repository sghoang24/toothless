# The `review` Step

`implement` proves the change runs green; `review` judges it against intent. It is the loop's quality gate, and the controller runs it **per unit** (a task, or a batch) in a **fresh session**. It also works standalone on any completed change.

The reviewer **finds and reports; it never fixes.** Fixes go back to the author so the author stays accountable for the change.

## Inputs

- The diff: `BASE..HEAD` (`HEAD` from `git rev-parse HEAD`; `BASE` recorded **before** the implementer was dispatched). For a **batch**, this diff covers every member — review the whole of it as one unit.
- The unit's brief and the implementer's report (the claims under review, to be treated as unverified). For a batch, the brief lists each member's file and change — check that the diff matches the list, no more and no less.
- The current spec (`specs/NNN-slug/spec.md`, `status: approved`; its `Revision:` line tells you which revision you are reviewing against), the feature's `design.md`, and the relevant parts of `docs/architecture/architecture.md`.

Hand the reviewer these as crafted context. Never hand it the author's session history.

## When to Use

- A task is implemented and its evidence is verified, and it has not been reviewed yet.
- Before merging a branch, opening a PR, or moving to the next task.

**Do NOT use** in the session that wrote the code. If you wrote it, stop — hand the review to a fresh session. Do not use as a substitute for `implement`'s verify gate: review judges the change against intent, verify proves it runs green. And do not let it sprawl — it is **bounded, diff-scoped work with a stop rule**, not an open-ended audit.

## Core Rules

A review is bounded work with a defined end. A review that keeps growing is broken.

- **Fresh session, not the author's.** If you wrote the code, stop and hand the review to a fresh reviewer subagent.
- **Read-only.** Inspect with `git show`, `git diff`, `git log`. Never mutate the working tree, index, or HEAD. If another revision must run, use a throwaway `git worktree` and remove it afterward.
- **The diff is the subject.** Read each changed file once. Read a caller, guard, or test only when a specific finding needs it. If judging a finding needs code far outside the diff, mark it "needs author context" instead of exploring.
- **No test runs on this checkout.** The implementer's report carries the test evidence; judge its honesty by reading and raise a missing case as a finding. Run a test only when reading raises a doubt no existing run answers, and then a focused test — never the whole suite. At most one targeted mutation, in a throwaway worktree.
- **No recursive dispatch.** One reviewer, one session. A reviewer that spawns a sub-reviewer duplicates a seat the controller already fills.
- **Oversized diff is the finding.** If a single task's diff is over about 300 lines (`AGENTS.md`) or does more than the task, send it back to split. A **batch** is judged as a whole against the same ~300-line bar, not member by member. Do not spend a review on an oversized unit.
- **Never pre-judge a finding.** The controller must not tell the reviewer to ignore an issue, treat it as Minor, or accept "the plan chose this." If a prompt contains "do not flag," that is pre-judging — the finding must be raised and adjudicated in the loop, not suppressed. **A batch is not an exception:** a batched diff gets the same scrutiny, only fewer dispatches.
- **Stop rule.** Stop when every changed file is read once, the three axes are walked, the tests are read, and each surviving finding has a severity and file:line. Then write the report and end.

## The Three Axes

A review answers three questions — does the change do what the requirement says, does it fit the architecture, and is the code clean. A line-by-line read that only asks "do the tests pass?" checked none of them.

| Axis             | The question                            | Look for                                                                                                   |
| ---------------- | --------------------------------------- | ---------------------------------------------------------------------------------------------------------- |
| **Requirements** | Does it do what the task/spec says?     | Spec match, edge cases (null/empty/boundary), error paths, off-by-one, race, state drift, unintended scope |
| **Architecture** | Does it fit the system?                 | New patterns, wrong layer, duplication, coupling, abstraction level                                        |
| **Clean code**   | Could a fresh engineer read it unaided? | Unclear names, tangled control flow, cleverness, unexplained intent                                        |

Keep it proportional to the change: a one-line fix earns a glance at each axis, not a paragraph.

**Out of scope here.** Security and deep performance work are **not** this pass. If a change touches untrusted input, auth, secrets, or personal data, that is `security-and-hardening`'s job — note it and hand off, do not audit it inline. The reviewer stays in its lane so it stays fast.

## Procedure

1. **Check size first.** Oversized or out of scope: stop and send it back (see Core Rules).
2. **Read the claims, not the trust.** The brief says what was requested; the report says what the implementer believes it built. Both are claims. Judge the diff on its merits — a stated rationale ("kept it simple per YAGNI") never downgrades a finding's severity. For a batch, also check **uniformity**: every member's change should be the same shape. One member that quietly did something different is a finding — a batch is the one place a scope slip hides well.
3. **Tests first, then the diff.** Tests reveal intent: do they test behavior rather than mocks, cover the edge cases, and would they catch a regression? Then walk the diff against the three axes and confirm it satisfies the task's criteria and nothing more.
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

Write the report to `specs/NNN-slug/review.md` (or reply with it if standalone), ordered by leverage, not discovery order:

```markdown
**Verdict:** Ready | Not ready | Ready with fixes — <one or two sentences>

### Strengths

<one to three specific items with file:line>

### Findings

**Critical**

1. <title> — <file:line>: <what is wrong>; <why it matters>; <the move, if structural>
   **Important** ...
   **Minor** ...

### Declined to judge

<behaviors set aside as out of scope, one line each with the reason; "nothing" if none>

### Recommendations

<non-blocking improvements, if any>
```

## Route and record

The controller routes the outcome; the reviewer's job is to name where the problem lives.

| The problem is in                      | Send to                                                              |
| -------------------------------------- | -------------------------------------------------------------------- |
| the code                               | `implement`                                                          |
| the task list (wrong or missing tasks) | `planning`                                                           |
| the design                             | `system-design`                                                      |
| the requirement                        | `roadmap` — a spec revision (delta only), then its Lavish gate again |
| the PRD itself                         | `brainstorming` (project mode)                                       |

- **Sending back:** the controller increments `cycle` in `specs/NNN-slug/status.md` (only a send-back increments it), sets `next` to the return target, and adds a one-line note. At `cycle_cap`, park with a ruling and ask the human.
- **Clean or minor-only:** the unit is done (`development-loop` step 3.7). The **branch** lands once per feature, after the final review — not per unit.
- **Re-review** after fixes is a fresh, bounded review of the new diff, with the same limits — it cannot wander onto untouched code.

## Red Flags

| Thought                                                          | Reality                                                             |
| ---------------------------------------------------------------- | ------------------------------------------------------------------- |
| "I wrote it, I'll review my own diff"                            | Fresh session. Your blind spots are the point.                      |
| "The report says it's clean, so it's clean"                      | The report is a claim. Judge the diff.                              |
| "Tests pass, so it's fine"                                       | Tests cover behavior, not conventions, boundaries, or scope.        |
| "The extra cleanup is nice, I'll allow it"                       | Unintended scope is a finding.                                      |
| "Let me read everything that calls this" / "just one more thing" | Bounded. Read what a finding needs, then apply the stop rule.       |
| "I'll edit the file and restore it, or run the suite"            | Read-only. Mutate only in a throwaway worktree, once.               |
| "I'll dispatch a sub-reviewer to double-check"                   | No recursive dispatch.                                              |
| "No findings, so I'll add a few nits"                            | Zero findings is valid. Inventing findings is dishonest.            |
| "The spec doesn't mention this case"                             | Judge silent behavior by what a reasonable user expects.            |
| "The plan chose this, so it's fine"                              | The plan does not grade its own work. Report it; the human decides. |

## Completion Check

- [ ] Ran in a fresh session, read-only, with no further dispatch.
- [ ] Diff-scoped: each changed file read once, ended on the stop rule; no suite run on this checkout.
- [ ] Oversized or out-of-scope diff was sent back, not reviewed.
- [ ] Checked against the task's criteria, the current spec, and the design; all three axes walked; tests read first.
- [ ] The report and its rationales were treated as claims, not evidence; no finding was pre-judged.
- [ ] Findings substantiated, each with severity, file:line, and impact; report ordered by leverage.
- [ ] `review.md` written; the controller updated `status.md` (`next`, `notes`, and `cycle` only on a send-back).

If a Critical or Important issue is open, the task does not proceed.
