---
name: roadmap
description: "Use after the PRD (docs/product/PRD.md) is approved, to slice the project into features, group them into phases, and set up specs/. Produces specs/_roadmap.md, specs/_state.md, the specs/_status.md PR ledger, and — via a spec-writer subagent, one per phase — the full specs/NNN-slug/spec.md for every feature. Reviews the roadmap with Lavish, then the specs with Lavish, before any design starts. Also use to replan when a phase closes or scope changes, and to revise a feature's spec after it is approved (create / replan / revise). It is the only writer of spec.md. Do not use to cut a feature into tasks (that is `planning`), to shape the solution (that is `system-design`), or to explore a vague idea or write the PRD (that is `brainstorming`)."
argument-hint: "Optional: 'create' (default), 'replan' (phase closed or scope changed), or 'revise <NNN-slug>' (change an approved spec)."
---

# Roadmap

Turns an approved PRD into an ordered set of features grouped into phases, **and into one approved spec per feature**, then keeps that plan honest as work finishes.

Workflow position: `brainstorming` (project mode) → **`roadmap`** → `system-design` → `planning` (per feature) → implement → verify → review → deploy → back to `roadmap` (replan). A change to an approved spec re-enters here as **Revise**, then flows on by feature state.

Three axes, never mixed:

- **Feature:** the unit of work (a vertical slice with observable behavior).
- **Phase:** a group of features that together deliver something that runs.
- **Stage:** the steps inside one feature (design, plan, implement, verify, review, deploy). A concept, not a stored field — see "Stage vocabulary" below.

This skill owns phases, the feature list, **and the feature specs** (written by a dispatched spec writer, never inline). It does not write `design.md`, `tasks.md`, or any code, and it never reopens the PRD.

## When to Use

- The PRD (`docs/product/PRD.md`) is approved and the project has no `specs/_roadmap.md` yet.
- Every feature of the current phase is `done`, the PRD or scope changed, or a blocked feature proves the plan wrong.
- A feature's approved spec must change (after the roadmap, mid-build, or after it shipped) — mode **Revise**.
- The user says "roadmap", "chia phase", "phase review", "what's next", "replan", "đổi spec", or "sửa tính năng".

**Do NOT use** to shape the solution or pick boundaries (that is `system-design`), to cut a feature into tasks (that is `planning`), or to explore a vague idea or write the PRD (that is `brainstorming`, project mode).

## Inputs

- `docs/product/PRD.md` — approved, with `FR-n` / `NFR-n` IDs.
- `docs/architecture/architecture.md` and `docs/architecture/adr/` — when they exist, so features follow agreed conventions.
- Existing `specs/` — the roadmap, the state cursor, and each feature's `status.md` (replan mode).
- `AGENTS.md` — the project's branch, PR-size, and test-command conventions.

## Pick the mode

- **Create:** no `specs/_roadmap.md` yet, or the user asks to start over.
- **Replan:** `_roadmap.md` exists and either every feature of the current phase is `done`, the PRD changed, a blocked feature shows the plan is wrong, or the user asks to change scope.
- **Revise:** a feature's `spec.md` is `approved` and someone wants it different — after the roadmap, mid-build, or after it shipped. Pick this whenever the request changes an existing requirement rather than adding a new feature. The routing rules — and why a shipped feature gets a follow-up feature instead — are in [references/change-requests.md](./references/change-requests.md).

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

**Every phase's features get cut now, with IDs.** "Phase detail is lazy" governs how much prose `_roadmap.md` carries, not whether a feature exists: each feature of each phase leaves this step with an `NNN-slug`, a one-line description, `covers`, and `depends_on`, so the spec writer in step 6 can spec them all. A phase whose features cannot be named yet is written as a sketch — and that phase gets its features cut at the next replan, before its specs are written.

### 4. Coverage check

Every FR and NFR is covered by at least one feature, or explicitly marked deferred or out of scope with a reason. List gaps.

### 5. Review the roadmap with Lavish, then get approval

Write `specs/_roadmap.md` with `status: draft` (template in `references/templates.md`).

