---
name: pdnd-align
description: Align the user's PDND Jira ticket statuses (In progress, In review, Ready to Dev) with the state of their PRs in every pagopa pdnd/interop repository. Run only when the user asks for it.
disable-model-invocation: true
---

# PDND Align

Running this skill is the user's request to move their Jira tickets. Do not ask for confirmation of forward moves.

Paths below are relative to the folder that contains this `SKILL.md`.

1. Read `../pdnd-context/references/workflows.md` and run its preflight.
2. Read `../pdnd-context/references/jira-status.md` and run "Align Ticket Status With PRs" end to end.
3. Report one table: ticket, PRs, base branch, old status, new status. Then list the tickets left as they are, with the reason.
