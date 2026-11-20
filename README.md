# Toothless

A portable **engineering workflow** for coding agents, packaged as a set of composable
**skills**. It turns a vague request into shipped, reviewed, tested code through one
disciplined pipeline — and it works across agent harnesses (Claude Code, Codex, Omp, …).

The point of Toothless is to stop the agent from jumping straight to code. Every change
moves through an explicit sequence of phases, each with its own gate. Each phase is a
**self-contained skill** you can use on its own; enter **fight mode** and the agent also becomes the
single coordinator ("chiefmate") that runs the phases in order, enforces the gates, dispatches
subagents, and keeps the loop honest.

> Think of it as the methodology layer, like superpowers — not a project template.

## The workflow spine

```
Brainstorm -> Roadmap -> Design -> Plan -> Development (loop) -> Deploy
```

Every change walks this pipeline. The **gates** between phases are hard — nothing skips one:

| Gate             | Rule                                                                                                                                                                        |
| ---------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| PRD → Roadmap    | No roadmap without an **approved PRD** (`docs/product/PRD.md`) and its `FR-n` / `NFR-n` IDs.                                                                                |
| Roadmap → Design | Two Lavish reviews passed: the **slicing** (`specs/_roadmap.md`, `specs/_state.md`) and the **specs** — every feature has `specs/NNN-slug/spec.md` with `status: approved`. |
| Design → Plan    | No planning until the **design is agreed** (`docs/architecture/architecture.md`, plus `docs/design/` for a UI).                                                             |
| Plan → Implement | No code before an **approved task list** (`specs/NNN-slug/tasks.md`) and its `## Test strategy`.                                                                            |
| Loop → next unit | The loop cap is **3 cycles per unit** (a task, or a batch of small same-shape tasks); each cycle must change the approach, and after the third, STOP and ask a human.       |
| Land             | A finished branch gets an **MR/PR raised for the human**; **merging needs explicit human approval** — never merge on your own.                                              |
| Deploy           | Requires **explicit human approval**. Never deploy on your own.                                                                                                             |

## Install

Install the skills into your project with the open
[skills CLI](https://github.com/vercel-labs/skills) (works with 70+ agents — Claude Code,
Codex, Copilot, Cursor, Omp, …):

```bash
npx skills add sghoang24/toothless          # choose skills + target agent
npx skills add sghoang24/toothless --list   # browse before installing
```

Add `-g` to install for all your projects instead of just the current one. Skills are plain
`SKILL.md` folders, so any agent that reads the Agent Skills layout can load them.

## Quick start

1. Install the skills (above), or open your agent CLI in a clone of this repo.
2. Start fight mode: run `/fight`, or just say **"fight mode"** — or use any skill directly.
3. State what you want to build. The coordinator takes it from there — it will brainstorm the
   PRD, slice and spec the roadmap (two Lavish reviews), design, plan the tasks (and the test
   strategy), then run the development loop task by task, stopping at each phase boundary for your
   sign-off.

`commands/fight.md` is the thin `/fight` entry point; the real rules live in the
`fight` skill.

### Basic workflow (calling each skill by hand)

If you don't want fight mode to drive, call the skills yourself **in this order** — each one
consumes the previous one's artifact:

| #   | Skill                            | What it gives you                                                                                         |
| --- | -------------------------------- | --------------------------------------------------------------------------------------------------------- |
| 1   | `brainstorming`                  | The approved **PRD** (`docs/product/PRD.md`)                                                              |
| 2   | `roadmap`                        | **Features** in **phases**, `specs/` set up, and an approved **`spec.md` per feature** (two Lavish gates) |
| 3   | `system-design` (+ `ui-design`)  | The agreed **design** (`docs/architecture/`, `docs/design/`, `specs/NNN-slug/design.md`)                  |
| 4   | `planning` (+ `test-strategy`)   | The **task list** with its test strategy (`specs/NNN-slug/tasks.md`)                                      |
| 5   | `development-loop`               | The tasks **built**, verified, and reviewed, one unit at a time                                           |
| 6   | `finishing-a-development-branch` | The **MR/PR** raised for you to review                                                                    |
| 7   | `deploy (optional)`              | The proven candidate **in production** (on your approval)                                                 |

