---
name: pdnd-work
description: Complete a PDND Jira ticket end to end from its key (for example PIN-1234): read the ticket and SRS, branch, implement, test, commit, open a draft PR, and update Jira. Run only when the user asks for it.
disable-model-invocation: true
---

# PDND Work

The user gives a Jira key, for example `PIN-1234`. If it is missing, ask for it.

Paths below are relative to the folder that contains this `SKILL.md`. Paths that start with `references/` are in `../pdnd-context/`. Read first:

- `../pdnd-context/SKILL.md`: project conventions and where to find everything else.
- `../pdnd-context/references/workflows.md`: preflight, stop and ask, local clones, Git safety, verification.

## Steps

1. Run the preflight.
2. Read the ticket and its linked pages with `references/atlassian-context.md`. The ticket defines the scope: do only what it says.
3. Decide whether the ticket is ready (`references/jira-tickets.md`, "Reviewing A Ticket"). If not ready, stop and ask with the blockers and the proposed answers. Propose moving the ticket to `💾 Blocked` and contacting the requester (`references/jira-status.md`).
4. Find the repository and the packages that own the change (`references/package-map.md`). Find the change type and the branch origin (`references/git-github.md`).
5. In the local clone, follow "Before Starting" in `references/git-github.md`, then create the branch `<prefix>/<KEY>_<short-kebab-description>` from the origin.
6. Move the ticket to `💾 In progress` and assign it to the user if it has no assignee (`references/jira-status.md`, "Moving A Ticket").
7. For a bug, reproduce it first (`references/debugging.md`). The regression test must fail before the fix.
8. Implement with the guide of the package type and the closest sibling code. Write the tests (`references/testing.md`).
9. Run the verification, plus `pnpm knip` if exports or dependencies changed and `lint:openapi` if OpenAPI changed.
10. Check "Before Saying Done" in `../pdnd-context/SKILL.md`.
11. When the work is finished, commit, push, and open the PR as a draft (`references/git-github.md`): title, template, labels. Add the release label only if the ticket already has a Fix Version. Never set the Fix Version ("Fix Version And Release Label").
12. The ticket stays in `💾 In progress` while the PR is a draft. If Jira automation moved it to `💾 In review`, report it.
13. Wait for the checks: `gh pr checks <n> --repo pagopa/<repo> --watch`. They can take a long time: run the wait in the background if the harness supports it. If no check is reported yet, wait one minute and try again.
14. If a check fails, read its log (`gh run view <run-id> --repo pagopa/<repo> --log-failed`):
    - Caused by the change: fix it, verify, commit, push, and go back to step 13. After three failed rounds, stop and ask.
    - The release label gate, and the ticket has no Fix Version: expected. Do not add a label: report it.
    - Not caused by the change (flaky test, infrastructure, another team's code): stop and ask. Do not re-run checks or change unrelated code on your own.
15. When every check passed, verify the other conditions of "Draft To Ready" in `references/git-github.md`. If they all hold, mark the PR ready: `gh pr ready <n> --repo pagopa/<repo>`, then move the ticket to `💾 In review` (`references/jira-status.md`, "Moving A Ticket"). Otherwise leave it as a draft and report what is missing.

## Report

The branch, the PR link, the ticket status, the checks run and their results, the checks not run, and the open points for the user.
