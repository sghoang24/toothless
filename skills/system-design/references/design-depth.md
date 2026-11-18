# Right-sizing the design

The single most common design failure is not a missing box — it is designing a
small system like a large one. Depth is a **dial**, set by the spec's real
complexity, not a default you always turn to maximum.

## Pick the tier

Read the spec and ask: how many components? how much data ownership? how many
interfaces others depend on? how expensive to reverse a wrong call? Then choose:

| Tier         | Choose when                                                                  | The design covers                                                             | Skip                                                    |
| ------------ | ---------------------------------------------------------------------------- | ----------------------------------------------------------------------------- | ------------------------------------------------------- |
| **Light**    | One component; small change; low risk; easy to reverse                       | Components in a paragraph, a data sketch, the 1–2 decisions that matter       | ADRs (usually), most diagrams, scaling/failure sections |
| **Standard** | A feature/service with a few components; some shared data or interfaces      | Components + boundaries, core data model, key interfaces, ADRs for hard calls | Deep scaling, multi-region, exhaustive state machines   |
| **Deep**     | Multi-service; data-sensitive; high blast radius; hard to reverse; regulated | Boundaries + contracts, failure modes, scaling, security, migration, ADRs     | Nothing with real risk — but still cut anything obvious |

State your tier in one line when you start drafting: _which_ and _why_.
If you're between two tiers, **take the smaller one** and say so.

## What each tier does and does not need

**Light**

- A paragraph naming the piece(s) and their responsibility.
- The data it touches, in a line or a small sketch.
- The one decision that would be annoying to undo.
- No ADR folder noise; no component diagram unless it genuinely clarifies.

**Standard**

- Components and their responsibilities; what is a module vs. a separate service.
- The core data model and who owns each entity.
- The interfaces between parts (shapes, contracts), at least in outline.
- An ADR for each hard-to-reverse decision (schema, API shape, auth).
- One or two diagrams (usually `architecture`, plus `sequence` or `erd`).

**Deep**

- Everything in Standard, plus:
- Failure modes and what happens on partial failure.
- Scaling and load characteristics that shape the design.
- Security/privacy boundaries and trust assumptions.
- Migration/compat plan if replacing something live.
- The diagram views that carry real risk (`sequence`, `erd`, `lifecycle`).

## Over-design red flags

| Signal                                                              | Why it's wrong                                                    |
| ------------------------------------------------------------------- | ----------------------------------------------------------------- |
| Adding a service, queue, cache, or layer "because we might need it" | No requirement asks for it. Speculative = cut.                    |
| Documenting an obvious component the builder would add anyway       | Wastes the reader's attention; design the non-obvious.            |
| Full multi-region/scaling section for a single-user internal tool   | Ceremony with no risk behind it.                                  |
| An ADR for a reversible, low-impact choice                          | ADRs are for hard-to-reverse calls only.                          |
| Designing all three tiers' worth "to be safe"                       | Over-built designs are unwound far more than they're used.        |
| Drawing every diagram type                                          | Draw the views the design needs; a diagram nobody reads is noise. |

## Under-design red flags

| Signal                                                    | Why it's wrong                                         |
| --------------------------------------------------------- | ------------------------------------------------------ |
| Two parts share a table and nobody said who writes to it  | Ownership is exactly what design is for.               |
| An interface other code will call, left unspecified       | Callers will each invent their own shape.              |
| A schema/API/auth decision made implicitly "while coding" | Hard to reverse; decide it here, not mid-PR.           |
| The design can't say what happens when a dependency fails | Failure behavior is design, not implementation detail. |

## The test

For every element in the draft, answer: **which spec line or which risk puts
this here?** If you can't answer, cut it. If the design is mostly elements you
can't justify, you're one tier too high — drop down.