**Enter anywhere the artifact already exists.** Already have a PRD? Start at step 2. Already have
an approved spec and a task list? Start at step 5. Each skill checks its inputs and sends you back
a step if one is missing — that's the point, not an obstacle.

Writing a spec for a single feature **without** a roadmap is the one place `brainstorming` feature
mode is the right call: it writes the same `specs/NNN-slug/spec.md` shape, from a conversation
instead of the approved feature rows.

Two things change the shape of that list: `e2e-testing` runs before step 7 at a milestone, and
**`debugging`, `security-and-hardening`, `git-worktrees`, and `dispatching-parallel-agents` are
on-demand** — call them whenever the work needs them, not at a fixed point in the sequence.

## The skills

Skills live in `skills/<name>/SKILL.md`. Each is **self-contained** — a standalone unit you can use
on its own, with no orchestrator required. `fight` is the optional **center**: when it's active
it sequences the skills and owns the gates, calling each one as the work reaches it. Use the skills
directly, or let fight drive — the skills don't depend on it.

### Center (optional)

- **fight** — the optional center: the coordinator persona ("chiefmate") and the workflow
  sequence: phase order, the gates, subagent dispatch, and session state. Enter
  it with `/fight` when you want the agent to drive the whole pipeline. It does not own _how_
  each phase is done — it calls the phase skills below, each of which also works on its own — nor
  the loop's rules, which live in `development-loop`.

### Phase skills (`Brainstorm -> Roadmap -> Design -> Plan`)

- **brainstorming** — turns a vague request into an agreed, testable document. Asks the questions
  that change what gets built, explores the codebase before proposing, lays out real
  alternatives. Project mode produces `docs/product/PRD.md`. Feature mode writes
  `specs/NNN-slug/spec.md` and is for **standalone work** only; its **revision mode** interviews you
  and returns a change brief — inside the pipeline the roadmap owns feature specs.
- **roadmap** — slices the approved PRD into **features**, groups them into **phases**, orders
  them by dependency, and sets up `specs/`. Then it writes the **spec for every feature of every
  phase** through one dispatched spec-writer subagent per phase, and gates both halves with
  **Lavish** reviews: first the slicing, then the acceptance criteria. Output: `specs/_roadmap.md`,
  `specs/_state.md`, `specs/_status.md`, `specs/NNN-slug/spec.md` for every feature, and
  `status.md` for the current phase's features. It is the **only writer of `spec.md`**, and also
  **replans** as phases close or scope shifts — re-validating the next phase's specs from their
  change logs.
- **system-design** — the **Design phase orchestrator**. It right-sizes the design
  (Light / Standard / Deep), dispatches the design work to fresh subagents, reconciles them,
  runs the Lavish review, and writes the agreed design. Output:
  `docs/architecture/architecture.md` (+ `docs/design/` for a UI) and ADRs.
- **ui-design** — the UI half of Design: turns the spec into screens, flows, states, and a
  design system, then mocks them as **static HTML** (no product code) for review. Dispatched by
  `system-design`; uses `ui-ux-pro-max` for the visual direction when it's installed.
- **planning** — plans **one feature**: turns the agreed spec and design into an ordered list of
  PR-sized tasks, each traced to acceptance criteria and carrying its own tests. It closes the Plan
  phase by settling the feature's `## Test strategy` (via `test-strategy`). Output:
  `specs/NNN-slug/tasks.md`.

### Loop skills (`implement -> verify -> review`)

- **development-loop** — the **whole loop, contained in one skill**. It takes an approved task list
  and drives the feature's tasks to landed code, one at a time: a fresh **implementer** subagent per
  task (which writes the test first, watches it fail, makes it pass, and proves it green — the
  `verify` bar lives in its evidence), then a fresh **reviewer** subagent, cycles capped at **3 per
  task**, and a **final whole-branch review** after the last task. It owns the loop's rules, so
  neither `fight` nor any single step restates them. Its step detail and subagent prompts live
  in `skills/development-loop/references/`.

  The three steps are the pack's shared vocabulary: **`implement`** builds one task test-first,
  **`verify`** holds the fresh-evidence gate (the implementer's own report, read and re-run by the
  coordinator), and **`review`** judges the diff against intent in a fresh session.

