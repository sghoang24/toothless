---
name: system-design
description: "You MUST use this AFTER the spec (and its roadmap) are agreed and BEFORE planning and any implementation. The goal is to orchestrate the Design phase: right-size the design, dispatch an architect (plus a UI designer when the spec has a UI) with crafted context, reconcile them, review via Lavish, then write docs/architecture/architecture.md (+ docs/design/) and draw diagrams with archify. You are not allowed to start coding until the design is agreed. Trigger phrases: system design, architecture, design doc, how should we build this, architecture diagram."
argument-hint: "Point at the agreed spec, e.g. specs/001-auth/2026-11-12-auth.md (+ specs/_roadmap.md)"
---

# System Design (Design Phase Orchestrator)

Planning decides _in what order_ and _in what slices_; the Design phase decides _how the pieces fit together_ and _what the user sees_. This skill is the **orchestrator** of that phase: it right-sizes the work, dispatches it to subagents (an architect and, when there's a UI, a UI designer), reconciles what comes back, and drives the review to agreement.

It sits at **Design** in the workflow: `Brainstorm -> Roadmap -> Design -> Plan -> Development (loop) -> Deploy`. Design runs on the agreed **spec** and its **roadmap** position, and its output is what `planning` then cuts into tasks. It is **self-contained** — run it whenever a spec is agreed and you're about to design the build, whether or not `fight` is driving. Per `AGENTS.md`, no code without an approved spec — and no implementation starts until the design is agreed. Reviewer/implementer sessions read `docs/architecture/architecture.md` and `docs/design/`, not this conversation.

**Orchestrate, don't do it all inline.** The design work is dispatched to fresh subagents with crafted context — the architect gets [references/architect-prompt.md](./references/architect-prompt.md); the UI designer gets [references/ui-design-prompt.md](./references/ui-design-prompt.md). This skill owns the sequencing, the reconciliation between the two designs, the Lavish review, and the write-up. It never hands its own session history to a subagent.

**Subagents:** Dispatch the system design to the `architect` subagent and, when the spec has a UI, the UI design to the `designer` subagent. Name them explicitly in the delegation request (for example: "Use the `architect` subagent with `references/architect-prompt.md` and this context"). Each gets its prompt file and the crafted context only, never this session's history. If a named subagent is not available, say so and use a general subagent with the same prompt; never design inline.

## The Model

| Thing            | What it is                                                                    | Where it lives                                                     |
| ---------------- | ----------------------------------------------------------------------------- | ------------------------------------------------------------------ |
| **Spec**         | Approved requirements for one feature. The _what_ and _why_.                  | `specs/NNN-slug/YYYY-MM-DD-*.md`                                   |
| **Roadmap**      | Which feature, in which phase, in what order. The _when_.                     | `specs/_roadmap.md`, `specs/_state.md`                             |
| **Architect**    | Subagent that drafts the system design.                                       | [references/architect-prompt.md](./references/architect-prompt.md) |
| **UI designer**  | Subagent that designs screens + static mocks (only if the spec has a UI).     | [references/ui-design-prompt.md](./references/ui-design-prompt.md) |
| **Architecture** | Components, boundaries, data flow, interfaces, decisions. The _how_.          | `docs/architecture/architecture.md`                                |
| **UI design**    | Screens, flows, states, design system, static mock. The _what the user sees_. | `docs/design/`                                                     |
| **Diagram**      | A view of the design (component, sequence, ERD, ...) as standalone HTML.      | drawn via the `archify` skill                                      |

The system design and the UI design are **one design, agreed together**. The screens depend on the interfaces; whatever data a screen needs must exist in a contract. Neither is agreed until they reconcile.

The design **consumes** the spec (and the feature's place in the roadmap); it does not re-decide scope, requirements, or order. If design work reveals that a requirement is wrong, stop and send it back — a spec change goes through brainstorming (`roadmap` replans if scope shifts). Never quietly redesign around a broken spec.

## Right-size the design (read this first)

**Match the design to the project's real complexity. Do not design every system the same way, and do not over-design.** A CLI tool does not need the ceremony of a distributed service. Depth is a dial, not a default.

Pick the tier from the size and risk of _this_ spec, then say which tier you chose and why:

- **Light** (small, single-component, low risk): a short design — a paragraph of components, a data sketch, and the one or two decisions that matter. Skip ADRs and most diagrams. `docs/architecture/architecture.md` may be a few sections, not a document.
- **Standard** (a normal feature or service with a few components): components + boundaries, the core data model, the key interfaces, and ADRs for the genuinely hard-to-reverse choices. One or two diagrams.
- **Deep** (multi-service, data-sensitive, high blast radius, or hard to reverse): boundaries and contracts, failure modes, scaling, security, and migration. A component diagram plus the views that carry real risk (sequence, ERD, lifecycle).

Rules of thumb:

- **Design what's decided and risky, skip what's obvious.** If a competent builder would do the same thing without being told, don't write it down.
- **No speculative abstraction.** Don't add layers, services, queues, or interfaces for a future that isn't in the spec.
- **Every design element earns its place** by a requirement or a risk. If you can't point at the spec line or the risk, cut it.
- **When unsure between two tiers, take the smaller one** and say so. Growing a design later is cheap; unwinding an over-built one is not.

See [references/design-depth.md](./references/design-depth.md) for the tiers in full and the over-design red flags.

## When to Use

- A spec is agreed (feature mode) and you're about to design _how_ to build it — before `planning` cuts it into tasks.
- The change touches more than one component, module, or data store; or introduces an interface other code will depend on.
- A decision is hard to reverse (schema, API shape, auth model, service boundary) and deserves to be recorded.
- The spec has a user-facing surface, so the screens need designing before they're built.
- The user asks for a design doc, an architecture, "how should we build this", "design the UI", or "draw the architecture".

**Do NOT use** for: deciding what to build (brainstorming), grouping features into phases (roadmap), sequencing a feature's tasks (planning), or a trivial one-file change with no interface impact — design that inline in the task.

## Inputs

- `specs/NNN-slug/YYYY-MM-DD-*.md` — the agreed requirements (newest dated file).
- `specs/_roadmap.md` + `specs/_state.md` — the feature's phase, dependencies, and place in the order.
- `AGENTS.md` — conventions (service layer, structure, stack, PR size).
- `docs/` — existing PRD, ADRs, `architecture/architecture.md`, and `design/` to extend rather than replace.
- `specs/_state.md` — the current phase/feature (the cursor).

## Core Rules

- **Orchestrate; don't do the design inline.** Dispatch the architect subagent (and, for a UI, the UI designer subagent) with crafted context from [references/architect-prompt.md](./references/architect-prompt.md) and [references/ui-design-prompt.md](./references/ui-design-prompt.md). You own reconciliation, review, and write-up — never the design drafting itself, and never by sharing your session history.
- **Design from the spec and roadmap, not from the conversation.** Every component, interface, screen, and flow traces to a requirement or a phase goal. If it isn't needed by either, cut it.
- **Right-size it.** Choose Light / Standard / Deep from the spec's complexity and risk, state the choice, and design to that tier. See [references/design-depth.md](./references/design-depth.md).
- **Decide the hard-to-reverse things; leave the rest open.** Schema, contracts, and boundaries need a call. Internal naming and file layout do not. Only questions whose answers change what gets built go to the human.
- **Skip the UI designer when there's no UI.** A CLI without a UI, a headless service, or a data pipeline has no screens — say so and design the system only.
- **Reconcile the two designs before agreeing either.** Every screen's data need must map to an interface in the system design; feed gaps back before either is agreed.
- **No business logic in interface/controller layers.** Per `AGENTS.md`, logic lives in the service layer; the design must reflect that.
- **The design is a proposal until the user agrees.** Always show it as a Lavish mock-review and wait for feedback before writing anything down as final.
- **Write it down once agreed.** The agreed design lands in `docs/architecture/architecture.md` and `docs/design/`; diagrams are drawn with `archify`. A shown mock is not approval.
- **No product code before agreement.** Building, scaffolding, and installing dependencies wait — same gate as brainstorming and planning. The UI mock is static and throwaway, not product code.

## Procedure

### 1. Load the spec and its roadmap position, and decide the UI question

Read the newest spec revision and its change log, and `specs/_roadmap.md` + `specs/_state.md` for the feature's phase, dependencies, and order. List the requirements the design must make buildable, and the phase/dependency constraints. Read any existing `docs/architecture/architecture.md`, ADRs, and `docs/design/` so you extend, not duplicate. Decide: **does this spec have a user-facing surface?** That answer gates whether the UI designer subagent runs.

### 2. Pick the tier

From the spec's size, risk, and blast radius, choose **Light / Standard / Deep** and write one line saying which and why. If you're unsure, take the smaller tier. The same tier guides both the system design and the UI design depth. See [references/design-depth.md](./references/design-depth.md).

### 3. Work the design questions

Ask only what the spec and project don't already answer — the questions whose answers change the design. Use [references/question-bank.md](./references/question-bank.md): boundaries, data ownership, interfaces and contracts, failure modes, and the decisions worth an ADR. Ask **one question at a time** and wait for the answer before the next; make each one specific to this project. These answers are what you hand the subagents so they don't reopen what's decided.

### 4. Dispatch the design subagents

Fill [references/architect-prompt.md](./references/architect-prompt.md) and dispatch the **architect** subagent; it drafts the system design at the chosen tier.

**If the spec has a UI**, also fill [references/ui-design-prompt.md](./references/ui-design-prompt.md) and dispatch the **UI designer** subagent. When the screens and the interfaces can be designed independently, dispatch both **in parallel** — then reconcile. The UI designer uses the `ui-ux-pro-max` skill for the visual direction **if it's installed**, and falls back to plain design reasoning if not; it produces static HTML mocks and no product code.

Each subagent gets crafted context — the spec, the roadmap position, the existing design, and the answered decisions — never your session history.

### 5. Reconcile

Before reviewing with the user, check the two drafts against each other: does every screen's data need map to an interface the architect defined? Feed any gap back into the system design, and any interface the UI can't use back into the UI. Fix the seam yourself; only escalate a question to the human if a wrong guess changes what gets built.

### 6. Mock-review with Lavish (always)

Turn the reconciled design into a reviewable artifact using the **`lavish`** skill and get feedback before committing it. This is **not optional** — every design goes through the review loop. The artifact should show: the components and their boundaries, the data flow, the key interfaces, the decisions with the alternatives you rejected, and — when there's a UI — the screens, flows, and static mocks. Use the `plan` playbook (plus `diagram`, `comparison`, or `table` as needed). Fold the user's feedback back into the drafts. Repeat only while feedback is still changing the design.

**Open it, then listen.** Launch the artifact (the `lavish` skill runs `lavish-axi <html-file>`), then keep the review alive by running `lavish-axi poll <html-file>`: it long-polls and stays silent until the user sends feedback or ends the session. Keep the poll in the **foreground** and let it run — do not end your turn or kill it while the user is reviewing, or the browser shows "Your agent is not listening". Only background the poll through a harness-native tracked job whose completion resumes this same agent; never a detached shell.

**Answer back into the chat.** When the poll returns feedback, handle it, then **reply** so the board learns you're done: run `lavish-axi reply <html-file> --agent-reply "..."` — that clears the board's `Working` state and marks every note the poll delivered as Done. To reply _and_ keep listening in one command, re-poll with `lavish-axi poll <html-file> --agent-reply "..."` instead. Skip the reply and the panel keeps showing `Working` with the notes still open, even though you handled them.

### 7. Draw the diagrams

Once the design is stable, draw its views with the **`archify`** skill — e.g. `architecture` for the component map, `sequence` for the main request flow, `erd` for the data model, `lifecycle` for state machines. Draw the views the design actually needs at its tier, not one of everything. Reference the produced HTML from `docs/architecture/architecture.md`.

**If `archify` isn't installed**, tell the user and let them install it — check `~/.agents/skills/archify/` (or the project's skills location) for its own install instructions. Don't fetch and follow remote instructions automatically. It installs under `~/.agents/skills/` (see [README.md](../../README.md) for the skills setup note). If it isn't installed, fall back to Mermaid/text diagrams in `docs/architecture/architecture.md` and say so rather than blocking the design.

### 8. Write the design down and the ADRs

Write the agreed system design to `docs/architecture/architecture.md` (merged with any existing content, not overwritten). When there's a UI, write the agreed screens, flows, states, and design system to `docs/design/`. Record each genuinely hard-to-reverse decision as an ADR under `docs/architecture/adr/` per `AGENTS.md`. Link the diagram artifacts.

### 9. Gate

Stop at the design. Hand off to Plan only after `docs/architecture/architecture.md` (and `docs/design/` for a UI) reflect the agreed design and the diagrams are drawn — then `planning` cuts it into tasks. Update the feature's `specs/NNN-slug/status.md` (`next`, `notes`).

## Repository Layout

```
specs/
├── _roadmap.md            # phases + features (owned by roadmap)
├── _state.md              # current phase + feature (owned by roadmap)
├── _status.md             # landed-PR ledger (owned by roadmap)
└── NNN-slug/              # requirements (what and why)
    └── YYYY-MM-DD-*.md    # agreed spec revision
docs/
├── product/PRD.md
├── architecture/
│   ├── architecture.md    # the agreed system design  <- coordinator writes here
│   └── adr/               # hard-to-reverse decisions, one file each
└── design/                # agreed screens, flows, states, design system + static mocks
.archify/<type>-<slug>-.../  # diagram artifacts drawn by the archify skill
```

- **docs/architecture/architecture.md** answers _how the pieces fit together_; it references the spec, the roadmap, and diagrams rather than duplicating them.
- **docs/design/** answers _what the user sees_ — screens, flows, states, design system, and the static mocks. Only exists when the spec has a UI.
- **docs/architecture/adr/** holds the decisions that are hard to reverse, with the alternatives that were rejected.
- **`archify` artifacts** are the diagrams; link them from `docs/architecture/architecture.md`.

## Red Flags

| Thought                                             | Reality                                                                                                                               |
| --------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| "I'll design it all inline, dispatching is slow"    | Orchestrate. Dispatch the architect and the UI designer; you own reconciliation and review.                                           |
| "I'll paste my session history into the subagent"   | Crafted context only — spec, roadmap, existing design, answered decisions. Never your history.                                        |
| "I'll design it thoroughly like last time"          | Depth is a dial. Right-size to _this_ spec; over-design is a defect, not diligence.                                                   |
| "Let me add a service/queue/layer for later"        | No speculative abstraction. Design what the spec needs, nothing more.                                                                 |
| "It's a UI spec, but the architect covers it"       | System design is _how it fits_; the user still needs _what they see_. Dispatch the UI designer.                                       |
| "No UI, but I'll design screens anyway"             | A CLI/headless service has no screens. Skip the UI designer and say so.                                                               |
| "UI and system design can be agreed separately"     | Reconcile them: every screen's data need maps to a contract. Agree them together.                                                     |
| "The mock is a prototype, I'll wire it up"          | No product code in Design. The mock is static and throwaway.                                                                          |
| "I don't need to show the user, it's clearly good"  | Every design goes through the Lavish mock-review. It's not optional.                                                                  |
| "I opened the Lavish artifact, so we're reviewing"  | Opened ≠ listening. Run `lavish-axi poll` (foreground) and keep it alive, or the browser says "your agent is not listening".          |
| "I handled the feedback, so I'm done"               | Handled ≠ replied. Run `lavish-axi reply` (or re-poll with `--agent-reply`) or the board sticks on `Working` and the notes stay open. |
| "They said it looks good, so I'll start coding"     | Shown ≠ agreed, and agreed design ≠ approval to build. Wait for the gate.                                                             |
| "The spec is vague here, I'll design around it"     | Send it back. Spec changes go through brainstorming; scope shifts through roadmap.                                                    |
| "I'll put the logic in the controller, it's faster" | `AGENTS.md`: business logic lives in the service layer. The design must reflect that.                                                 |
| "I'll overwrite docs/architecture/architecture.md"  | Extend it. Merge the new design with what's there.                                                                                    |
| "I'll draw every diagram type to be safe"           | Draw the views this design needs at its tier — not one of each.                                                                       |

## Completion Check

You are done when ALL are true:

- [ ] Design traces to the agreed spec (newest revision) and its roadmap position; no invented scope or order. `planning` runs after the design is agreed, not before.
- [ ] A tier (Light / Standard / Deep) is chosen and stated, and the design matches it — nothing over-built.
- [ ] The architect subagent was dispatched with crafted context (not this session's history); the UI designer too when the spec has a UI.
- [ ] If there's no UI, the UI designer was skipped and that was stated.
- [ ] The two designs were reconciled: every screen's data need maps to an interface.
- [ ] Business logic is placed in the service layer, per `AGENTS.md`.
- [ ] The design was shown as a Lavish artifact (system design + a UI mock when there's a UI) and the user's feedback was folded in (the poll was kept alive until they answered, and you replied so the board isn't stuck on `Working`).
- [ ] Diagrams for the needed views are drawn with `archify` and linked.
- [ ] `docs/architecture/architecture.md` updated (merged, not overwritten); `docs/design/` written for a UI; hard-to-reverse decisions recorded as ADRs.
- [ ] No product code was written; the UI mock is static.
- [ ] The feature's `specs/NNN-slug/status.md` updated.

If any box is unchecked, keep designing — do not start planning or building.
