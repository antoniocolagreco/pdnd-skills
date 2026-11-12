# Workflows

Shared rules for the workflow skills `pdnd-work`, `pdnd-align`, `pdnd-sync`, `pdnd-merge`, and `pdnd-status` (read-only). Read this file before starting any of them.

## Preflight

Check the dependencies before the first step of any skill of this plugin.

| Dependency | Needed by | Check |
| --- | --- | --- |
| GitHub CLI, logged in | every skill | `gh auth status` shows an active account |
| Atlassian MCP server | every skill | Jira tools are available (for example **getJiraIssue**) and a read of one known issue succeeds |
| Node and pnpm | `pdnd-work` | `node -v` satisfies `engines` in the repository's root `package.json`; `pnpm -v` matches its `packageManager` |

If one is missing, stop at once and explain to the user how to install it, with the commands for their system and harness from "Install Instructions". Give only the commands that apply: detect the system with `uname -s` and `/etc/os-release`, and name the harness you are running in. Then wait. Never install, log in, or change credentials yourself: they need the user's account.

## Install Instructions

GitHub CLI:

| System | Install |
| --- | --- |
| Arch Linux | `sudo pacman -S github-cli` |
| Debian, Ubuntu | `sudo apt install gh` |
| Fedora | `sudo dnf install gh` |
| macOS | `brew install gh` |
| Windows | `winget install --id GitHub.cli` |

Then: `gh auth login` (GitHub.com, HTTPS, browser login). The account needs access to the `pagopa` org.

Atlassian MCP server, URL `https://mcp.atlassian.com/v2/mcp`:

| Harness | Add | Authorize |
| --- | --- | --- |
| Claude Code | `claude mcp add --transport http atlassian https://mcp.atlassian.com/v2/mcp -s user` | `/mcp` in a session, then log in to Atlassian in the browser |
| Codex | `codex mcp add atlassian --url https://mcp.atlassian.com/v2/mcp` | `codex mcp login atlassian` |
| OpenCode | `opencode mcp add` (remote server, name `atlassian`, the URL above) | `opencode mcp auth atlassian` |
| Other harnesses | Add a remote HTTP MCP server named `atlassian` with the URL above, following the harness's MCP documentation | The harness's OAuth flow |

Restart the session after adding the server, so its tools load.

Node and pnpm:

- Node: install the version that `engines` requires (for example with `fnm install <version>` or `nvm install <version>`, or the system package manager).
- pnpm: `corepack enable`, so the version in `packageManager` is used.

## Stop And Ask

Stop and ask when an assumption could cause damage or a wrong choice. Examples:

- The ticket, the SRS, or the code leaves the expected behavior open.
- A conflict touches `pnpm-workspace.yaml`, `pnpm-lock.yaml`, dependency entries, or migrations.
- `develop` brings behavior opposite to what the PR implements.
- The working tree has changes you did not make, or the branch has diverged from its remote.
- An action is irreversible and a rule in this skill does not cover it.

Mechanical work needs no question: moved imports, renamed symbols, a value that both sides changed in the same way.

## Asking

- Offer a multiple choice: two to four concrete options, the recommended one first, and an open option for a different solution.
- Use the harness's question tool when it has one. Otherwise write numbered options and end with "Altro: descrivi la tua soluzione".
- Say in one line what you found and why it blocks you. Ask everything that is blocking in one round.

## Project Repositories

The project lives in the `pagopa` GitHub org. A repository belongs to it when its name contains `pdnd` or `interop`.

- The user's open PRs: `gh search prs --owner pagopa --author @me --state open --limit 100 --json repository,number,title,url,isDraft`. Keep only the repositories whose name contains `pdnd` or `interop`.
- The user's merged PRs: same command with `--merged --merged-at ">=<date>"` instead of `--state open`.
- Search allows 30 calls per minute. On error, check `gh api rate_limit --jq .resources.search` and wait for the reset.
- The Jira key of a PR is in its title, for example `(PIN-1234)`. If it is missing there, read the head branch name. If both lack it, report the PR and do not guess.

## Local Clones

- Look for the clone in `~/Projects/<repo>`. If it is not there, ask for its path.
- Find the remote that points to `pagopa/<repo>` with `git remote -v`. Its name can be `origin` or `github`.
- Work in the main clone. Do not create worktrees.
- Before switching branch, run `git status --short`. If there are changes you did not make, stop and ask.
- At the end, go back to the branch that was checked out at the start.

## Git Safety

- Never rebase, never force-push, never rewrite history.
- Update a branch by merging its base into it, with the default merge message.
- Before every push, check that it is a fast-forward: `git merge-base --is-ancestor <remote>/<branch> HEAD`.
- Check `git branch --show-current` before every commit.

## Verification

`pdnd-sync` and `pdnd-merge` skip local verification: the CI checks every pushed PR. The rules below apply to `pdnd-work`.

- Build the shared packages you changed (`pnpm build --filter <package>`), then trust `npx tsc -p tsconfig.check.json` in each changed package, not the editor.
- If the lockfile or a `package.json` changed, run `pnpm install --frozen-lockfile --offline`.
- Run only the test files that cover the change, at most five. Ask before running a whole package suite.
- Report what you ran and what you did not run. Never report an unrun check as passing.

## Report

End with one table per workflow, in the user's language. List what was done, what was left as it is, and why.
