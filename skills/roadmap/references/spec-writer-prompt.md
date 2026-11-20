# Spec Writer Prompt

Dispatch **one** subagent **per phase** to write the full `spec.md` for every feature in that
phase. The roadmap has already sliced the features and the human approved it, so this is
**derivation, not discovery**: the subagent turns approved feature rows + PRD requirements into
testable acceptance criteria. It asks no questions, invents no requirements, and writes no code.

## Why one subagent per phase

- **Phase-local consistency is free.** One context holds every feature of the phase, so shared
  surfaces (a `Makefile` target, a shared entity, an interface another feature consumes) are
  specced once and referenced, not re-invented per feature.
- **Cost is one dispatch per phase**, not one per feature. Phases are independent, so they
  dispatch **in parallel** (see the coordinator notes at the bottom).
- **A reject re-dispatches one phase**, not the whole roadmap.

Do **not** dispatch one subagent for all phases: the phase is the unit the human reviews, and a
single context spanning every phase drifts toward inventing detail for later work.

## The prompt template

Fill every `{PLACEHOLDER}`. Never paste your session history; the subagent gets crafted context only.

```
You are a senior engineer writing feature specifications. You have no access to any prior
conversation. You will NOT ask questions — everything you need is below, and anything genuinely
undecided goes into that feature's "Open questions" section.

## Context

- PRD (approved): {PRD_PATH}
- Roadmap (approved): {ROADMAP_PATH}
- Architecture, if it exists: {ARCHITECTURE_PATH}
- Conventions: {AGENTS_MD_PATH}
- Specs already written for earlier phases (for consistency and shared interfaces): {PRIOR_SPECS}
- Existing code relevant to this phase, if any: {CODE_CONTEXT}

## Your phase

Phase {PHASE_NUMBER}: {PHASE_NAME}
Goal: {PHASE_GOAL}
Exit criteria:
{PHASE_EXIT_CRITERIA}

The features of this phase, exactly as the approved roadmap defines them — these IDs, slugs,
`covers`, and `depends_on` are FIXED. Do not renumber, rename, add, or drop a feature:

{FEATURE_ROWS_VERBATIM}

Their one-line descriptions:
{FEATURE_DESCRIPTIONS}

## What to write

For EACH feature above, write `specs/{ID}/spec.md` using exactly this skeleton:

---
feature: {ID}
phase: {PHASE_NUMBER}
status: spec # spec | approved — the coordinator sets `approved` after human review
revision: {TODAY} # YYYY-MM-DD
covers: {the roadmap's covers, verbatim}
depends_on: {the roadmap's depends_on, verbatim}
---

# {Feature name} — spec

Revision: {TODAY} (initial)

## Intent and why
<2-4 sentences: what the user can do after this ships, and why the phase needs it.>

## Covers
<One line per requirement ID you were given, quoting the PRD's own wording for it.>

## Goals / Non-goals
<Goals: what this feature makes true. Non-goals: adjacent things it deliberately does not do.>

## Acceptance criteria
- **AC-1**: Given <state>, when <action>, then <observable result>.
- **AC-2**: ...

## AC → requirement mapping
| AC   | Requirement | Phase exit criterion (if it serves one) |
| ---- | ----------- | --------------------------------------- |
| AC-1 | {FR-n}      | {criterion, or —}                       |

## Open questions
<Only questions you could not answer from the PRD, roadmap, or code. State what you assumed
meanwhile, so the reviewer can correct it. Write "None." when there are none.>

## Left to the builder
<What the implementer must decide inside the tasks — file layout, local naming. Never schema,
API shape, or a boundary: those belong to the design phase.>

## Rules — follow all of them

1. **No invention.** Every acceptance criterion traces to a `FR-n` / `NFR-n` in the PRD, or to the
   phase's exit criteria. If you cannot point at one, the criterion does not belong in the spec —
   put the gap in "Open questions" instead.
2. **Every AC appears in the AC → requirement mapping table.** An unmapped AC is a defect.
3. **Testable by a stranger.** Each AC reads as Given/When/Then with an observable result —
   someone who never read this conversation must be able to write the test from it alone.
4. **At most 8 acceptance criteria per feature.** More than that means the feature should have
   been split. Do NOT split it yourself: write what you can and report it back so the roadmap can
   be replanned.
5. **Do not design.** No schema, no endpoint shapes, no component boundaries, no ADRs. If a
   criterion cannot be met without a design decision, say so in "Left to the builder" or
   "Open questions".
6. **Do not scope-creep the feature.** Only what the roadmap's one-line description and `covers`
   support. Anything else you noticed goes in your report, not into the spec.
7. **Respect the phase.** Do not depend on a feature from a later phase. Do not re-spec a feature
   from an earlier phase; reference its spec instead.
8. **No product code.** Markdown only.
9. **Write the file.** Every spec must exist on disk at `specs/{ID}/spec.md`. A spec described
   only in your reply is not written.

## Report back

Reply with a short report (the specs themselves live in the files):

- **Phase:** {PHASE_NUMBER} — {PHASE_NAME}
- **Status:** DONE | DONE_WITH_CONCERNS | BLOCKED
- **Files written:** one line per `specs/{ID}/spec.md`
- **AC → requirement coverage:** which `FR-n`/`NFR-n` each feature's ACs cover, and any
  requirement in this phase's features left with no AC
- **Over-sized features** (> 8 AC) or features that cannot be specced from the roadmap as cut
- **Open questions** raised across the phase, most important first
- **Concerns** that are not blockers
```

## Coordinator checks after the subagent returns

Treat the report as **claims, not evidence** (`fight`):

- [ ] Every feature of the phase has `specs/{ID}/spec.md` on disk, with the frontmatter intact
      (`feature`, `phase`, `covers`, `depends_on` matching `_roadmap.md` **verbatim**).
- [ ] Every `AC-n` appears in the AC → requirement mapping table, and every mapped requirement is
      in that feature's `covers`.
- [ ] Every requirement in the phase's `covers` is served by at least one AC.
- [ ] No feature has more than 8 ACs; if one does, it is a roadmap defect — send it to the human,
      do not let the subagent split it.
- [ ] Cross-feature consistency: a shared surface (a `make` target, a shared entity, an interface)
      is defined once and referenced, not defined differently in two specs.
- [ ] No schema, API shape, or boundary was decided in a spec — that is `system-design`'s.

A failing check sends **that phase** back to the same subagent with the specific finding, once.
If it fails again, fix the specs yourself in the coordinator session and note the call, or stop
and ask the human.

## Re-dispatch, not rewrite

When the human rejects one spec in a phase, re-dispatch the **same phase** with the human's
feedback verbatim plus the list of specs that were already accepted, and tell it to keep the
accepted ones untouched. Never re-dispatch every phase for one feature's problem.

## When to dispatch, and when not to

- **First spec for a phase:** step 6a of Create mode — one writer per phase, in parallel.
- **A replan added or recut a feature:** that phase's writer only, for the new feature.
- **An approved spec must change** (Revise mode, feature `todo`/`doing`): do **not** re-dispatch the
  phase. `roadmap` applies the delta itself from [change-requests.md](./change-requests.md) — a
  revision edits a document you already hold rather than drafting a new one. Re-dispatch only when
  the change rewrites most of the feature's criteria.
- **A `done`/shipped feature:** never. It gets a follow-up feature, whose spec is then dispatched
  normally.
