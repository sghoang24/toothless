---
name: dispatching-parallel-agents
description: "You MUST use this to fan out independent tasks to several agents at once instead of one after another. The goal is to run the safe-to-parallelize tasks concurrently - one worktree + branch + crafted context each - and merge results one at a time. You are not allowed to fan out tasks that share files, interfaces, or ordering (when in doubt, run in sequence), and you must own and verify the result, not trust an agent's claim. Trigger phrases: parallel agents, fan out, run tasks concurrently."
argument-hint: "Point at the independent tasks, e.g. specs/002-search T1-T4, or 'P1 tasks 1-3'."
---

# Dispatching Parallel Agents

One agent working tasks in sequence is safe but slow. Several agents working at once is fast but collides — same files, same working tree, same index. This skill is the safe way to fan out: **independent tasks, one isolated workspace each, results merged one at a time.**

This skill is **self-contained** — use it whenever you (or a coordinator) are running several independent tasks at once, whether or not the `fight` workflow is in play. `fight` is one caller: when it's active, it coordinates and calls this skill; when it isn't, you apply the same pattern directly. It never applies to a single task — that's the loop's `implement` in place.

## The Model

| Thing            | What it is                                                        | Where it lives                                          |
| ---------------- | ----------------------------------------------------------------- | ------------------------------------------------------- |
| **Coordinator**  | The session that dispatches, tracks, and merges. Owns the result. | whoever is coordinating (e.g. `fight` when active) |
| **Agent**        | One task in its own worktree + branch + context.                  | `git-worktrees`                                         |
| **Task**         | One reviewable PR (≈300 lines). The unit that fans out.           | `specs/NNN-slug/tasks.md`                               |
| **Independence** | No shared files, no dependency, no ordering between two tasks.    | the precondition for fan-out                            |

## When to Use

- Two or more tasks are ready and **independent** (no shared files, no shared interface, no ordering).
- The user asked to speed things up, or the roadmap/task list marks parallel tracks.

**Do NOT use** when tasks share files or interfaces, when one depends on another's output, or for a single task. When in doubt, run in sequence — a merge conflict costs more than the parallelism saves.

## Core Rules

- **Independence is the precondition, not a nice-to-have.** No shared files, no shared interface, no ordering. If a dependency exists, dispatch only the independent ones; the rest wait.
- **One worktree and branch per agent.** They cannot share a working tree — the index collides. Use `git-worktrees`; one `feat/NNN-slug` branch each.
- **Crafted context, never your own history.** Each agent gets its task, the spec, the design pointers, and its branch — not the whole session.
- **You own the result.** An agent's "success" is a claim; check the diff and verify. Never report an agent's result as fact unverified.
- **Merge is sequential.** Work in parallel; land one PR at a time. Resolve each branch to green before starting the next merge.
- **Respect the loop.** Each agent's task still goes through the loop's `verify` and `review` — parallelism changes the schedule, not the gates.
- **Cap the fan-out.** Per `AGENTS.md`, keep concurrency small (its config suggests up to 3). More agents on the same codebase means more collisions, not more speed.

## Procedure

### 1. Confirm independence

For each candidate task, check: does it touch files another candidate touches? does it depend on another's output? does it need to land before another? If any answer is yes, it isn't a candidate — sequence it. State which tasks fanned out and which waited, and why.

### 2. Give each agent its workspace

Create one worktree + branch per task (`git-worktrees`, `feat/NNN-slug` per `AGENTS.md`). Confirm each has a clean baseline before work starts.

### 3. Prepare crafted context per agent

For each agent, assemble: the task and its `Verify:` line, the newest spec revision, the relevant design sections, its worktree path and branch, and the conventions from `AGENTS.md`. Keep it to what the task needs — never your own session history.

### 4. Dispatch

Launch the agents (each running the loop's `implement` for its task). Cap concurrency per `AGENTS.md`. While they run, keep handling the user's messages and other unblocked work — don't idle waiting.

### 5. Collect and check

As agents finish, treat each report as a claim: read the diff, run the task's verification, confirm it meets its `Verify:` line. Reject or send back anything that doesn't.

### 6. Review and verify each

Each task goes through the loop's `verify` and `review` (fresh session), exactly as a serial task would. Parallelism doesn't lower the bar.

### 7. Merge sequentially

Land one branch at a time, resolving to green before the next — even if the work was built in parallel. Sequential merge is what keeps parallel work safe. Clean up each worktree when its branch is done.

## Red Flags

| Thought                                            | Reality                                                                        |
| -------------------------------------------------- | ------------------------------------------------------------------------------ |
| "They're probably independent, let's go"           | Check for shared files and ordering first. "Probably" is how conflicts happen. |
| "They can share one working tree"                  | The index collides. One worktree and branch per agent.                         |
| "More agents means faster"                         | Past the concurrency cap, collisions cost more than the speed gains.           |
| "The agent said it succeeded"                      | A report is a claim. Check the diff and verify it.                             |
| "Merge them all at once, it'll be fine"            | Land one at a time, each green, or you'll debug a tangled merge.               |
| "Parallel work can skip review, it's isolated"     | Isolation is about files, not quality. The loop still applies to each task.    |
| "Not sure if these depend on each other — send it" | Unsure means sequence it. The safe default is not to fan out.                  |

## Completion Check

A parallel run is done correctly when ALL are true:

- [ ] Every fanned-out task was confirmed independent (no shared files, interface, or order).
- [ ] Dependent tasks were sequenced, not fanned out, and this was stated.
- [ ] Each agent had its own worktree and `feat/NNN-slug` branch, with a clean baseline.
- [ ] Each agent got crafted context, not your own session history.
- [ ] Every agent result was checked against its `Verify:` line — not trusted as reported.
- [ ] Each task passed the loop's `verify` and `review` (fresh session).
- [ ] Branches merged one at a time to green; worktrees cleaned up.
