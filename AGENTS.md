# AGENTS.md — Source of truth for working on Toothless

## What this repo is

**Toothless** is a portable engineering workflow for coding agents, packaged as composable
**skills**. It is the _methodology layer_ (like superpowers), not a project template and not an
application. This file is the source of truth for agents **maintaining this skill pack**.

> A project that _uses_ the skills has its **own** `AGENTS.md`. Everything below governs this
> repo, `sghoang24/toothless`.

See `README.md` for the full guide.

## What lives here (and what does not)

This repo ships:

- `skills/<name>/SKILL.md` — the workflow, one folder per phase/skill (+ `references/` per skill)
- `commands/fight.md` — the thin `/fight` entry point
- `README.md`, `AGENTS.md`, `LICENSE`, `package.json`

The target project — _not this repo_ — grows `specs/`, `docs/`
as the workflow runs. Do not scaffold them here.

## Source of truth (no duplication)

- The **workflow and its gates** live in `skills/fight/SKILL.md`.
- The **development loop** (`implement -> verify -> review`) lives in `skills/development-loop/SKILL.md`.
- Each phase's **how** lives in its own skill; the spine links to it, never restates it.
- The **skills catalogue** is `skills/README.md`.

When two files disagree, fix the one that _owns_ the fact and link from the other.

## Authoring skills (the core convention)

- One folder per skill: `skills/<kebab-name>/SKILL.md`; `name` in frontmatter **must match** the
  folder name.
- `description` is the discovery surface: state **what** it does and **when** to use it
  (trigger phrases included). Keep it under 1024 chars. README/AGENTS must not summarize a
  skill's process — link to it.
- Skills are **model-free**. Never put model names, tool lists, or harness policy in a `SKILL.md`;
  that belongs in the per-harness adapter layer.
- Follow the house sections: **Model** or **Inputs**, **When to Use** (with "Do NOT use"), **Core Rules**,
  **Procedure**, **Red Flags**, **Completion Check**.
- Keep a `SKILL.md` under ~500 lines; push detail into `references/` (progressive loading).
- One concern per skill. If a new idea overlaps an existing skill, extend that skill instead of
  adding a near-duplicate.
- Cross-reference other skills by name and relative `./` path; never copy their content.

## Workflow (when building features _of the pack_)

`Brainstorm -> Roadmap -> Design -> Plan (incl. test-strategy) -> Development (loop) -> Deploy`

1. No skill change without an agreed spec/description of the intended behavior.
2. The loop is capped at **3 cycles per unit** (a task, or a batch of small same-shape tasks); after that, STOP and ask the human.
3. The reviewer runs in a **fresh session**, never the one that wrote the change.
4. Landing gets an **MR/PR raised for the human**; merging needs explicit approval, exactly like a
   deploy.

## Conventions

- **Docs are the product.** Changes are Markdown; keep prose tight and consistent in voice.
- Small commits, Conventional Commits (`feat:`, `fix:`, `docs:`, `test:`, `chore:`).
- One change = one branch (`feat/NNN-slug` or `docs/NNN-slug`) = one PR. Keep diffs under ~300 lines.
- Keep terminology consistent across skills (phase names, `specs/_state.md`, `specs/_status.md`, `feat/NNN-slug`).
- Validate frontmatter (`name` matches folder, `description` present) before committing.

## PR instructions

- Title format: `[project-name] <Title>`
- `main` is protected: land changes through a reviewed PR — never push directly.

## Boundaries (never do)

- Never commit the skill's scope creep: no application code, no `specs/`/`docs/` here.
- Never put model names, tool lists, or secrets into a `SKILL.md`.
- Never add a dependency without asking.
- Never force-push, and never push **directly** to `main`/`master` — land through a reviewed PR.
  Never merge or deploy without approval.
