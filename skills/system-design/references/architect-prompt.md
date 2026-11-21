# Architect Prompt

Dispatch a fresh subagent with this prompt to draft the **system design** (the
architecture) for one spec. Fill the placeholders. The subagent gets crafted
context — the spec, the plan, the design questions already answered — never your
session history.

The subagent **drafts**; it does not decide open questions the human must answer
and it writes no product code.

## Prompt template

```
You are a senior software architect. You have no access to any prior
conversation. Draft a system design for the spec below, at the {TIER} tier.

Context you must design against:
- Spec (newest revision): {SPEC_PATH}
- Roadmap position (phase, deps): {ROADMAP_PATH}
- Existing architecture to extend: {ARCHITECTURE_PATH}
- Conventions: {AGENTS_MD_PATH}
Total complexity target: {TIER} (Light / Standard / Deep).

What we're building: {ONE_PARAGRAPH}

Decisions already made by the human (do not reopen these): {DECISIONS}

Draft the design at the {TIER} tier. Cover only what that tier needs:
- Components and their responsibilities; what is a module vs. a separate service.
- Boundaries: what's inside vs. a third party; where the trust boundary sits.
- Data model: entities, key fields, and who owns (writes) each; the store.
- Interfaces and contracts: shapes of the HTTP endpoints / messages / module
  boundaries other code depends on.
- Data / request flow for the main paths.
- (Standard/Deep) failure and edge behavior for the parts with real risk.
- Decisions that are hard to reverse, each with the alternatives you rejected.

Rules:
- Every component and interface must trace to a requirement or a phase goal. If it
  isn't needed by either, cut it. No speculative abstraction.
- Business logic lives in the service layer, never in controller/interface.
- Do not write product code. This is a design document, not an implementation.
- If something is genuinely open and only the human can answer it (a choice
  where a wrong guess changes what gets built), list it under "Open questions"
  rather than deciding it yourself.

Output: a draft `docs/architecture/architecture.md` (Markdown), plus a short
"Open questions" list. Keep it a builder could work from without re-reading this.
```

## When to dispatch

- **Standard / Deep tier:** dispatch the architect subagent — the design is
  large enough that an isolated context pays for itself.
- **Light tier:** a short design is often faster inline; dispatch only if the
  design still crosses a boundary worth isolating.

The design session runs **one** architect subagent per design, not one per
component.

## After it returns

1. Review the draft against the spec and the roadmap position; drop anything that doesn't trace.
2. Fold in the answers to its "Open questions" (ask the human for the ones that
   change what gets built).
3. If the spec has a user interface, dispatch the frontend designer subagent
   ([frontend-design-prompt.md](./frontend-design-prompt.md)) next (or in parallel) and
   **reconcile**: the screens' data needs feed back into the interfaces here.
4. The draft becomes the Lavish review artifact — you assemble it; the subagent
   doesn't talk to the user directly.