**Where "Test" lives.** Test strategy is **not** a loop step — `test-strategy` settles it once per
feature inside the Plan phase (command, layers, seams, which task pins which `AC`), written to
`specs/NNN-slug/tasks.md`. The loop then runs it: `implement` writes each task's tests test-first
(RED-GREEN-REFACTOR), `verify` proves them green, and `review` judges the diff. Keeping the
strategy separate avoids both layer-tasks ("now write the tests") and per-task strategy drift. A
feature's tests are unit/integration; end-to-end journeys are the milestone gate (`e2e-testing`),
not a per-feature step.

### Test strategy (Plan phase, not the loop)

- **test-strategy** — settles a feature's test strategy **once**, as the last step of the Plan
  phase: what to test across the feature, at which layer (unit / integration), where the mocking
  boundary sits, determinism and flakiness, coverage that matters, and how to find the project's
  test command. **End-to-end is not a per-feature concern** — assembled-product journeys belong to
  `e2e-testing` at a milestone. Decides _which_ tests (into `tasks.md`); the loop's `implement`
  writes them test-first and `verify` proves them green.

**Loop rules live in `development-loop`.** The cycle cap (3, **per unit**, then STOP and ask), the
"each cycle changes the approach" escalation, the fresh-session reviewer requirement, the
brief/report file protocol, and the final whole-branch review are the loop skill's — one source of
truth, so the optional center (`fight`) does not own them.

### Standalone skills (on-demand, outside the loop)

These are **not** part of the workflow's sequence — `fight` never dispatches to them and no
phase depends on them. You (or the user) invoke them when the work calls for it. Inside fight mode
their results flow back through the normal loop; standalone, they still hold their own discipline.

- **debugging** — root-cause a broken thing: reproduce → isolate → hypothesize → prove the cause →
  fix test-first, shipping a regression test. Use it whenever something is red or unexplained — a
  failing test, a crash, a regression, a bug report — not as a stage you always pass through. It
  owns _how_ you find a cause, never _when_.
- **security-and-hardening** — harden code and audit the two surfaces review forgets:
  **dependencies** (supply-chain / `npm audit` triage) and **personal data** (privacy / GDPR-CCPA).
  Walk the OWASP Top Ten ([references/appsec-checklist.md](./skills/security-and-hardening/references/appsec-checklist.md))
  when you want a security pass; Critical/High findings block and go back to `implement`.

### Landing skill (loop exit)

- **finishing-a-development-branch** — closes the task's branch: raises the **MR/PR for the human
  to review** (the default), or merges **only on explicit human approval** (treated exactly like a
  deploy), then cleans up the worktree. Never pushes to `main`/`master`, never force-pushes.

### Deploy skill (spine exit)

- **deploy** — the workflow's final phase: takes the **proven candidate** to production. Requires
  explicit human approval, a green e2e on the merge candidate, a known rollback, and the smallest
  reversible step the system allows; smokes the live system after deploy and rolls back on failure.
  Never on your own initiative.

### Supporting skills

- **git-worktrees** — run work in parallel: create an isolated worktree on its own
  `feat/NNN-slug` branch (one per spec/agent), verify a clean baseline, and clean up when done.
- **dispatching-parallel-agents** — fan out **independent** tasks to several agents at once
  (one worktree + branch each), with crafted context and sequential merge. Self-contained; only
  independent tasks fan out.
- **e2e-testing** — at a release/milestone boundary, prove the assembled product works end to
  end: pick the critical user journeys, run them against the real system, triage failures. Not
  a per-feature step; green e2e gates `deploy`.

## How a change flows end to end

1. **Brainstorm.** You describe what you want. The agent asks the few questions whose answers
   change what gets built, explores the code, and proposes the approaches.
   → an approved PRD at `docs/product/PRD.md`.
2. **Roadmap.** The PRD is sliced into **features**, grouped into **phases**, ordered by
   dependency, and you approve the slicing through a **Lavish** review. Then one **spec writer**
   per phase writes `specs/NNN-slug/spec.md` for every feature, with testable acceptance criteria
   and an `AC → requirement mapping`; you approve those through a second Lavish review.
   → `specs/_roadmap.md`, `specs/_state.md`, `specs/_status.md`, and an approved `spec.md` per feature.
