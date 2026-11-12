---
name: pdnd-merge
description: Find which of the user's open PRs in pagopa pdnd/interop repositories can be merged, update them and resolve conflicts, then merge them and move their Jira tickets. Run only when the user asks for it.
disable-model-invocation: true
---

# PDND Merge

Running this skill is the user's request to merge their ready PRs.

Paths below are relative to the folder that contains this `SKILL.md`. Paths that start with `references/` are in `../pdnd-context/`. Read first:

- `../pdnd-context/references/workflows.md`: preflight, stop and ask, Git safety.
- `../pdnd-context/references/git-github.md`: merge strategy, labels, release gate, Fix Version.
- `../pdnd-context/references/jira-status.md`: target status after the merge.

## Ready To Merge

A PR is ready only when all of these hold. Read them with `gh pr view <n> --repo pagopa/<repo> --json isDraft,reviewDecision,statusCheckRollup,mergeable,mergeStateStatus,labels,baseRefName,title`.

- Not a draft.
- `reviewDecision` is `APPROVED`.
- Every required check passed.
- The labels are complete and the release label matches the gate in `references/git-github.md`.
- The Jira ticket has a Fix Version equal to the release label.

## Steps

1. Run the preflight.
2. List the user's open PRs in the project repositories and check each against "Ready To Merge".
3. For a PR that is ready but behind its base or in conflict, update it with the steps of the `pdnd-sync` skill (`../pdnd-sync/SKILL.md`). The push restarts the checks: do not merge it in this run. Report it as "updated, waiting for checks".
4. Merge each PR that is ready and up to date, with the strategy that `references/git-github.md` sets for its head branch: merge commit (`--merge`) for `release/*` and `align/*`, squash (`--squash`) for every other branch. Use `gh pr merge <n> --repo pagopa/<repo>`, with the PR title as the commit subject. Never use `--admin` and never bypass branch protection.
5. Move the Jira ticket of each merged PR with "Moving A Ticket" in `references/jira-status.md` (`Dev` for `develop`, `Ready to Dev` for `feature/*`). If the ticket has other PRs still open, leave it and report it.

## Report

One table: repository, PR, result (merged, updated and waiting for checks, not ready), missing conditions, Jira status. Ask nothing for the PRs that are not ready: list what each one is missing.
