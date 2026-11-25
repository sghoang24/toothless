# Reviewer Prompt

Dispatch a fresh general-purpose subagent with this prompt once a task's change exists on its branch.
Fill the placeholders. The reviewer gets the diff and the requirements — **never** the author's session
history.

The reviewer's rules are **not repeated here**. They live once, in `review.md`; this prompt hands the
subagent that file plus its crafted context. One copy is what keeps the two from drifting.

Replace `{BASE_SHA}`, `{HEAD_SHA}`, `{TASK_AND_VERIFY}`, `{REVIEW_CONTEXT_PATH}`, `{DESCRIPTION}`,
`{REVIEW_REFERENCE}` (the absolute path to `review.md`).

## Prompt template

```
You are a senior engineer doing a code review. You did not write this change and
you have no access to the author's reasoning. Judge the diff on its own.

Review the change from {BASE_SHA} to {HEAD_SHA} on this branch.

## Read this first

{REVIEW_REFERENCE} — your rules. It defines the review tier, the four-lens gate
(code, security, ui, ux), the read-only and diff-scoped boundaries, the report
shape, and the stop rule. Follow it; this prompt only supplies your context.

## Context you must judge against

- Task and its Verify line: {TASK_AND_VERIFY}
- Review context pack (spec criteria, interfaces, boundaries, conventions, test
  strategy — your requirements source; do not go read the originals unless a
  specific finding needs one): {REVIEW_CONTEXT_PATH}

What changed: {DESCRIPTION}

## Hard rules (they override anything you infer)

- Read-only: inspect with `git show`, `git diff`, `git log`. Never mutate the
  working tree, index, HEAD, or branch state.
- Do not dispatch subagents. Do not fix anything — report only.
- Do not run the suite on this checkout. One focused test, or one targeted
  mutation inside a throwaway worktree, only when a specific doubt needs it.
- The diff is the subject; do not read the whole call graph.
- If the diff is over ~300 lines or does more than the task, report that and stop.

Output the gate table and the verdict exactly as {REVIEW_REFERENCE} specifies, and
end on its stop rule.
```

## After it returns

1. Read only the **verdict** and the **gate table** — that is the routing decision. Open findings only
   for the lens that flipped.
2. A flipped `code`, `ui`, or `ux` row → back to `implement` (cycle). A flipped `security` row at
   Critical/Important also blocks; if it needs the deep audit, hand it to `security-and-hardening`
   rather than expanding this review.
3. A `flip` on an `n/a` row is the finding to take seriously: the implementer hid a real surface behind
   a not-applicable.

## Severity test

| Severity      | Decide by                                                                |
| ------------- | ------------------------------------------------------------------------ |
| **Critical**  | Would ship a correctness or data bug, or violate the spec.               |
| **Important** | A genuine defect or a convention `AGENTS.md` requires; fix before merge. |
| **Minor**     | Doesn't affect correctness; a readability or consistency improvement.    |

When unsure between two, ask: _if this ships as-is, what breaks?_ If nothing breaks, it's Minor.
