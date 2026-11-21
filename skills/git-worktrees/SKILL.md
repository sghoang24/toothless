---
name: git-worktrees
description: "You MUST use this to run work in parallel: an isolated git worktree on its own branch, so each agent (or spec) gets a separate workspace without colliding. The goal is one branch and a clean main tree per piece of work, torn down when it lands. You are not allowed to check the same branch out in two worktrees, and you must never force-remove a worktree with uncommitted work. Trigger phrases: worktree, parallel agents, isolated checkout, clean up worktree."
argument-hint: "Point at the work, e.g. 002-search (spec id) or a branch name."
---

# Git Worktrees

Parallel work needs parallel space. A worktree is a second working directory on its own branch, sharing the same git history — so two agents (or two specs) can build at once without stepping on each other's files or index.

Per `AGENTS.md`, one feature = one branch = one PR. When several features are ready in the roadmap, a worktree per feature lets them proceed in parallel. This skill is **self-contained** — use it whenever you want an isolated workspace; whoever is coordinating (you directly, or `fight` when it's active) sets up and tears down the space for it.

## The Model

| Thing         | What it is                                                    | Where it lives            |
| ------------- | ------------------------------------------------------------- | ------------------------- |
| **Worktree**  | A second working dir on its own branch, sharing one history.  | `<parent>/<repo>-<slug>/` |
| **Branch**    | One feature = one branch = one PR.                            | `feat/NNN-slug`           |
| **Main tree** | The original checkout, usually on `main`/a long-lived branch. | repo root                 |

## When to Use

- Two or more independent specs/features should be built at the same time.
- You want to compare or run two branches side by side without stashing.
- You're starting a feature and want an isolated workspace that keeps the main checkout clean.

**Do NOT use** when the work is a single task on the current branch — just work in place. And never use worktrees as a way to work on the same branch in two places: one branch, one worktree.

## Core Rules

- **One branch, one worktree.** Git refuses a branch checked out in two worktrees; don't fight it.
- **Name for the work, not the person.** Use the spec id (`002-search`), so the space is traceable to the plan.
- **Create the branch first** (or let the worktree create it), named `feat/NNN-slug` per `AGENTS.md`.
- **Verify a clean baseline.** Run the project's setup and its test command in the new worktree before starting — a red baseline you didn't cause goes in your report by name.
- **Never force-remove a worktree with uncommitted work.** If it has changes, stop and ask; it may be the user's in-progress work.
- **Clean up when the branch is done.** Once the branch is landed or discarded (`closing-a-phase`), remove its worktree (this skill owns that clean-up).

## Procedure

### 1. Pick the work and its branch

From the roadmap (`specs/_roadmap.md`) and the feature's `specs/NNN-slug/status.md`, pick an independent feature that can run in parallel. Its branch is `feat/NNN-slug` (`AGENTS.md`). Confirm it isn't already checked out somewhere: `git worktree list`.

### 2. Create the worktree

Place it as a sibling of the main repo so it's easy to find and outside the working tree:

```bash
git worktree add -b feat/NNN-slug ../<repo>-NNN-slug
```

For a throwaway comparison, add one on an existing branch instead of `-b`.

### 3. Set up and verify the baseline

In the new worktree, install dependencies and run the project's declared test command (`AGENTS.md` — e.g. its setup + test targets) to confirm a clean baseline before writing anything. A red baseline you didn't cause goes in your report by name.

### 4. Work

Hand the worktree to the loop — `implement -> verify -> review` (`development-loop`). Whoever is coordinating keeps the main tree clean while this runs.

### 5. Clean up

When the branch is finished and merged (or discarded), remove the worktree:

```bash
git worktree remove ../<repo>-NNN-slug
```

Pass `--force` **only** after confirming there are no uncommitted changes you'd lose. Then prune stale entries: `git worktree prune`.

## Red Flags

| Thought                                       | Reality                                                                        |
| --------------------------------------------- | ------------------------------------------------------------------------------ |
| "Two worktrees on the same branch is fine"    | Git won't allow it, and it wouldn't help anyway. One branch, one worktree.     |
| "I'll force-remove it, there's nothing there" | Check for uncommitted changes first — they may be the user's in-progress work. |
| "Skip setup, I'll just start coding"          | An unverified baseline hides failures you'll later blame on your change.       |
| "Name it after me / just `tmp`"               | Name by spec id so the space traces back to the plan.                          |
| "Leave finished worktrees lying around"       | Remove them when the branch is done; run `git worktree prune`.                 |

## Completion Check

A worktree is used correctly when ALL are true:

- [ ] It's on its own `feat/NNN-slug` branch, and `git worktree list` shows no branch in two places.
- [ ] It's a sibling of the main repo, named for the spec.
- [ ] Dependencies installed and the test command run — baseline confirmed green (or the pre-existing red named).
- [ ] Work was done through the phase skills; the main tree stayed clean.
- [ ] When finished, the worktree was removed (no force while uncommitted work existed) and `git worktree prune` run.
