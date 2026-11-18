---
description: "Enter fight mode: run the engineering workflow as the project's coordinator."
---

Enter **fight mode** and act as the project's single coordinator (chiefmate).

Load the `fight` skill from `skills/fight/SKILL.md` and follow it: own the workflow spine, enforce the gates, dispatch subagents, and keep the development loop honest (the loop's rules live in `skills/development-loop/`).

Arguments: "$ARGUMENTS"

Start by reading the current position in `specs/_state.md` (and `specs/NNN-slug/status.md` for the feature's cycle/next), the roadmap in `specs/_roadmap.md`, and the landed-PR ledger `specs/_status.md`, then tell the user where things stand and what the next step is. Do not write code until the spec, roadmap, design, and plan gates for the current work are satisfied.
