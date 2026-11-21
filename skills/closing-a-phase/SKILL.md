---
name: closing-a-phase
description: "You MUST use this when a phase is finishing or you ask to close one — 'finish this phase', 'complete phase X', 'close the phase', 'is the phase done', 'next phase'. Closing a phase means: land the phase's last feature's branch (raise the MR/PR for the human; merge only on explicit approval; clean up the worktree), verify the phase's exit criteria with evidence, walk the release gate for an explicit human yes, flip the phase status, and open the next phase. You are not allowed to merge on your own initiative (raising an MR is the default, merging is opt-in; silence is not approval), or to close a phase without the human's explicit yes at its release gate. Trigger phrases: finish this phase, complete phase X, close the phase, phase done, is the phase done, next phase, finish the branch, raise an MR, open a PR, merge this, land the branch."
argument-hint: "Point at the phase being closed, e.g. phase 2, or the branch that closes it — feat/001-auth."
---

# Closing a Phase

A phase ends when its **last feature lands** — and closing it is more than landing one branch. This skill is the exit from a phase: it lands that final branch, then verifies the phase against its exit criteria, takes the human's release-gate approval, and opens the next phase. `roadmap`, `deploy`, and `fight` never close a phase; this skill is the only place it happens.

It is also **self-contained**: use it whenever a phase is finishing, whether or not `fight` is driving. If you were only asked to land a single branch that is **not** the phase's last feature, you still start here and stop after step 5 — the phase steps simply don't apply.

## The Model

| Thing            | What it is                                                                      | Where it lives                          |
| ---------------- | ------------------------------------------------------------------------------- | --------------------------------------- |
| **Phase**        | A group of features that ships together. Closes only here.                      | `specs/_roadmap.md`                     |
| **Branch**       | One feature = one branch = one PR. The phase's last feature is its last branch. | `feat/NNN-slug`                         |
| **MR/PR**        | The proposal for the human to review and merge. The **default** landing.        | the remote host (GitHub / GitLab / ...) |
| **Merge**        | Making the branch part of the target branch. Reversible only by another merge.  | the host, or locally                    |
| **Approval**     | The human's explicit "merge it" / "close it". The **only** unlock.              | the conversation                        |
| **Release gate** | The human's explicit yes to close the phase. A green e2e is not it.             | the conversation                        |
| **Worktree**     | The isolated workspace, if one was used. Removed once landed.                   | `<parent>/<repo>-<slug>/`               |

## When to Use

- The current phase's features are all `done` and you want to close the phase.
- The user asks to "finish this phase", "complete phase X", "close the phase", "is the phase done", or "open the next phase".
- The phase's last feature's branch is implemented, reviewed clean, and verified green and needs to land.

**Do NOT use** to close a phase whose features aren't all `done` — the phase continues; land the work and stop at step 5.
**Do NOT use** to merge by default: raising an MR is the default, merging is opt-in.
**Do NOT use** to close a branch that hasn't passed review and verification — that's the loop's job.

## Core Rules

- **Closing a phase is the job; landing the last branch is how it starts.** The branch steps exist so the
  phase can close — not the other way around.
- **Raising an MR is the default; merging is not.** When the branch is done, **raise the MR/PR** for the
  human to review and merge. You do not merge on your own.
- **Merge requires explicit human approval.** Treat it exactly like `deploy` (`AGENTS.md`): the human
  says "merge it" and you do; silence is not approval. "Looks good" on the design or the diff is **not**
  approval to merge.