3. **Design.** `system-design` picks a tier, dispatches an **architect** subagent (and a **UI
   designer** when the spec has a UI), reconciles them, shows the result via **Lavish**, then
   writes `docs/architecture/architecture.md` (+ `docs/design/`, plus the feature's short
   `specs/NNN-slug/design.md`) and draws diagrams with **archify**.
4. **Plan.** `planning` turns the feature's spec and agreed design into an ordered list of
   PR-sized tasks, each traced to acceptance criteria with its own tests. As the last step of this
   phase, `test-strategy` settles the feature's `## Test strategy` (command, layers, seams).
   → `specs/NNN-slug/tasks.md`.
5. **Development loop.** For each task, `development-loop` extracts the task into a **brief file**,
   dispatches a fresh **implementer** subagent (which builds it test-first following the feature's
   test strategy, proves it green, and writes a report), then reads that evidence and dispatches a
   fresh **reviewer** against the `BASE..HEAD` diff. Clean verify plus clean review → the task is
   done; red → back to `implement` with the cause found before the retry (optionally via the
   standalone `debugging` skill). Capped at **3 cycles per unit** (a task, or a batch of small
   same-shape tasks), each changing the approach. After
   the last unit, one **whole-branch review** judges the feature as a whole.
6. **Land.** `finishing-a-development-branch` raises the **MR/PR for you to review** — the default.
   It merges only when you explicitly approve (like a deploy), then cleans up the worktree.
7. **Parallelize (optional).** Independent tasks fan out via `dispatching-parallel-agents`, one
   worktree + branch each, merged sequentially.
8. **Deploy (milestone).** `e2e-testing` proves the assembled product; `deploy` takes the proven
   candidate to production — only with explicit human approval.

The coordinator **stops at every phase boundary** by default: it walks the phase's exit criteria
and targets in `specs/_roadmap.md`, reports what's true, and waits for you before starting the
next phase. Widen the run only when you ask ("run to the end of P1", "chạy liền P1→P2").

## Parallel work

To run several independent tasks at once, use **`git-worktrees`** for the isolated workspace
and **`dispatching-parallel-agents`** to fan out: **one worktree + separate `feat/NNN-slug`
branch per agent**. Only tasks with no shared files, interface, or ordering may fan out; the
rest stay sequential. Merge one branch at a time, to green. Keep concurrency small (≈3).

## Design subagents

The Design phase (`system-design`) is the orchestrator; the design work itself is dispatched to
fresh subagents under `skills/system-design/references/`:

- **architect** ([`architect-prompt.md`](./skills/system-design/references/architect-prompt.md))
  — drafts the system design at the chosen tier.
- **UI designer** ([`ui-design-prompt.md`](./skills/system-design/references/ui-design-prompt.md))
  — designs screens + static mocks when the spec has a UI.

Both get **crafted context** (the spec, the roadmap position, existing design, and the decisions
already answered), never the calling session's history. The design session reconciles the two
drafts and owns the review.

## Repository layout

