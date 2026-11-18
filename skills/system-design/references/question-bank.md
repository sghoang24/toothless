# Design question bank

Ask **only** the questions the spec and project don't already answer, and only
those whose answer changes the design. Don't read these out as a script; pick the
few that matter for this spec and make each one specific to it.

The rule of thumb: if a competent builder would make the same call without being
told, it's not a design question — decide it and say so. Reserve questions for the
choices that are genuinely open and consequential.

## Contents

1. [Boundaries](#boundaries)
2. [Data ownership](#data-ownership)
3. [Interfaces and contracts](#interfaces-and-contracts)
4. [Flow and failure](#flow-and-failure)
5. [Decisions worth an ADR](#decisions-worth-an-adr)
6. [Right-sizing the questions themselves](#right-sizing-the-questions-themselves)

## Boundaries

What is one component and what is a separate system? What's inside the app vs. a
service? What's a third party?

- Where does this spec's work live — inside the existing app, or a new module/service? What forces that split?
- What already exists that this must reuse (auth, datastore, queue, shared lib)? What must it not duplicate?
- What is a trust boundary here — what's public, what's authenticated, what's internal only?
- Is there an existing component this belongs to, or a new one? Why?

## Data ownership

For every entity the design touches: **who writes it, who reads it, and where it
lives.** Shared write access is the thing to catch here.

- Which entities does this spec introduce or change? Who is their owner (single writer)?
- Does any other component already read or write these? If we share, how do we keep one writer?
- What's the store (relational, document, cache, object storage, queue)? Why that one?
- What's the schema shape at a level callers depend on (keys, required fields, relationships)?
- Is there existing data to migrate, or is this greenfield? If migrating, is it online and reversible?

## Interfaces and contracts

Anything other code or another service will depend on. Give shapes, not just names.

- What interfaces does this expose (endpoints, messages, module APIs, events)? For each, who calls it?
- What's the request/response (or event) shape? Which fields are required and which are optional?
- Are callers external? If so, what are the versioning and back-compat expectations?
- Is the interface sync or async? If async, what's the delivery/ordering guarantee?
- Per `AGENTS.md`, is the business logic in the service layer with a thin interface on top?

## Flow and failure

The main paths and what happens when they break. Depth here scales with risk.

- What is the primary flow, step by step, from entry to result?
- Where can it fail, and what should happen then — retry, fail fast, degrade, compensate?
- Which steps must be idempotent (retried or replayed)? How is that achieved?
- Is there a long-running or async step (job, webhook, batch)? How is progress/state tracked?
- What are the important edge cases the spec calls out, and how does the design handle each?

## Decisions worth an ADR

Only **hard-to-reverse** choices get an ADR. Ask yourself for each decision:
_if we're wrong, how expensive is it to undo?_

- Schema / storage choice: reversing it means a migration.
- API shape or protocol: reversing it means breaking callers.
- Auth / trust model: reversing it touches security.
- Service boundary: reversing it means moving code and data across a boundary.
- Vendor / third-party dependency: reversing it means a rewrite of an integration.

Everything else (naming, file layout, internal helper structure) is a reversible
call — make it, note it in a line, and move on. No ADR.

## Right-sizing the questions themselves

- **Light tier**: ask maybe 1–3 questions, only about the one decision that's hard to reverse.
- **Standard tier**: ask about boundaries, data ownership, and the key interfaces.
- **Deep tier**: also ask about the risky flows, failure modes, and migration.
- If you're asking more than a handful of questions, you may be over-designing — trim to the ones whose answer changes the design.
