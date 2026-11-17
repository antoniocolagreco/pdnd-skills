# pdnd-skills

Agent skills for PDND Interoperability work. They follow the Agent Skills format (`SKILL.md` with `name` and `description`), so any harness that reads skills can use them.

## Skills

| Skill | What it does | Invoked by |
| --- | --- | --- |
| `pdnd-context` | Project knowledge: conventions, package map, Git and Jira rules, shared workflow rules. | The agent, automatically, or the user |
| `pdnd-work` | Completes a Jira ticket from its key: branch, code, tests, draft PR, Jira status. | The user only |
| `pdnd-align` | Aligns the user's Jira tickets (`In progress`, `In review`, `Ready to Dev`) with their PRs in every pdnd/interop repository. | The user only |
| `pdnd-sync` | Updates the branches of the user's open PRs with their base and resolves conflicts. | The user only |
| `pdnd-merge` | Merges the user's PRs that are approved and green, after updating them, and moves their tickets. | The user only |
| `pdnd-status` | Read-only report on the user's open PRs and their tickets, with the next action for each. | The agent, automatically, or the user |

The skills live in `plugins/pdnd/skills/`. The workflow skills read `pdnd-context` (`references/workflows.md` and the other references), so install all of them together.

## Install

Claude Code (commands `/pdnd:pdnd-context`, `/pdnd:pdnd-align`, ...), inside a session:

```text
/plugin marketplace add antoniocolagreco/pdnd-skills
/plugin install pdnd@pdnd-skills
/reload-plugins
```

Codex, inside a session:

1. Run `/plugins`.
2. Choose "Add a marketplace" and enter `antoniocolagreco/pdnd-skills`.
3. Select `pdnd` and press Enter to install it.

Harnesses without plugin marketplaces: copy or link the folders in `plugins/pdnd/skills/` into the harness's skills folder (for example `~/.agents/skills`).

## Update

Claude Code, inside a session:

```text
/plugin marketplace update pdnd-skills
/reload-plugins
```

Codex: `codex plugin marketplace upgrade pdnd-skills` from the shell, then start a new session.

## Development

Every change to the skills needs a higher `version` in `plugins/pdnd/.claude-plugin/plugin.json`. The harnesses cache the plugin by version: without a new version, an update keeps the old files.

For local development, add the marketplace from the clone path instead of `owner/repo`. Claude Code reads a local marketplace in place. Codex copies the plugin into its cache, so reinstall it after each change.

## Harness notes

- `disable-model-invocation: true` in the workflow skills is a Claude Code field. Other harnesses ignore it and can load a skill on their own, so each description says to run it only on request.
- Dependencies: GitHub CLI (`gh`, logged in) and the Atlassian MCP server. The workflow skills check them first and ask the user to install what is missing.
- `pdnd-context/agents/openai.yaml` is Codex interface metadata.

## Check

```bash
claude plugin validate .
claude plugin validate ./plugins/pdnd
# from the interop-be-monorepo root:
node /home/ac/Projects/pdnd-skills/plugins/pdnd/skills/pdnd-context/scripts/skill-check.mjs check
```
