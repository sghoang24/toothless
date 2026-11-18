# Task Sizing

A task is the smallest unit a human would review as one change. The target, per
`AGENTS.md`: one feature = one branch (`feat/NNN-slug`) = one PR, diff under
~300 lines.

## The size test

| Too small                                                 | Just right                                                    | Too big                                                                         |
| --------------------------------------------------------- | ------------------------------------------------------------- | ------------------------------------------------------------------------------- |
| Rename a variable; add a single field with no use; a TODO | A coherent change with its tests, doing one discernible thing | Touches many layers/modules; "build the auth system"; no single thing to verify |
| Would be merged with its neighbor anyway                  | Could stand alone as a reviewable PR                          | Needs its own plan internally                                                   |
| Its outcome can't be described without the next task      | Outcome is describable in one sentence                        | Outcome is a sentence with "and" three times                                    |

## Heuristics

- **Vertical, not horizontal.** A task should cross the layers needed to make
  one behavior work (per `AGENTS.md`: business logic lives in the service layer,
  not the controller). Avoid "add all the model fields" as a standalone task.
- **One verb.** If a task title needs "and", it's usually two tasks — unless the
  parts are inseparable (a migration and its model change).
- **Testable on its own.** If you can't write the `Verify:` line, it's not sized.
- **Don't split by file.** Splitting one logical change across files into separate
  tasks creates coordination overhead with no review value.
- **Spikes are tasks too.** A timeboxed investigate-and-report task is fine; label
  it and keep its output throwaway unless a new request adopts it.
- **A run of too-small tasks is a batch, not a defect to fix late.** If a feature
  genuinely needs many one-line, same-shape edits (rename a constant, add a field,
  fix the same call in six files), do not force them into one artificial task and
  do not leave them as six. Group them as a **batch** — `development-loop`
  dispatches and reviews it as one unit. Note the group in `tasks.md` (consecutive
  IDs and the same shape) so the loop can see it. The batch is a legitimate answer to
  "this is too granular", not an excuse to leave the granularity in place.

## Anti-patterns

- **Micro-tasks** — a dozen tasks for one feature. Merge them, or batch the ones that
  are small _and the same shape_ (see Heuristics); a micro-task run that is not
  uniform is really a sign the split is wrong.
- **Megatasks** — "implement the dashboard". Split along behaviors.
- **Layer tasks** — "create the DTOs", "now the services", "now the controller".
  Slice vertically instead.
- **Invisible tasks** — no outcome, no verify. These hide unknown work.
