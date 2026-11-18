# Idea Refine

Where normal brainstorming starts from a request ("build X") and converges toward a spec, **idea
refine starts from a raw idea** — a half-formed thought, a "wouldn't it be cool if", a plan you're
not sure survives contact — and turns it into a sharp, actionable concept you can actually
brainstorm and plan from.

It is a **technique inside `brainstorming`, not a separate phase.** Run it when the idea is still
vague, then hand off to the normal conversation. Two movements, in order: **diverge**, then
**converge**. Collapsing them into one step is the mistake this exists to prevent.

## When to run it

- The request is still an **idea**, not a feature: no clear "what", or several unrelated whats.
- You're about to commit to a plan and want to **stress-test the assumptions** behind it.
- The user says **"ideate", "refine this idea", "stress-test my plan"**, or "here's a vague
  thought…".
- They want **more options** before picking one.

**Skip it** when the request is already sharp — go straight to the normal conversation. And it is
_not_ a substitute for the design: refined idea → still needs the spec, the approaches, and the
gate.

## Movement 1 — Diverge (expand before you narrow)

Goal: put real options on the table so the one you pick isn't the only one you saw. Generate wide;
judge later. Produce **5–10 candidate framings**, not one polished answer.

Techniques (use the two or three that fit; don't run the whole list):

- **Reframe the problem.** Restate the idea as the job it does ("people don't want a drill, they
  want a hole"). What problem is really being solved? Whose?
- **Analogies.** How is this like something in another domain? What does that domain do that we
  don't?
- **Constraints as fuel.** Flip the tightest constraint to an extreme ("what if it had to be free /
  instant / offline / one screen?") and see what falls out.
- **How might we…** Turn the goal into several open prompts, one per angle.
- **10× / 10%.** What if this had to be 10× bigger, or 10% the effort? Both reveal what's actually
  load-bearing.
- **Inversion.** How would we guarantee this _fails_? Every answer is a risk to design against.
- **The wildcard.** Include one option that seems too ambitious or too strange. It often moves the
  frame.

Keep them concrete enough to picture. An option you can't sketch isn't an option yet.

## Movement 2 — Converge (choose, with a reason)

Goal: reduce to **two or three viable directions**, each with what it trades off, and a pick.

- **Cluster.** Group the candidates that are really the same idea wearing different clothes.
- **Score against intent.** For each, weigh it against the _why_, the goals, the non-goals, and the
  anti-goals. Drop anything that works against why it's wanted — including your favorites.
- **Kill criteria.** Name the specific thing that would make an option wrong (too costly, ruled out
  by existing work, unbuildable). Kill on criteria, not on vibes.
- **Pick one, say why.** Recommend a direction with its trade-offs; note what it gives up. The
  human decides direction (per `brainstorming`), so a proposal they can correct beats a menu.

## Stress-test the assumptions

Before the refined idea goes into the normal conversation, surface what has to be true for it to
work:

- **Assumption inventory.** List the beliefs the idea rests on, most load-bearing first. Mark each
  as known, assumed, or guess.
- **Pre-mortem.** It's six months on and this failed — what happened? The top reasons are the risks.
- **Cheapest test.** For each load-bearing assumption, name the smallest thing that would confirm
  or kill it (a question to the user, a spike, a search, a look at the code).
- **Reversibility.** Which decisions are hard to undo (schema, API shape, a public commitment)?
  Flag those for the design; the rest are safe to change later.

Stress-testing is **adversarial on purpose** — it's for finding the crack while it's cheap, not for
reassuring. A plan that survives this is worth committing to; one that doesn't just saved you a
build.

## Output — feed it back to brainstorming

Idea refine produces a hand-off, not a document:

- **The sharpened concept** in one or two sentences — the "what", sharp enough to brainstorm from.
- **The shortlist** (two or three directions) with their trade-offs and your pick.
- **The load-bearing assumptions** and the cheapest test for each.
- **The hard-to-reverse decisions** to carry into the design.

Then continue in the normal flow: the conversation, the approaches, the written design, the gate.
Never let a refined idea skip the spec.
