# Question Bank

Use sparingly — only where a wrong guess would change the plan. Prefer stating
your call and letting the user correct it. If you do need to ask, ask **one
question at a time** and wait for the answer before the next — never a batch.

## Sequencing & dependencies

- Which specs must be built before others can start? (data models, auth, shared services, APIs)
- Which specs share foundations, so those should land early?
- Is there a hard external deadline or release that fixes an order?
- Which specs are independent enough to build in parallel (separate worktrees)?

## Phases (grouping specs)

- Which specs are related enough to share a phase (same goal or foundation)?
- Is any spec large enough to be its own phase?
- What must be demoable at the end of each phase, and which specs close?
- Is there a "walking skeleton" — the thinnest end-to-end path — we should hit first?
- Which specs are nice-to-have and can slip to a later phase?

## Sizing & review

- How big a change will the reviewer accept as one PR? (default: `AGENTS.md` ~300 lines)
- Does a spec really describe several features that should be named separately?
- Any tasks that must stay together (migration + model, contract + client)?

## Spec revisions

- Which existing specs changed (a newer `Revision:` in `spec.md`), and what does its change log mark NEW/CHANGED?
- Is the change additive (new features) or does it supersede work already shipped?
- Which requirements are UNCHANGED (confirm nothing here needs re-planning)?
- Any unbuilt tasks from the old revision this change replaces (to strike through)?
- Which new phase should the NEW/CHANGED delta land in — a fresh one, or a still-open future phase?
- Does the revision justify a new feature instead (a different requirement), or stay a revision of this one?

## Risk

- What's the riskiest assumption in this order? What would invalidate it?
- Any spec likely to churn requirements mid-build?
- What's the fallback if a task overruns its cycle cap (park it with a ruling and continue)?

## Layout preference

- Confirm the plan lives in `specs/_roadmap.md` (phases + features), the cursor in
  `specs/_state.md`, and per-feature `tasks.md` files in each `specs/NNN-slug/`.
  Prefer a flat backlog instead? (see `repo-layout.md`)
