# Skills Catalogue

Every skill here is **self-contained** — usable on its own, addressed to you, with no orchestrator
required. `fight` is the optional **center**: when it's active it runs the sequence, owns the
gates, and calls each skill as the work reaches it. The skills below work with or without it.

- **fight** — the optional center: the workflow, the gates, the rules, subagent dispatch, and state. The coordinator persona (chiefmate). Enter with `/fight`. When active, it calls the other skills; when not, you use them directly.
- **brainstorming** — turn a vague request into an agreed, testable document: a PRD (`docs/product/PRD.md`, project mode) or a spec (`specs/NNN-slug/spec.md`, feature mode — standalone only). In the pipeline, `roadmap` owns feature specs; a change to an approved spec goes through its **revision mode**, which interviews you and returns a **change brief** rather than writing the file. When what you have is a **raw idea**, its **Idea Mode** sharpens it first (divergent, then convergent thinking + assumption stress-test) via [references/idea-refine.md](./brainstorming/references/idea-refine.md), then continues to the spec.
- **roadmap** — slice the approved PRD into **features**, group them into **phases**, order by dependency, and set up `specs/` (`specs/_roadmap.md`, `specs/_state.md`, `specs/_status.md`). Then dispatch **one spec writer per phase** to write `specs/NNN-slug/spec.md` for every feature of every phase, and gate both halves with **Lavish** reviews: the slicing first, then the acceptance criteria. Also replans as phases close or scope shifts — re-validating the next phase's specs from their change logs. It is the **only writer of `spec.md`**, including revisions after approval ([references/change-requests.md](./roadmap/references/change-requests.md)).
- **system-design** — orchestrate the Design phase: right-size the design, dispatch an **architect** per feature (in parallel across a phase) and, when a spec has a UI, a **UI designer** for the screens (`docs/design/`); reconcile per feature and across features, review with `lavish`, draw with `archify`. Designs one feature, or every feature of the current phase in one pass, so `planning` can plan the phase in one go.
- **ui-design** — the UI half of Design: turn the spec into screens, flows, states, and a design system, then mock them as static HTML (no product code) for review. Dispatched by `system-design`; uses `ui-ux-pro-max` for the visual direction when installed.
- **planning** — plan **a feature or a whole phase**: turn agreed specs and designs into ordered lists of PR-sized tasks in `specs/NNN-slug/tasks.md`, each traced to acceptance criteria and carrying its own tests. Phase mode plans every feature of the current phase in one run, once their designs are agreed, and settles one shared `## Test strategy` (via `test-strategy`).

The workflow these implement: `Brainstorm -> Roadmap -> Design -> Plan -> Development (loop) -> Deploy`.

## Loop skills (`implement -> verify -> review`)

- **development-loop** — build a feature's tasks, one unit at a time (usually one task, or a **batch** of small same-shape tasks), from an approved task list to landed code. It is the **whole loop, contained in one skill**: dispatch a fresh implementer subagent per unit (which writes the test first, watches it fail, and proves it green — the `verify` bar lives in its evidence), then a fresh reviewer subagent, cap the cycles at 3 per unit, and finish with a whole-branch review. Triggers: build the feature, run the loop, implement the tasks, work through tasks.md, batch small tasks. Its references hold the step detail ([implement.md](./development-loop/references/implement.md), [review.md](./development-loop/references/review.md), [final-review.md](./development-loop/references/final-review.md)) and the subagent prompts ([implementer-prompt.md](./development-loop/references/implementer-prompt.md), [reviewer-prompt.md](./development-loop/references/reviewer-prompt.md), [brief-and-report.md](./development-loop/references/brief-and-report.md)).
- **finishing-a-development-branch** — the loop's exit: raise the MR/PR for the human to review (the default), or merge **only on explicit human approval** (like a deploy), then clean up the worktree.

The three steps are the pack's shared vocabulary: `implement` builds one task test-first, `verify` holds the fresh-evidence gate, `review` judges the diff in a fresh session. They are defined once, in `development-loop`.

## Test strategy (Plan phase, not the loop)

- **test-strategy** — settle a test strategy **once per phase** (or per feature when planned alone), as the last step of the Plan phase: what to test, at which layer (unit / integration), where the mocking boundary sits, determinism and flakiness, coverage that matters, and how to discover the project's test command. **End-to-end belongs to the milestone**, not the feature (`e2e-testing`). Decides _which_ tests (into `tasks.md`); the loop's `implement` writes them test-first and `verify` proves them green.

**Where "Test" lives.** Test strategy is **not** a loop step — `test-strategy` sets it once per phase in the Plan phase, written to each feature's `tasks.md`. The loop runs it: `implement` writes each task's tests (RED-GREEN-REFACTOR), `verify` proves them green, and `review` judges the diff. Keeping the strategy separate avoids both layer-tasks ("now write the tests") and per-task strategy drift. A feature's tests are unit/integration; the assembled product is proven once, at the milestone (`e2e-testing`).

**Loop rules live in `development-loop`.** The cycle cap (3, per unit, then STOP and ask), the "each cycle changes the approach" escalation, the fresh-session reviewer requirement, the brief/report file protocol, the **batching** rule for small same-shape tasks, and the final whole-branch review are the loop skill's — one source of truth, so that neither `fight` (the optional center) nor each step owns them.

## Standalone skills (on-demand, outside the loop)

These are **not** part of the workflow's sequence — `fight` never dispatches to them and no stage depends on them. Invoke them when the work calls for it; inside fight mode their results flow back through the normal loop.

- **debugging** — root-cause a broken thing on demand: reproduce → isolate → hypothesize → prove the cause → fix test-first with a regression test. It is **not** a workflow stage — no phase routes a red result to it automatically. It owns _how_ you find a cause, never _when_, and holds its discipline whether or not you're in fight mode.
- **security-and-hardening** — harden code and audit the two surfaces review forgets: **dependencies** (supply-chain / `npm audit` triage) and **personal data** (privacy / GDPR-CCPA). It is a **standalone** skill, not a cross-cutting hook into the phases — run it when a change or a milestone warrants a security pass. Walks the OWASP Top Ten ([references/appsec-checklist.md](./security-and-hardening/references/appsec-checklist.md)); Critical/High findings block and go back to `implement`.

## Supporting skills

- **git-worktrees** — run work in parallel: create an isolated worktree on its own `feat/NNN-slug` branch (one per spec/agent), verify a clean baseline, and clean up when done.
- **dispatching-parallel-agents** — fan out independent tasks to several agents at once (one worktree + branch each), with crafted context and sequential merge. Self-contained; only independent tasks fan out.
- **e2e-testing** — at a release/milestone boundary, prove the assembled product works end to end: pick the critical user journeys, run them against the real system, triage failures. Not a per-feature step; green e2e gates `deploy`.

## Deploy skill (spine exit)

- **deploy** — the workflow's final phase: take the proven candidate to production. Requires explicit human approval, a green e2e on the merge candidate, a known rollback, and a smallest-reversible-step rollout; smokes the live system and rolls back on failure. Never on your own initiative.

## Design subagents

The Design phase (`system-design`) is the orchestrator; the design work itself is dispatched to fresh subagents under `system-design/references/`:

- **architect** ([architect-prompt.md](./system-design/references/architect-prompt.md)) — drafts the system design at the chosen tier.
- **UI designer** ([ui-design-prompt.md](./system-design/references/ui-design-prompt.md)) — designs screens + static mocks when the spec has a UI.

Both get crafted context (spec, plan, existing design, answered decisions), never the calling session's history; the design session reconciles the two drafts and owns the review.
