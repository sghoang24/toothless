---
name: finishing-a-development-branch
description: "You MUST use this when a task's branch is built, reviewed clean, and verified green. The goal is to raise the MR/PR for the human to review, or merge it only on explicit approval, then clean up the worktree. You are not allowed to merge on your own initiative (raising an MR is the default, merging is opt-in; silence is not approval). Trigger phrases: finish the branch, raise an MR, open a PR, merge this, land the branch."
argument-hint: "Point at the finished branch, e.g. feat/001-auth, or the task it closes."
---

# Finishing a Development Branch

The `development-loop` built, verified, and reviewed the change; this skill is what happens **after** all of that: the branch is done, and it has to land. Landing is the one step in the workflow that is **hard to reverse and visible to others**, so it has its own rules.

The `development-loop` ends here; this skill is the exit. It is **self-contained** — use it whenever a branch is ready to land, whether or not `fight` is driving.

## The Model

| Thing        | What it is                                                                     | Where it lives                          |
| ------------ | ------------------------------------------------------------------------------ | --------------------------------------- |
| **Branch**   | One feature = one branch = one PR.                                             | `feat/NNN-slug`                         |
| **MR/PR**    | The proposal for the human to review and merge. The **default**.               | the remote host (GitHub / GitLab / ...) |
| **Merge**    | Making the branch part of the target branch. Reversible only by another merge. | the host, or locally                    |
| **Approval** | The human's explicit "merge it". The **only** thing that unlocks a merge.      | the conversation                        |
| **Worktree** | The isolated workspace, if one was used. Removed once landed.                  | `<parent>/<repo>-<slug>/`               |

## When to Use

- A task's branch is implemented, reviewed clean, and verified green.
- The user asks to finish, raise an MR/PR, land, or merge the branch.

**Do NOT use** to close a branch that hasn't passed review and verification — that's the loop's job.
Do NOT use to merge by default: raising an MR is the default, merging is opt-in.

## Core Rules

- **Raising an MR is the default; merging is not.** When a branch is done, you
  **raise the MR/PR** for the human to review and merge. You do not merge on your own.
- **Merge requires explicit human approval.** Treat it exactly like `deploy` (`AGENTS.md`):
  the human says "merge it" and you do; silence is not approval.
  "Looks good" on the design or the diff is **not** approval to merge.
- **Green is the precondition, not the claim.** Before raising or merging, confirm the branch was
  **freshly verified** (the loop's `verify` gate) and reviewed clean (`review`) — and, when the project's
  CI runs the project's declared test command (`AGENTS.md`) on the merge candidate, that it's green there too.
- **Merge through the MR, not by pushing.** Merging an approved MR into `main`/`master` (via the host, e.g. `gh pr merge`) is allowed on explicit approval. What is **never** allowed is a local merge followed by `git push` to a protected branch, or any direct push / force-push to `main`/`master` (`AGENTS.md`) — that bypasses review and rewrites shared history. If in doubt, use the MR's merge.
- **One branch, one MR.** Per `AGENTS.md`, one feature = one branch = one PR. If the branch carries
  more than its task, stop — that's a scope defect back to `implement`.
- **Clean up after landing.** Once the branch is merged (or explicitly discarded), remove its
  worktree (`git-worktrees`) so the workspace doesn't drift.
- **Always tell the human where it stands.** After raising an MR, report the link and the state
  (green, awaiting review). After merging, say it merged, on whose approval, and what's next.

## Procedure

### 1. Confirm the branch is actually done

Check the exit conditions before anything lands: the task's `Verify:` line is met, `review`
found nothing Critical/Important, and `verify` is green on a **fresh** run. If the project has
CI (the project's declared test command, `AGENTS.md`, on the merge candidate), confirm it's green. If any is not true,
stop — it's not ready; go back to the loop.

### 2. Raise the MR/PR (the default)

Create the MR/PR from `feat/NNN-slug` into the target branch (`AGENTS.md` PR title format:
`[toothless] <Title>`; body summarizes the task, the spec, and the verification evidence).
Report the link and state to the human, and **wait**. This is the normal, expected ending — the
human reviews and merges.

### 3. Merge (only on explicit approval)

If — and only if — the human explicitly approves the merge, merge **through the MR** into the
target branch (e.g. `gh pr merge`). Never merge locally and then `git push` the protected branch:
the MR path is what keeps review and history intact. Never infer approval from a passing review, a
materialized design, or a general "looks good". If the human says it can be merged automatically,
that instruction is the approval: merge, then report. If there is any doubt, raise the MR and ask.

### 4. Clean up

Once the branch has landed (merged or explicitly discarded), remove its worktree and prune
(`git-worktrees`): `git worktree remove <path>` then `git worktree prune`. Never force-remove a
worktree with uncommitted work — stop and ask; it may be the human's in-progress work.

### 5. Record the landing and report

Append one row to the project ledger `specs/_status.md` (template in the `roadmap` skill): the PR
number, its **phase**, **feature**, and **task**, and a one-line note. Then clear the feature's
`cycle` in `specs/NNN-slug/status.md` and set `next`. No narrative here — the ledger is a table,
not a report. Tell the human what landed, on what approval, and what the next task or phase is.

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

## Completion Check

A branch is finished correctly when ALL are true:

- [ ] The branch passed `review` (fresh session) and a fresh `verify` — plus CI, if the
      project gates on it.
- [ ] An MR/PR was raised for the human to review — **or** the human explicitly approved the
      merge, and only then did it merge.
- [ ] No push to `main`/`master`, no force-push; the merge (if any) went **through the MR**, not a local push.
- [ ] One branch = one PR; no out-of-scope changes riding along.
- [ ] The worktree (if any) was removed and pruned after landing — with no force while
      uncommitted work existed.
- [ ] One row appended to `specs/_status.md` (PR · phase · feature · task · note); the feature's
      `cycle` cleared; the human told what landed and what's next.

If the branch merged without explicit approval, that's a gate violation — re-read the Core Rules.
