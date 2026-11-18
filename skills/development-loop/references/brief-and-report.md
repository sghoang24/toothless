# Brief and Report Files

The controller hands work to a subagent as a **file**, not as pasted text. Two files carry every unit: a **brief** (the requirements going in) and a **report** (the account coming back). They keep the controller's context for coordination instead of for text that gets re-read on every later turn.

A **unit** is one dispatch: a single task, or a **batch** of small same-shape tasks (see the `development-loop` skill's _Batching_). A batch gets **one** brief listing every member, and **one** report.

This is the mechanism behind the rule **"never make a subagent read the whole plan."** A fresh implementer needs its task, the interfaces it touches, and the global constraints — nothing else. Pasting the session's history into a dispatch is the failure this prevents: a real session's dispatch can hit tens of thousands of characters of which almost none is the actual task.

## Why files, not pasted text

- **Context cost.** Everything printed back stays in the controller's context and is re-read on every subsequent turn.
- **Craft, don't inherit.** A subagent never inherits the session's history; the controller constructs exactly what it needs.
- **Survives compaction.** The files on disk outlive the controller's memory of them. After a compaction, the brief and the report are still there to re-read.

## The brief file

Extract the unit's **full text** from `specs/NNN-slug/tasks.md` — the outcome, the acceptance criteria, `Touches:`, `Tests:`, `depends_on`, and the `Verify:` line — into its own file. The task text is the single source of requirements; the dispatch points at the file rather than restating it.

Where to put it: a scratch location that is **git-ignored** — the feature folder is tracked, so do not write briefs there. Use whatever scratch directory the project already excludes from version control. Name it per unit so two units never collide:

```
<scratch>/specs-NNN-slug/unit-<N>-brief.md
<scratch>/specs-NNN-slug/batch-T4-T6-brief.md
```

The brief contains:

- The unit's full text, verbatim from `tasks.md`.
- The feature's `## Test strategy` lines that bind it (command, layer, seams).
- The exact values that must be used verbatim (numbers, strings, signatures, test cases).

**For a batch**, the brief carries every member as a row — its ID, its file, and its change, one line each — plus the same `Test strategy` and values. The implementer makes every listed change; it does not decide scope. If a listed file turns out to need a judgment call, the implementer reports `BLOCKED` rather than improvising: that member belonged on its own.

Names and content stay in the language of the task list. Never make the subagent read the whole plan file.

## The report file

Name it after the brief so the pairing is obvious:

```
<scratch>/specs-NNN-slug/unit-<N>-report.md
<scratch>/specs-NNN-slug/batch-T4-T6-report.md
```

Put the path in the dispatch. The implementer writes the full report there — what it built, the TDD evidence (RED and GREEN), the full-suite result, files changed, self-review findings, and concerns — and returns only the short status contract to the controller. For a batch, one report covers every member.

On a fix round, the implementer **appends** a fix report to the same file: what changed, the covering tests it ran, the command, and the output. Reviewers read that, not a re-run — the report is the test evidence.

## What the dispatch contains

A dispatch describes **one unit**, not the session. Compose it as:

1. **One line** on where this unit fits in the project.
2. **The brief path**, introduced as "read this first — it is your requirements, with the exact values to use verbatim."
3. **Interfaces and decisions from earlier units** the brief cannot know.
4. **Your resolution of any ambiguity** you noticed in the brief.
5. **The report-file path** and the report contract.

Do not carry accumulated prior-unit summaries ("state after Tasks 1-3") into later dispatches. A pointer to a parked finding in the area this unit touches is the one exception: carry that ledger entry, nothing more.

## Rules

- **Hand artifacts over as files.** Brief in, report out. Never paste the session's history into a dispatch.
- **One brief per unit; one report per unit.** Name them after the unit so the pairing is mechanical — a task ID, or the batch's member range.
- **The brief is the interface.** If a requirement is missing from it, fix the brief and re-dispatch — do not paper over a gap with chat text.
- **Scratch is git-ignored scratch.** Briefs and reports are working files, not deliverables. `git clean -fdx` destroys them; the durable record is the commit log and the ledger.
- **The controller keeps a pointer.** Note the brief/report paths for the current unit in `specs/NNN-slug/status.md` notes, so a compacted session can find them again.
