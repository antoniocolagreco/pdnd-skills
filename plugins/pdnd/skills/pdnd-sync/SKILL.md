---
name: pdnd-sync
description: Update the branches of all the user's open PRs in pagopa pdnd/interop repositories with their base branch and resolve the conflicts. Run only when the user asks for it.
disable-model-invocation: true
---

# PDND Sync

Running this skill is the user's request to update and push the branches of their open PRs.

Paths below are relative to the folder that contains this `SKILL.md`. Read first:

- `../pdnd-context/references/workflows.md`: preflight, stop and ask, local clones, Git safety, verification.
- `../pdnd-context/SKILL.md`: project conventions.

## Steps

1. Run the preflight.
2. List the user's open PRs in the project repositories. For each, read `gh pr view <n> --repo pagopa/<repo> --json headRefName,baseRefName,mergeable,mergeStateStatus,isDraft`.
3. Keep the PRs that are behind their base (`mergeStateStatus` is `BEHIND`) or in conflict (`mergeable` is `CONFLICTING`). Leave the others.
4. For each kept PR, in its local clone:
   1. Check the working tree, then fetch the remote.
   2. Check out the head branch. If the remote branch is ahead, fast-forward to it. If it diverged, stop and ask.
   3. Merge `<remote>/<base>` into the branch.
   4. Resolve the conflicts. Keep both sides' intent: the PR follows its ticket, the base brings everyone else's work. Apply "Stop And Ask" for any conflict that needs a decision.
   5. Check what the merge broke even without conflicts: for example a code or constant the PR adds that the base now uses for something else. Renumber or rename on the PR side and update the PR description if it cites the old value.
   6. Commit with the default merge message, check the fast-forward, push. Do not run builds, typecheck, lint, or tests: the CI checks every pushed PR.
5. Go back to the branch checked out at the start.
6. Re-read `mergeable` and `mergeStateStatus` of every PR you pushed. Do not wait for the checks.

## Report

One table: repository, PR, what was wrong (behind or conflict), how you resolved it, new GitHub status. Say that no local verification ran and the CI checks are pending. List the PRs left as they are, with the reason.