Show it with the **`lavish`** skill — the roadmap artifact (phases, features with `covers`/`depends_on`, the coverage table, the calls you made), using the `plan` playbook. Open it (`lavish-axi <html-file>`) and keep listening: run `lavish-axi poll <html-file>` in the **foreground** and leave it running — do not end your turn or kill it, or the browser shows "Your agent is not listening". Answer with `lavish-axi reply <html-file> --agent-reply "..."` (or re-poll with `--agent-reply`) so the board is not stuck on `Working`. If `lavish` is not installed, say so and present the same content as a Markdown summary in chat.

This is **gate one**: the human approves the _slicing_ — phases, feature boundaries, `covers`, `depends_on` — before any spec gets written. Fold their feedback into `_roadmap.md` and re-show while it is still changing. Only the current phase needs full detail in the artifact; later phases show their feature rows, since they are the input to their specs.

### 6. Write the specs, review them, then set up specs/

Approval of the roadmap is **not** the end of this skill: the human's yes starts this step. A run that stops after step 5 produced one file and is incomplete.

**6a. Dispatch one spec writer per phase.** Fill [references/spec-writer-prompt.md](./references/spec-writer-prompt.md) and dispatch **one subagent per phase** — the spec writer for phase N gets that phase's feature rows verbatim, the PRD requirements those features `covers`, and the exit criteria. Phases are independent, so dispatch them **in parallel**; the current phase's spec writer is never blocked by a later phase's. Each one writes `specs/NNN-slug/spec.md` for every feature of its phase. Do not write the specs inline, and never hand a subagent this session's history.

**6b. Check what came back, then review the specs with Lavish (gate two).** Treat the report as claims: verify each feature has its `spec.md` on disk with `covers`/`depends_on` matching `_roadmap.md` verbatim, every `AC-n` mapped to a requirement in the `AC → requirement mapping` table, and no feature over ~8 ACs. Then show every phase's specs in a Lavish artifact (one section per feature) and let the human approve the _content_ — acceptance criteria and open questions. This gate is separate from gate one on purpose: gate one fixes what the features _are_, gate two fixes what "correct" _means_ for each. On approval, set each spec's frontmatter `status: approved`. A rejected spec is re-dispatched **for its phase only**, with the feedback verbatim and the accepted specs marked as untouchable.

**6c. Files (always — pipeline or standalone):**

- Set `_roadmap.md` to `status: approved`.
- Write `specs/_state.md`.
- Create `specs/_status.md` — the project PR ledger, empty (template in `references/templates.md`).
- For the **current phase only**, create `specs/NNN-slug/status.md` for each feature (`status: todo`, `cycle: 0`, `cycle_cap: 3`, `next: design`). Template in `references/templates.md`. Status files for later phases wait until replan advances them — their `spec.md` already exists.
- Do **not** write `design.md` or `tasks.md`, and do not open a later phase.
- Set `current_feature` to the first feature whose dependencies are all `done`.

**6d. Handoff:** **in the pipeline**, invoke `system-design` for that feature (or `planning` if there is no design skill) and nothing else. **On its own**, invoke nothing — name the next step in one line. Either way, the 6c files exist before you stop.

## Replan mode

1. **Read state.** `_state.md`, `_roadmap.md`, each feature's `status.md` and `review.md` in the current phase.
2. **Verify the exit criteria.** Check each against evidence (tests, a demo, the feature statuses). If you cannot verify one, ask the user to confirm it. Do not assume.
3. **Record the phase outcome** in the Phase log of `_roadmap.md`: what shipped, what was cut or moved, what you learned.
4. **Re-run the coverage check** against the current PRD (the PRD may have changed).
5. **Adjust later phases:** reorder, split, add, or drop features. Explain every change in one line each. For a change that alters later phases, get the user's approval in one message.
6. **Mid-phase change requests:** add a feature to the current phase only if it is needed to meet its exit criteria; otherwise put it in a later phase or the backlog section.
7. **Re-validate the next phase's specs, then advance.** A phase's specs were written when it was planned, and the roadmap has moved since. Before the phase becomes current, diff each `specs/NNN-slug/spec.md` of that phase against `_roadmap.md` (feature renamed, split, merged, or dropped?) and against the current PRD (a `FR-n`/`NFR-n` it `covers` changed?). Matching specs keep their revision and are marked `status: approved`. Drifting ones get the delta only — the changed acceptance criteria — as a new `Revision:` line with a change log, then go back through gate two. A feature the replan add or recut has no spec yet: dispatch a spec writer for that phase (step 6a) and review it (6b).
8. **Advance:** mark the finished phase `done`, the next one `in-progress`, create `status.md` for its features, update `_state.md`, then hand off to `system-design` for the first ready feature (on its own, name that step in one line and stop). The new phase's specs are re-validated and its `status.md` files exist before you hand off.

