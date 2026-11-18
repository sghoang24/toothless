---
name: roadmap
description: "Use after the PRD (docs/product/PRD.md) is approved, to slice the project into features, group them into phases, and set up specs/. Produces specs/_roadmap.md, specs/_state.md, and the specs/_status.md PR ledger, and creates specs/NNN-slug/status.md folders for the current phase only. Also use to replan: when every feature in the current phase is done, when scope or the PRD changes, or when the user says 'roadmap', 'chia phase', 'phase review', 'what's next', or 'replan'. Do not use to write feature specs or plans (that is `planning`) or to explore a vague idea (that is `brainstorming`)."
argument-hint: "Optional: 'create' (default, from the approved PRD) or 'replan' (after a phase or a scope change)."
---

# Roadmap

Turns an approved PRD into an ordered set of features grouped into phases, and keeps that plan honest as work finishes.

Workflow position: `brainstorming` (project mode) → **`roadmap`** → `system-design` → `planning` (per feature) → implement → verify → review → deploy → back to `roadmap` (replan).

Three axes, never mixed:

- **Feature:** the unit of work (a vertical slice with observable behavior).
- **Phase:** a group of features that together deliver something that runs.
- **Stage:** the steps inside one feature (design, plan, implement, verify, review, deploy). A concept, not a stored field — see "Stage vocabulary" below.

This skill owns phases and the feature list. It does not write `spec.md`, `design.md`, `tasks.md`, or any code.

## Pick the mode

- **Create:** no `specs/_roadmap.md` yet, or the user asks to start over.
- **Replan:** `_roadmap.md` exists and either every feature of the current phase is `done`, the PRD changed, a blocked feature shows the plan is wrong, or the user asks to change scope.

## Create mode

### 1. Preconditions and reading

Read `docs/product/PRD.md`, `docs/architecture/architecture.md` if present, existing code and any existing `specs/`. If the PRD is missing or `status: draft`, stop and invoke `brainstorming` in project mode. Never ask what the PRD or the code already answers.

If the PRD lacks requirement IDs (`FR-n`, `NFR-n`), add them as a mechanical edit, log it in the PRD revision log, and tell the user.

### 2. Slice into features

- Each feature is a **vertical slice**: it delivers behavior someone can observe, not a technical layer. Avoid "build all models", "build all APIs", "build all screens".
- Size: one feature should fit one `development-loop` run (roughly a day of agent work, at most about 8 acceptance criteria). If a feature feels larger, split it.
- **Enabler features** (scaffold, CI, auth/RBAC, shared data model) are allowed when other features depend on them. Name them as enablers and say what they unblock.
- Each feature gets: ID `NNN-slug` (three digits, global, never reused; kebab-case ASCII), one-line description, `covers` (FR/NFR IDs), `depends_on`.
- Cross-cutting NFRs (security, observability) either become enabler features or are written as conventions in `docs/architecture/architecture.md` that every feature must follow. Say which.

### 3. Order and group into phases

Draw the dependency order and confirm there are no cycles. Then group by **value that runs**, not by technical layer. A typical shape (adapt, do not force):

| Phase               | Purpose                                                                  |
| ------------------- | ------------------------------------------------------------------------ |
| 0. Foundation       | Repo, CI, auth, shared model; exit: builds, CI green, empty deploy works |
| 1. Walking skeleton | Thinnest end-to-end path; exit: the main flow can be demoed              |
| 2. Core features    | The features that make it usable; exit: usable for the primary scenario  |
| 3. Hardening        | Performance, security, observability, edge cases, docs                   |
| 4+. Expansion       | Everything optional or later                                             |

Rules per phase: a one-sentence goal, **verifiable exit criteria**, usually 3 to 5 features, dependencies only on earlier or same-phase features. Skip a phase that the project does not need (a small project may have two phases).

### 4. Coverage check

Every FR and NFR is covered by at least one feature, or explicitly marked deferred or out of scope with a reason. List gaps.

