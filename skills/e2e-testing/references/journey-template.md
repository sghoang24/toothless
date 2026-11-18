# Journey Template

One entry per end-to-end journey. Keep it short — this is the design of the
test, not the test code. Write it before you write the test.

## Template

```markdown
### Journey: <name the user goal, e.g. "New member joins and sees the board">

- **User goal:** <what the person is trying to accomplish, in their words>
- **Critical because:** <why this must never break — core flow / money / security / data loss>
- **Covers:** <the components and seams this exercises, so we know what it proves>
- **Setup:** <data and state this journey needs, created by the test itself>
- **Steps:**
  1. <user-visible action with a stable selector or role>
  2. <...>
  3. <...>
- **Expected:** <the single observable outcome that means it worked>
- **Cleanup:** <what the test removes so the next run is independent>
- **Source:** <spec acceptance criterion / PRD line it comes from>
```

## Good vs weak journeys

| Weak                                    | Why                             | Better                                                           |
| --------------------------------------- | ------------------------------- | ---------------------------------------------------------------- |
| "Settings page loads without error"     | A page, not a user goal.        | "Member updates their display name and sees it everywhere."      |
| "Click every button on the dashboard"   | Mirrors the UI, huge and slow.  | "Member creates a task and it appears in the board and list."    |
| "API returns 200 for /health"           | Unit-level, e2e adds nothing.   | "Signed-in member's first load shows their data, not a spinner." |
| "Full checkout with 12 payment methods" | Rare paths dilute the critical. | "Member buys the base plan; card is charged once."               |

## Ranking the critical few

Order journeys by blast radius if they silently broke:

1. **Core flow** — the reason the product exists (sign up → do the thing).
2. **Money / security / data loss** — payment, auth, delete, destructive writes.
3. **Seams a risky change touched** — the components this milestone's work joined.

Stop when the next journey is a variant of one you have. Coverage of _edges_ is
the unit suite's job; e2e proves the critical paths hold end to end.

## Naming

Name the journey by the **outcome**, not the mechanism: `member-joins-and-sees-board`,
not `board-render-test`. The name should read like the acceptance criterion it
answers.