If a feature is `blocked` because its spec or the PRD was wrong, send the user to `brainstorming` (project mode when the PRD changes; otherwise fix the spec here through the spec writer), then come back here to replan.

## Revise mode

A spec is approved and the user wants it changed — after the roadmap, mid-build, or after it shipped. **Route on whether the feature has shipped**, per [references/change-requests.md](./references/change-requests.md):

- **`todo` or `doing`** → revise `spec.md` in place. Rewrite it as a complete, stand-alone document (never a diff), bump the `Revision:` line, add a change-log entry (`NEW` / `CHANGED` / `UNCHANGED`), keep AC IDs stable (`AC-n (removed)` for a retired one), set `status: spec`, then re-run **gate two** with Lavish. If the user gave no brief, `brainstorming` (revision mode) interviews them and returns one — it does not write the file.
- **`done` and shipped** → do **not** edit that spec. Replan and cut a **follow-up feature** (`00N-slug-2`) with its own spec, branch, and PR. The shipped spec, its `review.md`, and its PR are the record of what "correct" meant then.
- **A shipped behaviour that is simply wrong** → a defect, not a revision: route to `debugging`, or a small follow-up feature if it needs its own PR.
- **The PRD itself is wrong** → `brainstorming` (project mode) fixes the PRD, then replan and re-run the coverage check.

Always:

- `roadmap` is the **only writer** of `spec.md`. One writer, one format.
- **Never create a second dated spec file.** History lives in the `Revision:` line, the change log, and git (`git log -p specs/NNN-slug/spec.md`). If a project has no usable version control, opt into `specs/NNN-slug/history/YYYY-MM-DD-spec.md` snapshots — no skill reads them.
- The **design can go stale**: if an interface, data model, or screen moved, set the feature's `status.md` `next: design` so `system-design` re-validates the delta at the Light tier. A wording-only revision skips that.
- The next plan must update `tasks.md`'s `Spec: … Revision <date>` header, or delta planning cannot tell what moved.

## Rules

- **Phase detail is lazy, feature existence is not.** Prose detail in `_roadmap.md` covers the current phase; every phase's features are cut with IDs, `covers`, and `depends_on` because their specs depend on it.
- **Specs are dispatched, never written inline.** One spec-writer subagent per phase, with crafted context from [references/spec-writer-prompt.md](./references/spec-writer-prompt.md) — never this session's history.
- **Two gates, both real.** Gate one: the human approves the slicing. Gate two: the human approves the specs' acceptance criteria and open questions. Neither is "sounds good" to a summary. A revision re-runs gate two — no shortcut for "it was only one AC".
- **One writer of `spec.md`.** This skill. `brainstorming` contributes a change brief; a second writer means a second format. A revision re-runs gate two — no shortcut for "it was only one AC".
- **One source of truth:** `_roadmap.md` for phases and features, `_state.md` for where we are now, `_status.md` for the landed-PR ledger, each `status.md` for one feature, each `spec.md` for what "correct" means. Update them the moment something changes, before moving on.
- **Never renumber** existing features or requirement IDs.
- **One question per message** when you must ask; otherwise state your call as a proposal.
- Write documents in the user's language; keep file names, IDs, and keys in English.
- No product code, no dependency installs, no design decisions.

## Stage vocabulary (shared with AGENTS.md)

