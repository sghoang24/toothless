# Change Requests — revising a spec after it is approved

A spec is `approved` and someone wants it different. The answer depends on **one thing**: has the
feature shipped? Everything below routes on that, and `roadmap` is the **only writer** of
`spec.md` in every case.

## One writer, one format

| Skill           | Role in a revision                                                                                                                                                                                                                                                                              |
| --------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `brainstorming` | **Conversation only.** It interviews the user — why, what changes, what the new acceptance criteria are — and returns a **change brief**. It writes no file in the pipeline; two writers means two formats, which is how a spec ends up with a `status.md` frontmatter and no `Revision:` line. |
| `roadmap`       | **The writer.** Takes the change brief (or the user's request directly), applies the delta to `spec.md`, bumps `Revision:`, runs gate two.                                                                                                                                                      |

Standalone work (no roadmap) is the exception: `brainstorming` feature mode writes `spec.md` itself,
using the same skeleton. There is still only one writer for that project.

## Route by state

| #     | State                                                                  | Route                                                           | What happens                                                                                                                                                                                                                                                                 |
| ----- | ---------------------------------------------------------------------- | --------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **A** | Spec approved, nothing built yet (`status: todo`)                      | `roadmap` → **Revise**                                          | Edit `spec.md` in place: bump `Revision:`, add a change-log entry, re-run gate two. `planning` then runs its first plan.                                                                                                                                                     |
| **B** | Feature mid-build (some tasks ticked)                                  | `roadmap` → **Revise** → `planning` revision-delta              | Only NEW/CHANGED requirements get tasks. Unbuilt superseded tasks are struck through (`~~T4~~ dropped: …`), built ones stay. Nothing already merged is rewritten.                                                                                                            |
| **C** | Feature `done` and shipped — **new or extended behaviour**             | `roadmap` → **Replan** → a **follow-up feature** (`00N-slug-2`) | Normal pipeline: new feature row, new spec, new branch, new PR, new ledger row. **Do not edit the shipped spec.** Its AC, its `review.md`, and its PR are the record of what "correct" meant at that point in time; editing it rewrites history and breaks every link to it. |
| **D** | Feature shipped — the shipped behaviour is **wrong** (bug, regression) | `debugging` (+ a small task)                                    | Not a spec change. A defect against an unchanged requirement is a fix, not a revision. If it is big enough to need its own PR, it is a follow-up feature.                                                                                                                    |
| **E** | The **PRD itself** is wrong (a `FR-n`/`NFR-n` misstates the need)      | `brainstorming` **project mode** → PRD revision log             | Then `roadmap` replans and re-runs the coverage check, because the requirement→feature mapping may have changed.                                                                                                                                                             |

Rule of thumb: **`todo` or `doing` → revise the spec. `done` → cut a new feature. PRD wrong → fix the
PRD first.**

## Applying a revision (modes A and B)

1. **Read the change brief** (or take the request directly) and the current `spec.md`. Never revise
   from memory, and never revise a spec you have not read in full.
2. **Rewrite `spec.md` as a complete, stand-alone document** — not a diff. The body always states the
   _current_ truth; the change log is what carries the history.
3. **Bump the revision** and add the change log at the top:

   ```markdown
   Revision: 2026-11-27 (supersedes 2026-11-20)
   Changes since 2026-11-20:

   - NEW: AC-6 — a partner can revoke a shared note
   - CHANGED: AC-3 — a partner sees the note read-only, never edits it
   - UNCHANGED: AC-1, AC-2, AC-4, AC-5 (by name only)
   ```

   AC IDs are stable: a changed criterion keeps its ID, a new one takes the next free number, and a
   retired one is marked `AC-n (removed)` — never renumbered, because `tasks.md` and the tests refer
   to them.

4. **Update the frontmatter**: `status: spec` (it is unapproved again), new `revision` date. Keep
   `covers`/`depends_on` in sync with `_roadmap.md` — if a requirement changed, the roadmap row may
   need it too.
5. **Re-run gate two** with Lavish — the human approves the changed criteria. Same gate, no shortcut
   because "only one AC moved".
6. **Set `status: approved`** once they do, then hand off per state:
   - **A:** `planning` (first plan).
   - **B:** `planning` (revision-delta mode) with the change log as its input.

## Consequences people forget

- **The design may be stale.** If the revision touches an interface, a data model, or a screen,
  `specs/NNN-slug/design.md` and `docs/architecture/architecture.md` may no longer match it. Set the
  feature's `status.md` `next: design` and let `system-design` re-validate the delta at the Light
  tier — it does not re-design the feature from scratch. A wording-only revision skips this.
- **`tasks.md` records the revision it was planned from.** Its header line
  (`Spec: specs/NNN-slug/spec.md — Revision <date>`) must be updated on the next plan, or
  `planning`'s delta mode cannot tell what moved.
- **Never create a second spec file.** `spec.md` is the current spec for every feature. History lives
  in the `Revision:` line, the change log, and git:
  `git log -p specs/NNN-slug/spec.md`, `git show <sha>:specs/NNN-slug/spec.md`.
- **`history/` snapshots are opt-in, and normally off.** Only a project with no usable version
  control needs them. When enabled, `roadmap` copies the outgoing revision to
  `specs/NNN-slug/history/YYYY-MM-DD-spec.md` **before** overwriting. No skill reads `history/`; it is
  a lookup aid, not a source of truth. Everything else keeps trusting git — a second copy of the truth
  drifts.

## Red Flags

| Thought                                                       | Reality                                                                                                                          |
| ------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| "I'll write a `2026-11-27-spec.md` so it's easy to reference" | Two current specs means every reader picks one and half of them are wrong. Bump `Revision:` in `spec.md`; git holds the old one. |
| "The feature shipped, I'll just edit its spec"                | `done` means cut a follow-up feature. The shipped spec is the record of what was agreed then.                                    |
| "I'll let `brainstorming` write the file, it has the context" | One writer. `brainstorming` returns a change brief; `roadmap` writes.                                                            |
| "It's a one-line AC change, I'll skip gate two"               | A one-line AC change is a one-line change to what "done" means. Gate two is cheap; a wrong AC costs a whole loop.                |
| "Only the wording moved, the design is fine"                  | Check it: interface, data model, screen. If any moved, re-validate the design delta.                                             |
| "I'll delete the retired AC so the file stays clean"          | Mark it `AC-n (removed)`. `tasks.md` and the tests point at it.                                                                  |
| "The revision supersedes built work, I'll rewrite that task"  | If it is built, add a task to change it. If it is not, strike it through with a reason. Never rewrite what happened.             |
