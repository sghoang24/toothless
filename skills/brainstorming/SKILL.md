---
name: brainstorming
description: "Use this before building anything new or changing behavior whenever the request is vague, underspecified, or has more than one reasonable reading: a new product or project, a feature, a component, a behavior change. Turns the request into an agreed, testable scope through one-question-at-a-time conversation, a comparison of approaches, and a written design that the user approves before any code exists. Project mode produces docs/product/PRD.md (suggested next step: roadmap); feature mode produces a spec in specs/NNN-slug/ (suggested next step: system-design, then planning). Works on its own: the next step is offered, never forced. Also use to sharpen a raw idea or stress-test a plan (triggers: ideate, refine this idea, stress-test my plan). Skip for pure questions, explaining existing code, and fully specified trivial fixes."
argument-hint: "Describe what you want to build or change (one sentence is enough)."
---

# Brainstorming

You are very good at building. You build what you believe your human partner wants, and that belief is usually thinner than it feels. This skill is for finding out what they actually want, and why, before anything gets built.

People often haven't finished working out what they want. Good questions help them think it through. When you understand the why, you make the hundred decisions they never mention the way they would have made them.

This skill is the entry point of the engineering workflow. Workflow position: usually the entry point of `brainstorming` → `roadmap` → `system-design` → `planning` → implement, but it also works on its own. It never depends on the later skills being present.

## Step 0: Pick the mode

| Mode             | When                                                                                                                  | Output                             | Suggested next               |
| ---------------- | --------------------------------------------------------------------------------------------------------------------- | ---------------------------------- | ---------------------------- |
| **Project**      | No `docs/product/PRD.md` yet, or the request is a whole product/system, or a new capability spanning several features | `docs/product/PRD.md`              | `roadmap`                    |
| **Feature**      | A PRD exists and the request is one feature or change                                                                 | `specs/NNN-slug/<date>-<topic>.md` | `system-design` → `planning` |
| **Small change** | Touches no public interface and no data model, fits in one implement loop                                             | A short design in chat             | `planning`                   |

If it is unclear, state which mode you chose and why as a one-line proposal, then continue. Do not ask the user to pick a mode.

Mode rules that prevent mix-ups:

- **Project mode never creates `specs/NNN-slug/` folders.** The `roadmap` skill creates them after slicing the PRD into features.
- **Feature mode** reuses `specs/NNN-slug/` if `roadmap` already created it (a `status.md` is there). Otherwise it creates the folder with the next free number.
- If the feature is already in `_roadmap.md` and the PRD already makes it clear, say so and offer to go straight to `system-design` (then `planning`).

## Standalone use

You are standalone unless the user is clearly running the pipeline (they said so, `specs/_state.md` or `_roadmap.md` exists, or `AGENTS.md` defines the workflow). Standalone means:

- You may be used just to think something through. The deliverable is then the agreed scope in chat, or one document saved where the user wants it. Do not force the PRD, `specs/`, or any folder convention on a user who did not ask for it.
- The mode table and the paths in this skill are defaults for the pipeline, not requirements. Outside it, pick a sensible location, say where, and let the user's preference override.
- The design gate still applies if the user wants to build; if they only wanted to think, there is nothing to gate.
- The next skill is offered in one line at the end, never invoked on your own.

## Idea Mode (for a raw idea)

Normal brainstorming starts from a **request** ("build X") and converges toward a spec. When what you have is a **raw idea** — a half-formed thought, a "wouldn't it be cool if", a plan you're not sure survives — run **idea mode first**: it sharpens the idea through structured divergent then convergent thinking, and stress-tests its assumptions, before the normal conversation begins. See [references/idea-refine.md](./references/idea-refine.md).

Run it when the request is still an idea, when the user says **"ideate", "refine this idea", "stress-test my plan"**, or wants more options before choosing. It is a **technique inside this skill, not a separate phase** — diverge, then converge, then hand the sharpened concept back to the normal flow. Once the concept is sharp, continue below as usual. A refined idea does **not** skip the spec or the gate.

## Look Before You Ask

In an existing project, or when they've given you context (a spec, a plan, a handoff, files they pointed at, earlier conversation), dig in before your first message. Read what they referred to and the code or material it connects to, and keep reading until you can explain how the relevant parts work today and what this change would touch. Never ask what the project or their context already answers, or reopen what they've already decided.

