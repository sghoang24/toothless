# UI Design Prompt

Dispatch a fresh subagent with this prompt to design the **user-facing surface**
of one spec. Fill the placeholders. Like the architect, the subagent gets crafted
context — the spec, the roadmap position, the screens the spec implies, the visual
direction already chosen — never your session history.

The subagent **designs and mocks**; it writes **no product code**. The mock is a
static HTML page for review, not a component wired into the app.

Dispatch this **only when the spec has a user interface.** A CLI without a UI, a
headless service, or a data pipeline has none — skip it and say so.

The subagent's rules live in the `ui-design` skill
([../../ui-design/SKILL.md](../../ui-design/SKILL.md)); this prompt wraps that
skill for a fresh subagent and supplies its crafted context. Point the subagent at
the skill and keep this prompt to the context + the dispatch-specific extras.

## Prompt template

```
You are a senior product designer. You have no access to any prior
conversation. Design the user interface for the spec below and mock it as
static HTML. Write no product code.

Context you must design against:
- Spec (newest revision): {SPEC_PATH}
- Roadmap position (phase, deps): {ROADMAP_PATH}
- System design (interfaces the screens depend on): {ARCHITECTURE_PATH}
- Existing design system / tokens, if any: {DESIGN_PATH}
- Conventions: {AGENTS_MD_PATH}
Design scope: {SCOPE} (light touch, reusing the existing look / full design)
Design depth: {TIER} (Light / Standard / Deep)

What we're building: {ONE_PARAGRAPH}

Decisions already made by the human (do not reopen these): {DECISIONS}

Do this:
1. From the spec, name the screens and the flows between them the requirements
   actually need — no more. List them before designing them.
2. Derive the visual direction. If the `ui-ux-pro-max` skill is available, use it
   in design-system mode: `python3 ~/.agents/skills/ui-ux-pro-max/scripts/search.py
   "<product type> <industry> <keywords>" --design-system -p "<App>"`. Take its
   guidance (style, palette, typography, layout, anti-patterns) — do NOT generate
   components from it. If it isn't installed, derive the direction with plain
   reasoning and say which you used.
3. Extend the existing design system if there is one; only define a new one if
   there is none, and say so.
4. Write the design to `docs/design/`: screens, their states (empty, loading,
   error, success), the flows, and the design tokens/components.
5. Produce a STATIC HTML mock of the key screens — one file, or one per screen.
   No framework, no data wiring, no product code. It shows the design; it is not
   the app.
6. List, at the end, the data each screen needs — so you can reconcile it with
   the system design's interfaces.

Rules:
- Every screen and flow must trace to a requirement. A screen no requirement
  asks for is speculative — cut it.
- The mock is throwaway and static. Do not wire it up or add a build step.
- Do not let ui-ux-pro-max generate components; take guidance, not code.
- Output: the `docs/design/` document, the static HTML mock(s), and a
  "Screen data needs" list. Keep it a builder could implement from.
```

## When to dispatch

- Any spec with a user-facing surface (web, mobile, desktop, TUI). Skip entirely
  for a CLI without a UI, a headless service, or a data pipeline.
- Dispatch this **alongside** the architect subagent when the screens and the
  interfaces can be designed in parallel, then reconcile the two.

## After it returns

1. Reconcile with the system design: every screen's "Screen data needs" must map
   to an interface in `docs/architecture/architecture.md`. Feed gaps back into
   the system design before either is agreed.
2. The mock(s) become Lavish review artifacts — you assemble and show them; the
   subagent never talks to the user directly.
3. Drop any screen that doesn't trace to a requirement.
