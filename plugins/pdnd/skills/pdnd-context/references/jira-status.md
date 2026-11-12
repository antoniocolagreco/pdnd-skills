# Jira Status

Source: Confluence page `2136113416` ("[DRAFT] Processo di sviluppo"), section 4.1. Read it again if the user questions a status.

## Status Meaning

| Status | Meaning |
| --- | --- |
| `💾 In progress` | Development started, no PR yet. |
| `💾 In review` | The work is finished and the PR is open, draft or ready. Jira automation sets it when a PR is opened. |
| `💾 Ready to Dev` | The PR is merged into a feature branch, not yet into `develop`. Only on a feature board (issue under an Epic). |
| `💾 Dev` | The PR is merged into `develop`. A merge into `develop` deploys to DEV automatically. |
| `💾 QA`, `Ready to Collaudo`, `Collaudo`, `Done` | Moved by QA, the Feature Lead, or whoever deploys. |

Hotfix, Small Activity, and Refactor tickets (Mixed Activities board) skip `Ready to Dev`: they go from `In review` to `Dev`.

The user manages only `In progress`, `In review`, `Ready to Dev`, and `Dev`. Never move a ticket into another status, and never move a ticket that is in another status (`To do`, `Blocked`, `Rejected`, `QA` and later): report it.

## Align Ticket Status With PRs

Use this when the user asks to align ticket statuses with their PRs. Follow `references/workflows.md` for preflight, PR search, and asking.

1. List the user's open PRs in the project repositories (`references/workflows.md`, "Project Repositories"). Take the Jira key of each.
2. List the user's tickets that may be behind: JQL `assignee = currentUser() AND status in ("💾 In progress", "💾 In review", "💾 Ready to Dev") ORDER BY key`.
3. For each ticket without an open PR, search its PRs: `gh search prs --owner pagopa "<KEY>" --json repository,number,state,title`. Keep only PRs whose title or head branch has the key: ignore Dependabot bumps and PRs that only cite the key.
4. For each merged PR, read the base: `gh pr view <n> --repo pagopa/<repo> --json baseRefName,mergedAt`.
5. Decide the target status from the ticket's PRs:

   | PRs of the ticket | Target |
   | --- | --- |
   | At least one open, draft or not | `💾 In review` |
   | All merged, at least one into `develop` | `💾 Dev` |
   | All merged, into `feature/*` only | `💾 Ready to Dev` |
   | Closed without merge, or none found | No change: report it |

6. A target that moves the ticket backwards (for example from `Dev` to `In review` because a new PR is open) needs the user's choice: ask.
7. Show the table (ticket, PRs, base, current status, target status). When the user runs the `pdnd-align` skill, do not ask for confirmation of the forward moves.
8. Move each ticket, then re-read the statuses with JQL `key in (...)` and report them.

## Moving A Ticket

- Read the transition ids with the Atlassian operation **listJiraIssueTransitions** (tool **executeRead**). In project PIN, `Dev` is id `61` and `Ready to Dev` is id `8`, but check them first.
- Call the tool **executeWrite** with `name: "transitionJiraIssue"` and `inputs: { issueIdOrKey, transitionId: "<id>" }`. The input `transition` is rejected.
- The harness can block Jira writes that it considers unrequested (for example Claude Code auto mode). If that happens, stop and tell the user. The user's explicit request with the ticket keys unblocks it.