## When to Use

- The user wants to create a feature, build a component, add functionality, or modify anything new.
- The request is ambiguous, underspecified, or has more than one reasonable interpretation.
- The user has a **raw idea** to sharpen, or wants to **stress-test a plan's assumptions** before committing (see Idea Mode above).
- You are tempted to ask "should I just start coding?" — stop and run this instead.
- The user has a goal but no clear acceptance criteria.

**Do NOT use** for: answering questions, reading/explaining existing code, or trivial unambiguous fixes (typo, one-line change the user fully specified).

## Core Rules

- **No product code before an approved design.** Creating documentation folders and files (`docs/`, `specs/`) is fine; source code, scaffolding, dependency installs, and project creation are not.
- **Ask only when the answer changes what gets built** and neither the request nor the project answers it. Otherwise state your call plus your reason as a proposal they can correct, and keep moving. Process choices (where files go, what to read next, how to present) are always your calls: make them and say so.
- **One question at a time, and wait.** Ask a single question, then stop and wait for the answer before the next. **Never dump a list of questions** as plain text. When a decision has nameable choices, prefer concrete options over an open-ended prompt.
- **Explore, don't guess.** Verify what already exists in the codebase before proposing anything.
- **The user decides.** Present options and trade-offs; the human chooses direction.

## The conversation

Ask a question only when the answer would change what gets built and neither the request nor the project answers it. Otherwise, state your call and your reason as a proposal they can correct, and move the work forward. Process steps are always your calls: writing things up, where files go, looking further, how to show something. Do them and say so.

**Ask one question at a time, and wait for the answer before the next.** One question, one message — never a wall of questions. When the harness can render a structured question (clickable options), present the single question that way with concrete choices; otherwise make it one clear line they can answer plainly. Build each question on their last answer or on what you found in the code. Use plain language, pitched just a little above where your human partner is. Match their vocabulary; never use process jargon. You never need to quote this skill to your human partner.

**Ground to cover**. This is territory, not a script. Never read these out as questions:

- what kind of thing this is, and what makes it special
- why now, and what success looks like to them
- goals, non-goals, and anti-goals (what would make this a failure even if it technically works)
- prior art: what they've seen elsewhere, loved or hated
- current state: what exists now, what they've tried
- the details they care about

**Make every question specific** to their project and what they've told you. Ask Socratically, the way a sharp colleague would: build each question on their last answer or on something you found, so it helps them think through a consequence or a case they may have missed ("Voting closes before the meeting, and two members joined this month. Does someone who joins mid-round get a vote?"). When the reason isn't already plain, say what you're hoping to learn or what the answer would change. A proposal they can correct, plus your reason for it, often beats a question.

**Do reconnaissance**. Once you have a bit of grounding and looking further would help, go look: deeper into their codebase or project, their own files and data, or the web. Say what you looked for and what you found.

**Lay out the approaches**. Once you understand what they want, and before you write anything up, put the ways of building it in a message of its own. Come up with several real alternatives and drop the weak ones, including any that work against why they want it and any the existing work rules out. Sketch the two or three that remain concretely enough to picture, grounded in what the project actually does, with what each trades off. Recommend one and say why, then ask which way they want to go; if they've handed you the how, say you'll go with your pick instead. When there's genuinely one sensible way, say so in that message and explain why the others don't fit; the design comes in the next one. This is for how it gets built; for what they want, get them describing first.

**Parallelize the looking, not the asking**. When research, exploration, or code reading is needed, do it in parallel — several reads and searches at once, or a background subagent — rather than blocking on it. But questions still go to your human partner **one at a time**: ask one, wait for the answer, then the next. If a later question depends on an earlier answer, that dependence _is_ the ordering — ask it after the first is answered, never both up front.

## The Written Design

Write a document a talented builder could plan from without going back to the user. Templates are in references/template.md.

**Project mode → PRD** (`docs/product/PRD.md`): purpose, goals, non-goals, anti-goals, users and scenarios, functional requirements with IDs (`FR-1`...), non-functional requirements (`NFR-1`...), constraints and assumptions, risks and open questions, success metrics. IDs matter because `roadmap` maps features to them and checks coverage later. If a PRD exists, extend it and add a revision-log entry; never overwrite or delete content without asking.