This repo is the **source of the skill pack**. It ships a canonical `skills/` tree plus the
`/fight` command; the target project grows `specs/` and `docs/` as the workflow
runs (the first spec bootstraps what's missing).

```
.
├── README.md              # this guide
├── AGENTS.md              # source of truth for maintaining this skill pack
├── LICENSE                # MIT
├── package.json           # npm metadata + published files whitelist
├── commands/
│   └── fight.md      # thin /fight entry point
└── skills/                # the workflow, one folder per phase
    ├── README.md          # skills catalogue
    ├── fight/        #   spine + coordinator
    ├── brainstorming/     #   spec / PRD
    ├── roadmap/           #   features + phases + feature specs
    │                      #     (+ references/spec-writer-prompt.md, templates.md)
    ├── system-design/     #   design orchestrator (+ references/architect-prompt.md,
    │                      #     ui-design-prompt.md, design-depth.md, question-bank.md,
    │                      #     architecture-template.md)
    ├── ui-design/         #   screens + static mocks
    ├── planning/          #   per-feature tasks (+ references/tasks-template.md,
    │                      #     task-sizing.md, repo-layout.md, question-bank.md)
    ├── test-strategy/     #   Plan phase: feature test strategy (what/where/how much)
    ├── development-loop/  #   the loop: implement -> verify -> review, per unit (task or batch)
    │                      #     (+ references/implement.md, review.md, final-review.md,
    │                      #        implementer-prompt.md, reviewer-prompt.md, brief-and-report.md)
    ├── finishing-a-development-branch/  # loop exit: raise MR / merge on approval
    ├── deploy/            #   spine exit: ship the proven candidate (on approval)
    ├── debugging/         #   standalone: root-cause a broken thing (on demand)
    ├── security-and-hardening/  # standalone: app-sec + supply-chain + privacy (on demand)
    │                      #   (+ references/appsec-checklist.md,
    │                      #     supply-chain-and-privacy.md)
    ├── git-worktrees/     #   isolated parallel workspaces
    ├── dispatching-parallel-agents/  # fan out independent tasks
    └── e2e-testing/       #   milestone gate: prove the assembled product
```

In a project running the workflow, expect this structure to appear:

```
specs/
├── _roadmap.md            # phases (groups of features) + coverage (owned by roadmap)
├── _state.md              # current phase + feature (owned by roadmap)
├── _status.md             # landed-PR ledger: PR / phase / feature / task (owned by roadmap)
└── NNN-slug/              # one feature
    ├── spec.md            # requirements + AC-n (roadmap writes; `status: approved`)
    ├── status.md          # per-feature cursor: status / cycle / next (current phase only)
    ├── design.md          # shape of this feature's solution (system-design)
    ├── tasks.md           # the feature's PR-sized tasks
    └── review.md          # review findings (owned by the loop's `review`)
docs/
├── product/PRD.md
├── architecture/
│   ├── architecture.md    # the agreed system design
│   └── adr/               # hard-to-reverse decisions, one file each
└── design/                # agreed UI: screens, flows, states, design system
```

## Companion skills (install separately)

The workflow composes with three personal skills that live in `~/.agents/skills/`. They are
**tools, not dependencies** — the workflow degrades gracefully without them:

- **lavish** (used by the review loop) — turns a plan, design, or comparison into a reviewable
  HTML artifact you can annotate.

  ```bash
  npx skills add kunchenguid/lavish-axi --skill lavish
  ```

  Add `-g` to install it for all projects. See https://github.com/kunchenguid/lavish-axi.

- **archify** (used for design diagrams) — draws architecture, sequence, ERD, and lifecycle
  diagrams as standalone HTML. Install it from its own project instructions; `system-design`
  will ask you to install it, then falls back to Mermaid/text diagrams if you don't.

- **ui-ux-pro-max** (optional, used by `ui-design`) — UI/UX design intelligence for the visual
  direction.

  ```bash
  npx skills find ui-ux-pro-max        # then:
  npx skills add <owner/repo@skill>
  ```

  `ui-design` falls back to plain design reasoning when it isn't installed.

## Conventions that matter

Fight mode enforces the **target project's** `AGENTS.md` rather than restating it. (This repo's
own `AGENTS.md` governs _maintaining the skill pack_ — a separate concern.) The conventions the
workflow leans on most in a project:

- **Business logic lives in the service layer** — never in controller/interface.
- **One feature = one branch (`feat/NNN-slug`) = one PR**, diffs under ~300 lines.
- **Small commits**, Conventional Commits (`feat:`, `fix:`, `docs:`, `test:`, `chore:`).
- **Tests when behavior changes**, test-first when possible.
- **Boundaries (never):** commit secrets or edit `.env`; edit existing migrations (add new
  ones); add a dependency without asking; force-push; push to `main`/`master`; deploy without
  approval.

## Distribution

This repo is the **source of the Toothless skill pack** (like superpowers), not a project
template. It ships a canonical `skills/` tree plus per-harness adapters so the workflow can be
installed into an existing project. Install with the open skills CLI:

```bash
npx skills add sghoang24/toothless
```

The repo is also a valid npm package (`package.json` whitelists `skills/`, `commands/`,
`AGENTS.md`, `README.md`, `LICENSE`) so it can be published or consumed by tooling that reads
npm package metadata. `.omp/`, `.vscode/`, and other editor artifacts are git-ignored and never
shipped.

Design note: skills are **model-free** — the workflow describes _what_ each phase does, and the
harness decides _which model_ runs it. Per-role model policy (e.g. a stronger model for the
coordinator and reviewer, a cheaper one for exploration) belongs in the per-harness adapter
layer, not in `SKILL.md`.

## Still to add

- (none currently open)