- **Green is the precondition, not the claim.** Before raising or merging, confirm the branch was
  **freshly verified** (the loop's `verify` gate) and reviewed clean (`review`) — and, when the project's
  CI runs the project's declared test command (`AGENTS.md`) on the merge candidate, that it's green there too.
- **Merge through the MR, not by pushing.** Merging an approved MR into `main`/`master` (via the host,
  e.g. `gh pr merge`) is allowed on explicit approval. What is **never** allowed is a local merge followed
  by `git push` to a protected branch, or any direct push / force-push to `main`/`master` (`AGENTS.md`) —
  that bypasses review and rewrites shared history. If in doubt, use the MR's merge.
- **One branch, one MR.** Per `AGENTS.md`, one feature = one branch = one PR. If the branch carries more
  than its task, stop — that's a scope defect back to `implement`.
- **Clean up after landing.** Once the branch is merged (or explicitly discarded), remove its worktree
  (`git-worktrees`) so the workspace doesn't drift.
- **Always tell the human where it stands.** After raising an MR, report the link and the state (green,
  awaiting review). After merging, say it merged, on whose approval, and what's next.
- **A phase closes here, or nowhere.** When the last feature of a phase lands, this skill verifies the
  exit criteria against evidence, walks the release gate with the human, flips the phase status, and
  creates the next phase's `status.md` files. It never reorders features or touches `covers` /
  `depends_on` — those stay `roadmap`'s. And it never closes a phase on a green e2e alone: that gate
  needs an explicit yes.

## Procedure

### 1. Confirm the last branch is actually done

Check the exit conditions before anything lands: the task's `Verify:` line is met, `review` found
nothing Critical/Important, and `verify` is green on a **fresh** run. If the project has CI (the
project's declared test command, `AGENTS.md`, on the merge candidate), confirm it's green. If any is
not true, stop — it's not ready; go back to the loop.

### 2. Raise the MR/PR (the default)

Create the MR/PR from `feat/NNN-slug` into the target branch (`AGENTS.md` PR title format:
`[toothless] <Title>`; body summarizes the task, the spec, and the verification evidence). Report the
link and state to the human, and **wait**. This is the normal, expected ending — the human reviews and merges.

### 3. Merge (only on explicit approval)

If — and only if — the human explicitly approves the merge, merge **through the MR** into the target
branch (e.g. `gh pr merge`). Never merge locally and then `git push` the protected branch: the MR path
is what keeps review and history intact. Never infer approval from a passing review, a materialized
design, or a general "looks good". If the human says it can be merged automatically, that instruction
is the approval: merge, then report. If there is any doubt, raise the MR and ask.

### 4. Clean up

Once the branch has landed (merged or explicitly discarded), remove its worktree and prune
(`git-worktrees`): `git worktree remove <path>` then `git worktree prune`. Never force-remove a
worktree with uncommitted work — stop and ask; it may be the human's in-progress work.

### 5. Record the landing and report

Append one row to the project ledger `specs/_status.md` (template in the `roadmap` skill): the PR
number, its **phase**, **feature**, and **task**, and a one-line note. Then clear the feature's
`cycle` in `specs/NNN-slug/status.md` and set `next`. No narrative here — the ledger is a table, not
a report. Tell the human what landed, on what approval, and what the next task or phase is.

**If this branch was not the phase's last feature, stop here** — the phase continues.

### 6. Close the phase

This is the skill's namesake and the only place a phase becomes `done`.

**Check `specs/_roadmap.md`: is every feature of the current phase `done`?** Read the `status` field
from each feature's `status.md`, not from the roadmap. If any is not `done`, stop: the phase continues.

If they all are:

1. **Verify the exit criteria with evidence** — the phase's tests/demo, the milestone `e2e-testing`
   result, and a smoke result if this phase deploys. Item by item. If you cannot produce evidence for
   one, say which and ask; never tick it because the features merged.
2. **Walk the release gate with the human** and get an explicit yes. Closing a phase is a gate exactly
   like merge and deploy: a green e2e is not approval, and silence is not approval.
3. **Record the outcome** in the Phase log of `_roadmap.md` — what shipped, what was cut or moved,
   what you learned. Do not touch feature rows, `covers`, or `depends_on`: the plan belongs to `roadmap`.
4. **Flip the status** in `_roadmap.md`: the finished phase `done`, the next one `in-progress`.
   **This skill is the only place a phase becomes `done`** — not `roadmap`, not `deploy`, not `fight`.
5. **Create `specs/NNN-slug/status.md`** for every feature of the new phase (`status: todo`,
   `current_unit: null`, `cycle: 0`, `cycle_cap: 3`, `next: design` — template in the `roadmap` skill).
   Their `spec.md` already exists; only the cursor is new. Say which ones you created.
6. **Re-validate the new phase's specs** against `_roadmap.md` and the current PRD. Drifting specs go
   back to `roadmap` (a delta revision, then its gate two) — do not edit a `spec.md` here.
7. **Update `_state.md`** (`current_phase`, `phase_status`, `current_feature`) and hand off to
   `system-design` for the first ready feature. Standalone: name that step in one line and stop.

## Red Flags

| Thought                                              | Reality                                                                                    |
| ---------------------------------------------------- | ------------------------------------------------------------------------------------------ |
| "Review passed, so I'll just merge it"               | Clean review ≠ approval to merge. Raise the MR; wait for the human.                        |
| "The design was approved, so merging is fine"        | Design approval is not merge approval. Separate gates.                                     |
| "I'll merge to save the human a step"                | Merging is not your call. Raise the MR by default.                                         |
| "They didn't say don't merge"                        | Silence is not approval. Only an explicit "merge it" unlocks a merge.                      |
| "It's green locally, that's enough to merge"         | If CI gates the merge candidate, green means green in CI too.                              |
| "I'll force-push to fix the branch up"               | Never force-push. Merge through the MR; never `git push` a protected branch (`AGENTS.md`). |
| "I'll leave the worktree, someone may need it"       | Remove it once the branch lands; leave the tree clean (`git-worktrees`).                   |
| "This branch is a bit big but it's basically one PR" | One branch = one PR. Oversized means two tasks — split it.                                 |
| "All the features merged, so the phase is done"      | Merged features are not met exit criteria. Verify each one with evidence, and ask.         |
| "The e2e was green, that's the gate"                 | Green e2e is one input. The release gate needs an explicit yes from the human.             |
| "I'll close the phase and let roadmap fix the plan"  | Close the phase, never the plan. Feature rows and `covers` stay `roadmap`'s.               |

## Completion Check

A phase is closed correctly when ALL are true:

- [ ] The phase's last branch passed `review` (fresh session) and a fresh `verify` — plus CI, if the
      project gates on it.
- [ ] An MR/PR was raised for the human to review — **or** the human explicitly approved the merge,
      and only then did it merge.
- [ ] No push to `main`/`master`, no force-push; the merge (if any) went **through the MR**, not a local push.
- [ ] One branch = one PR; no out-of-scope changes riding along.
- [ ] The worktree (if any) was removed and pruned after landing — with no force while uncommitted
      work existed.
- [ ] One row appended to `specs/_status.md` (PR · phase · feature · task · note); the feature's
      `cycle` cleared; the human told what landed and what's next.
- [ ] Exit criteria verified with evidence (not assumed), the release gate confirmed **explicitly** by
      the human, the Phase log written, the finished phase `done` and the next `in-progress`, the next
      phase's `status.md` files created (and named), and `_state.md` updated. No feature row, `covers`,
      or `depends_on` was edited here.

If the branch merged without explicit approval, that's a gate violation — re-read the Core Rules.