**Feature mode → spec** (`specs/NNN-slug/YYYY-MM-DD-<topic>.md`): intent and why, which PRD requirements it covers, goals/non-goals, approaches considered, decisions as the user made them, your calls (marked), **testable acceptance criteria**, risks and open questions, what is left to the builder. Each revision is a new dated file (add `-r2` for a second revision on the same day), a complete stand-alone document with a change log at the top. Never overwrite an old revision.

**Small change:** the same points in a line or two each, shown in chat, no builder check.

The change log is a pointer, not a duplicate — the body below it is the full,
current requirements. Unchanged requirements are still written out in full, but
planning won't re-plan them (see below).

Otherwise, pick a sensible place and say where. Your human partner's
preferences override both.

**Builder check** (for a project): dispatch a fresh subagent with
`builder-check-prompt.md` in this directory. When you tell your human
partner you're writing it up, say a reviewer will read it first and you
may come back once with a few questions. Don't hand over the document
until the check is back. Sort what the builder asks into three piles:

- **Answered already** by the conversation or the project: put the answer
  in the document.
- **Minor:** a sensible builder could settle it without changing what gets
  built. Settle it yourself and mark it in the document as your call.
- **Theirs:** only your human partner can answer it, and the answer changes
  what gets built.

Bring the "theirs" pile in one message, most important first. Update the
document with their answers, then hand it over with a short list of the
calls you made so they can check those while reading. That's the only
round of questions the check produces. Without a subagent tool, read the
document as that builder yourself.

<HARD-GATE>
Nothing is built until the user approves the written design (PRD, spec, or short design).

Approval means an explicit yes to the document you showed, not "sounds good" to the idea. If they ask for changes, revise and ask again; for features that means a new dated revision.

Building includes writing product code, scaffolding, installing dependencies, creating projects, and invoking an implementation skill. Reading and exploring are fine. If the user opted out of questions, this skill and its gate are done. A spike they said yes to may be built, stays labeled throwaway, and keeping it is a new request that returns to this gate.
</HARD-GATE>

## Handoff

After approval:

1. If `specs/_state.md` or the feature's `status.md` exists, update it (`next: design` for features). If not, skip.
2. **In the pipeline:** project mode → invoke `roadmap`; feature mode → invoke `system-design` (which hands off to `planning`); small change → invoke `planning` directly. Invoke only that skill, and never an implementation skill from here.
3. **Standalone, or the next skill is not installed:** do not invoke anything. Say in one line what the natural next step is ("Next would be `roadmap` to slice this into phases; want me to run it?") and stop. If they say no or want to build by hand, the approved document is the deliverable.

## Red Flags

| Thought                                                   | Reality                                                                                                                                     |
| --------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| "I'll offer a menu of what they might want"               | Options steer. Ask them to describe it first.                                                                                               |
| "I'll fill in sensible defaults"                          | Good. Say them out loud. A default they never heard is one they couldn't reject.                                                            |
| "I've answered their questions, so I'll write the doc"    | Lay out the approaches first, in a message of their own (or, when nothing was open from the start, in your first message's short design).   |
| "I should explain the process first"                      | Ask your question. The process shows itself.                                                                                                |
| "They said 'sounds good' to the idea"                     | Approval covers what you showed them. A design doc you haven't written isn't approved.                                                      |
| "The spike works, I'll keep building on it"               | Keeping it is a new request. Back through the gate.                                                                                         |
| "The idea's clear enough, skip straight to the spec"      | A raw idea that skips converge-then-stress-test is where the wrong build starts. Run idea mode first — a refined idea still needs the gate. |
| "Let me list all my questions so they can answer at once" | A wall of questions is noise and gets skimmed. Ask one, wait for the answer, then the next.                                                 |

## Optional integrations

- **fight:** if active, it coordinates and calls this skill; on your own, apply the same discipline.
- **Visual companion (`lavish`):** only when the user asks for a mockup or "show me". Run `lavish-axi <html-file>`, then `lavish-axi poll <html-file>` in the foreground until they send feedback or end the session, and answer with `lavish-axi reply <html-file> --agent-reply "..."`. A shown mockup is not approval; the gate still holds.
- **AGENTS.md / subagents:** if absent, follow the same rules directly and do the builder check yourself.
