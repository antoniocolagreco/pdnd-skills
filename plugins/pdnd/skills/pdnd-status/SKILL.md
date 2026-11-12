---
name: pdnd-status
description: Read-only report on the user's open PRs in pagopa pdnd/interop repositories and their Jira tickets - review, checks, conflicts, labels, Fix Version, ticket status - with the next action for each. Use when the user asks for the status of their PRs or tickets.
---

# PDND Status

This skill only reads. Never push, merge, comment, edit a PR, or move a ticket.

Paths below are relative to the folder that contains this `SKILL.md`. Paths that start with `references/` are in `../pdnd-context/`. Read first:

- `../pdnd-context/references/workflows.md`: preflight, project repositories, rate limits.
- `references/jira-status.md`: the status each ticket should have.
- `references/git-github.md`: release label gate, "Draft To Ready", merge conditions.

## Steps

1. Run the preflight for `gh` and the Atlassian MCP server.
2. List the user's open PRs in the project repositories.
3. For each PR, read `gh pr view <n> --repo pagopa/<repo> --json title,isDraft,reviewDecision,statusCheckRollup,mergeable,mergeStateStatus,labels,baseRefName,headRefName,updatedAt`.
4. Take the Jira key from the title or the head branch. Read the ticket's status, Fix Version, and assignee in one JQL query: `key in (...)`.
5. For the release label of each base, read `ALLOWED_RELEASE_TOWARDS_DEVELOP_BRANCH` from `.github/workflows/release-version-validation.yaml` on that repository's base, when the file exists.
6. List the user's tickets in `💾 In progress`, `💾 In review`, or `💾 Ready to Dev` that have no open PR.
7. For each PR, compute:
   - Checks: passed, failed (name the failing check), or pending.
   - Review: approved, changes requested, or waiting.
   - Branch: up to date, behind, or in conflict.
   - Metadata: missing labels, release label that does not match the gate, missing or different Fix Version.
   - Ticket: current status and the status it should have (`references/jira-status.md`).
   - Next action, the first that applies:
     1. Conflict or behind: `pdnd-sync`.
     2. Failed check: fix the cause.
     3. Missing metadata: complete labels or Fix Version.
     4. Ticket in the wrong status: `pdnd-align`.
     5. Draft with every "Draft To Ready" condition met: mark it ready.
     6. Ready, approved, checks passed: `pdnd-merge`.
     7. Otherwise: wait for review.

## Report

In the user's language:

1. One line of totals: open PRs, ready to merge, blocked, drafts.
2. One table, one row per PR, sorted by next action: repository, PR (link), ticket, ticket status, checks, review, branch, metadata problems, next action.
3. The tickets without an open PR, with their status.
4. The PRs without a Jira key.

Keep each cell short. Do not repeat information across sections.
