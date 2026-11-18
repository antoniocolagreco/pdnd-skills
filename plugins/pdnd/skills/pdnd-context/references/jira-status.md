# Jira Status

Sources: Confluence pages `2136113416` ("[DRAFT] Processo di sviluppo", section 4.1) and `3418685526` ("[DRAFT] Gestione delle Mixed Activities", phases 1 and 4).

`Ready to Dev` applies to every ticket, Mixed Activities included. Table 4.1.4 and phase 4 ("PR mergiata: Dev") omit it by mistake: see the user's comment `3477110914` on page `2136113416`.

## Status Meaning

| Status | Meaning |
| --- | --- |
| `💾 In progress` | Development started. No PR yet, or the PR is still a draft. |
| `💾 In review` | The work is finished and the PR is ready for review (not a draft). |
| `💾 Blocked` | The ticket lacks information needed to develop it. The requester must unblock it. |
| `💾 Ready to Dev` | The PR is merged, into `develop` or into a `feature/*` branch. |
| `💾 Dev` | Deployed to the DEV environment. Only whoever deploys moves it: the deploy is manual and is not visible from the PRs. |
| `💾 QA`, `Ready to Collaudo`, `Collaudo`, `Done` | Moved by QA, the Feature Lead, or whoever deploys. |

The user manages only `In progress`, `In review`, `Ready to Dev`, and `Blocked`. Move a ticket to `Blocked` only after the user confirms it, and tell the user to contact the requester. Never move a ticket into another status, `Dev` included, and never move a ticket that is in another status (`To do`, `Blocked`, `Rejected`, `Dev`, `QA` and later): report it.

Jira automation can set `In review` when a PR is opened, even as a draft. A ticket in `In review` with only draft PRs is behind the rule: report it, and move it back only with the user's choice.

## Align Ticket Status With PRs

Use this when the user asks to align ticket statuses with their PRs. Follow `references/workflows.md` for preflight, PR search, and asking.

1. List the user's open PRs in the project repositories (`references/workflows.md`, "Project Repositories"). Take the Jira key of each.
2. List the user's tickets that may be behind: JQL `assignee = currentUser() AND status in ("💾 In progress", "💾 In review", "💾 Ready to Dev") ORDER BY key`.
3. For each ticket without an open PR, search its PRs: `gh search prs --owner pagopa "<KEY>" --json repository,number,state,title`. Keep only PRs whose title or head branch has the key: ignore Dependabot bumps and PRs that only cite the key.
4. For each merged PR, read the base: `gh pr view <n> --repo pagopa/<repo> --json baseRefName,mergedAt`.
5. Decide the target status from the ticket's PRs:

   | PRs of the ticket | Target |
   | --- | --- |
   | At least one open and not a draft | `💾 In review` |
   | Open PRs, all drafts | `💾 In progress` |
   | All merged, into `develop` or `feature/*` | `💾 Ready to Dev` |
   | Closed without merge, or none found | No change: report it |

6. A target that moves the ticket backwards (for example from `Ready to Dev` to `In review` because a new PR is open) needs the user's choice: ask.
7. Show the table (ticket, PRs, base, current status, target status). When the user runs the `pdnd-align` skill, do not ask for confirmation of the forward moves.
8. Move each ticket, then re-read the statuses with JQL `key in (...)` and report them.

## Moving A Ticket

- Read the transition ids with the Atlassian operation **listJiraIssueTransitions** (tool **executeRead**). In project PIN, `Ready to Dev` is id `8`, but check it first. If the ticket has no transition to `Ready to Dev`, do not pick another one: report it.
- Call the tool **executeWrite** with `name: "transitionJiraIssue"` and `inputs: { issueIdOrKey, transitionId: "<id>" }`. The input `transition` is rejected.
- The harness can block Jira writes that it considers unrequested (for example Claude Code auto mode). If that happens, stop and tell the user. The user's explicit request with the ticket keys unblocks it.