### 5. Present for approval

Write `specs/_roadmap.md` with `status: draft` (template in `references/templates.md`) and show a summary in chat: phases, features, the coverage table, and the calls you made. Ask for approval in one message. Only the current phase gets detail; later phases get a one-line description per feature at most.

The user approves the roadmap, not individual decisions you stated as calls; list those calls so they can reject any.

### 6. After approval: set up specs/

- Set `_roadmap.md` to `status: approved`.
- Write `specs/_state.md`.
- Create `specs/_status.md` — the project PR ledger, empty (template in `references/templates.md`).
- For the **current phase only**, create `specs/NNN-slug/status.md` for each feature (`status: todo`, `cycle: 0`, `cycle_cap: 3`, `next: design`). Template in `references/templates.md`.
- Do **not** create folders for later phases and do **not** write `spec.md`, `design.md`, or `tasks.md`.
- Set `current_feature` to the first feature whose dependencies are all `done`.
- **In the pipeline:** invoke `system-design` for that feature (or `planning` if there is no design skill) and nothing else. **Running on its own:** invoke nothing; say in one line what the next step is and stop.

## Replan mode

1. **Read state.** `_state.md`, `_roadmap.md`, each feature's `status.md` and `review.md` in the current phase.
2. **Verify the exit criteria.** Check each against evidence (tests, a demo, the feature statuses). If you cannot verify one, ask the user to confirm it. Do not assume.
3. **Record the phase outcome** in the Phase log of `_roadmap.md`: what shipped, what was cut or moved, what you learned.
4. **Re-run the coverage check** against the current PRD (the PRD may have changed).
5. **Adjust later phases:** reorder, split, add, or drop features. Explain every change in one line each. For a change that alters later phases, get the user's approval in one message.
6. **Mid-phase change requests:** add a feature to the current phase only if it is needed to meet its exit criteria; otherwise put it in a later phase or the backlog section.
7. **Advance:** mark the finished phase `done`, the next one `in-progress`, create `status.md` folders for its features, update `_state.md`, then invoke `system-design` for the first ready feature.

If a feature is `blocked` because its spec or the PRD was wrong, send the user to `brainstorming` (feature mode, or project mode if the PRD changes), then come back here to replan.

## Rules

- **Phase detail is lazy.** Write detail only for the current phase; later phases stay one line per feature.
- **One source of truth:** `_roadmap.md` for phases and features, `_state.md` for where we are now, `_status.md` for the landed-PR ledger, each `status.md` for one feature. Update them the moment something changes, before moving on.
- **Never renumber** existing features or requirement IDs.
- **One question per message** when you must ask; otherwise state your call as a proposal.
- Write documents in the user's language; keep file names, IDs, and keys in English.
- No product code, no dependency installs, no spec content.

## Stage vocabulary (shared with AGENTS.md)

The order a feature moves through: `design` → `plan` → `development-loop` (`implement` <-> `verify` <-> `review`) → `land` → `deploy` → `done` (a feature may also sit at `todo`). This is **not** a field stored in `status.md`: the position is inferred from the feature folder and pointed at by `next`. A review that finds a code issue returns to `implement`; one that finds a requirement issue returns to `brainstorming`, a design issue to `system-design`, a task-list issue to `planning`. `cycle` increments on each return of the **current task**; past `cycle_cap`, the task is parked with a ruling and the human is asked.

## Red Flags

| Thought                                      | Reality                                         |
| -------------------------------------------- | ----------------------------------------------- |
| "I'll create folders for every phase now"    | Later phases will change. Current phase only.   |
| "I'll split by layer: DB, then API, then UI" | Nothing runs until the end. Slice vertically.   |
| "I'll write the spec while I'm here"         | That is `planning`. Stay at feature-list level. |
| "The phase probably met its exit criteria"   | Verify, or ask.                                 |
| "The PRD has no IDs, I'll skip coverage"     | Add IDs, then check coverage.                   |
