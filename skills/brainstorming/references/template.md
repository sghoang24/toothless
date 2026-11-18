# Brainstorming templates

Write in the user's language. Keep IDs, file names, and keys in English.

## 1. PRD (`docs/product/PRD.md`, project mode)

A living charter. Extend it as direction firms up; log every change at the bottom.

```markdown
---
status: draft # draft | approved
updated: YYYY-MM-DD
---

# PRD: <project name>

## 1. Purpose

One paragraph: what it is, for whom, why now.

## 2. Goals and success metrics

- G1: <goal> — measured by <metric / how we will know>

## 3. Non-goals and anti-goals

- Non-goal: <deliberately not doing>
- Anti-goal: <would make this a failure even if it technically works>

## 4. Users and key scenarios

- <user type>: <scenario in one or two lines>

## 5. Functional requirements

- FR-1: <observable behavior, one requirement per line>
- FR-2: ...

## 6. Non-functional requirements

- NFR-1: <performance / security / multi-tenancy / compliance / ...>

## 7. Constraints and assumptions

- Stack and platform (mark which are the user's decisions and which are your calls)
- Budget, time, team, existing systems

## 8. Risks and open questions

- R1: <risk> — <mitigation or owner>
- Q1: <question only the user or later research can answer>

## Revision log

- YYYY-MM-DD: <what changed and why>
```

Rules: one requirement per ID, each phrased so it can be checked; never renumber existing IDs (retire them with "(removed)" instead), because `roadmap` and `specs/` refer to them.

## 2. Feature spec (`specs/NNN-slug/YYYY-MM-DD-<topic>.md`, feature mode)

A complete, stand-alone document each revision; the newest date is current.

```markdown
# <feature> — spec

Revision: YYYY-MM-DD (supersedes YYYY-MM-DD) # first revision: "Revision: YYYY-MM-DD (initial)"
Changes since previous revision:

- NEW: ...
- CHANGED: ...
- UNCHANGED: (names only)

## Intent and why

## Covers

FR-3, FR-4 (from docs/product/PRD.md)

## Goals / Non-goals

## Approaches considered

Chosen: <approach> — why. Rejected: <approach> — why.

## Decisions (the user's)

## Calls I made (mark for the user to check)

## Acceptance criteria

- AC-1: Given <state>, when <action>, then <observable result>.
- AC-2: ...

## Risks and open questions

## Left to the builder
```

Every acceptance criterion must be testable by someone who has not read the conversation. `planning`, `test-strategy`, and the loop's `review` rely on them.

## 3. Short design (small change, shown in chat)

One or two lines each: what changes, why, what it touches, how we will know it works (one or two checks), your calls. End by asking for approval.

## 4. Builder-check triage

| Pile             | Action                                        |
| ---------------- | --------------------------------------------- |
| Answered already | Put the answer in the document                |
| Minor            | Decide it, mark it as "Calls I made"          |
| Theirs           | One message to the user, most important first |