The order a feature moves through: `design` → `plan` → `development-loop` (`implement` <-> `verify` <-> `review`) → `land` → `deploy` → `done` (a feature may also sit at `todo`). This is **not** a field stored in `status.md`: the position is inferred from the feature folder and pointed at by `next`. A review that finds a code issue returns to `implement`; one that finds a requirement issue returns here — `roadmap` writes a spec revision (delta only) and re-runs gate two — a design issue returns to `system-design`, a task-list issue to `planning`. `cycle` increments on each return of the **current task**; past `cycle_cap`, the task is parked with a ruling and the human is asked.

## Red Flags

| Thought                                           | Reality                                                                                         |
| ------------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| "I'll create folders for every phase now"         | `status.md` is current-phase only. A spec per feature exists for every phase — a folder is not. |
| "I'll split by layer: DB, then API, then UI"      | Nothing runs until the end. Slice vertically.                                                   |
| "I'll write the specs myself while I'm here"      | Dispatch a spec writer per phase. You own the gates and the checks, not the drafting.           |
| "The roadmap is approved, so the specs can wait"  | Gate one approves the slicing; gate two approves the specs. No design starts until gate two.    |
| "I'll skip the Lavish review, it's clearly right" | Both gates are Lavish reviews. A gate you did not run is a gate you did not pass.               |
| "The specs for later phases might drift, fine"    | Re-validate them at replan (step 7). A stale spec goes back through gate two, delta only.       |
| "I'll write a new dated spec file for the change" | Bump `Revision:` in `spec.md`. A second current spec means half the readers pick the wrong one. |
| "The feature already shipped, I'll edit its spec" | A follow-up feature instead. The shipped spec is the record of what was agreed then.            |
| "It's one AC, I'll skip gate two on the revision" | One AC changes what "done" means. Gate two is cheap; a wrong AC costs a whole loop.             |
| "Only wording moved, the design still fits"       | Check interface, data model, screen. If any moved, re-validate the design delta.                |
| "The phase probably met its exit criteria"        | Verify, or ask.                                                                                 |
| "The PRD has no IDs, I'll skip coverage"          | Add IDs, then check coverage.                                                                   |
| "The roadmap is approved, so I'm done"            | Approval starts step 6. Without the specs and 6c's files the run is incomplete.                 |

## Completion Check

**Create and Replan modes** are done when ALL are true:

- [ ] Every feature of **every** phase has an ID, a one-line description, `covers`, and `depends_on`; every FR/NFR is covered or explicitly deferred with a reason.
- [ ] Phases are ordered by dependency with verifiable exit criteria; only the current phase carries prose detail.
- [ ] `specs/_roadmap.md` is written, the human **approved it through a Lavish review** (gate one), and its `status` is `approved`, not `draft`.
- [ ] Every feature has `specs/NNN-slug/spec.md` on disk, written by a dispatched spec writer — with `feature`, `phase`, `covers`, `depends_on` matching `_roadmap.md`, a `Revision:` line, testable `AC-n`, and an `AC → requirement mapping` table.
- [ ] The specs were **approved through a Lavish review** (gate two) and every spec's frontmatter reads `status: approved`; no feature carries more than ~8 ACs.
- [ ] `specs/_state.md` exists and `current_feature` is the first ready feature whose dependencies are all `done`.
- [ ] `specs/_status.md` exists with its header and empty ledger table.
- [ ] The current phase's features each have `specs/NNN-slug/status.md` (`status: todo`, `cycle: 0`, `cycle_cap: 3`, `next: design`); no later phase has one yet.
- [ ] No `design.md` or `tasks.md` was written here, and no code was touched.
- [ ] The handoff was made (pipeline: `system-design` for the current feature; standalone: the next step named in one line).

**Revise mode** is complete when: the change was routed by feature state (`todo`/`doing` → revise; `done` → follow-up feature; PRD wrong → project mode); `spec.md` was rewritten as a stand-alone document with a bumped `Revision:` and a `NEW`/`CHANGED`/`UNCHANGED` change log; AC IDs stayed stable; gate two approved the changed criteria and `status` is back to `approved`; and a design touching an interface, data model, or screen sent the feature to `next: design`. No second dated spec file was created.

Missing specs, `_state.md`, `_status.md`, or the current phase's `status.md` files means the run stopped early: finish step 6 before reporting anything as done.
