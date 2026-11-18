# `docs/architecture/architecture.md` template

One design document per project, **extended** over time — never overwritten.
Keep only the sections the chosen tier needs; delete the rest. Merge new designs
into the existing file rather than replacing it.

```markdown
# Architecture

## Context

What this system is and the problem it solves, in a few lines. Link the PRD
(`docs/product/PRD.md`) and the specs it implements. Note the current revision /
date so readers know how fresh it is.

## Components

The parts of the system and each one's responsibility. Name them the way the code
does. Say what is a module inside the app vs. a separately deployed service.

- **<Component>** — <responsibility>. <is it in-process / a service / a datastore>
- ...

## Boundaries

What is inside this system and what is a separate system or third party. Where the
trust boundary sits (what's authenticated, what's public).

## Data model

The entities, their key fields, and **who owns (writes) each**. Note the store
(DB, cache, bucket, queue). At Standard/Deep, an `erd` diagram can carry this;
link it instead of re-listing every column.

## Interfaces and contracts

Each interface other code or another service depends on: HTTP endpoints, message
shapes, function/module boundaries, events. Give the shape, not just the name.
Note versioning/back-compat expectations if callers are external.

## Data / request flow

The path of the main flows: what calls what, in order, for the happy path and the
important branches. At Standard/Deep this is often a `sequence` or `dataflow`
diagram — link it.

## Failure and edge behavior

What happens when a dependency is down, a write is retried, input is malformed.
Only for the parts with real risk (Standard/Deep). Where idempotency, retries, or
compensation matter, say so.

## Decisions

Summarize each hard-to-reverse decision in a line and link its ADR. One line here,
the full reasoning in `docs/architecture/adr/NNN-<topic>.md`.

## Diagrams

Links to the `archify` artifacts for this design, with a word on what each shows.

- [Component map](./<path>.html) — services and their connections
- [Sequence: <flow>](./<path>.html) — the request path
- [ERD: <domain>](./<path>.html) — the data model
```

## ADR shape

One file per hard-to-reverse decision, `docs/architecture/adr/NNN-<topic>.md`:

```markdown
# NNN. <Decision>

- Status: accepted | superseded by NNN
- Context: the forces that make this decision necessary.
- Decision: what we chose.
- Alternatives: what we rejected, and why.
- Consequences: what this locks in, and what it costs.
```

## Rules

- **Extend, don't overwrite.** Merge the new design into the existing document.
- **Reference, don't duplicate.** Link the spec, the roadmap, and diagrams.
- **Only hard-to-reverse decisions get an ADR.** Reversible choices stay in prose.
- **Trim to the tier.** Light designs may use three sections; delete the unused
  ones rather than filling them with "N/A".
