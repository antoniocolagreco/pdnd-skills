# Git, GitHub, And Jira Metadata

Source: Manuale di Integrazione GitHub-Jira (Confluence page `1900806152`), "[DRAFT] Gestione delle Mixed Activities" (page `3418685526`), "[DRAFT] Processo di sviluppo" (page `2136113416`), plus the CI workflows. Never add AI authorship text anywhere: see the rule at the top of `SKILL.md`.

## Before Starting

Collect the Jira key, a short English description, the change type, and the affected packages. Ask for everything missing in one question. Read the Fix Version from the ticket: never ask for it and never choose it (see "Fix Version And Release Label").

If the task involves a commit, push, or PR:

1. Run `git fetch` and `git status -sb`.
2. Behind upstream with a clean tree: `git pull --ff-only`.
3. Behind with local changes, or diverged: stop and ask. Never merge, rebase, or force-push on your own.

## Branches

Format: `<prefix>/<JIRA_KEY>_<short-kebab-description>`. The key is uppercase and the description is in English. Only `release/x.y.z` has no key. One issue maps to one branch and one PR.

| Type | Branch | Origin | Merge |
| --- | --- | --- | --- |
| `feature` | `feature/PIN-123_add-rate-limiter` | `develop` | Squash to `develop` |
| `fix` | `bugfix/PIN-456_fix-typo` | `develop`, `feature/*`, `release/*` | Squash to origin |
| `hotfix` | `hotfix/PIN-789_resolve-visibility` | `main` | Squash to `main`, then align |
| `refactor` | `refactor/PIN-321_extract-retry-logic` | `develop`, `feature/*`, `release/*` | Squash to origin |
| `perf` | `perf/PIN-582_reduce-polling` | `develop`, `feature/*`, `release/*` | Squash to origin |
| `docs` | `docs/PIN-782_extend-readme` | `develop`, `feature/*`, `release/*` | Squash to origin |
| `build` | `build/PIN-232_change-dockerfile` | `develop`, `feature/*`, `release/*` | Squash to origin |
| `ci` | `ci/PIN-232_change-github-action` | `develop`, `feature/*`, `release/*` | Squash to origin |
| `chore` | `chore/PIN-232_format-code` | `develop`, `feature/*`, `release/*` | Squash to origin |
| `test` | `test/PIN-825_add-eservice-test` | `develop`, `feature/*`, `release/*` | Squash to origin |
| `release` | `release/2.7.0` | `develop` | Merge commit to `main`, then align |
| `align` | `align/PIN-836_2025-03-27` | `main` | Merge commit to `develop` |

Bug fixes use the branch prefix `bugfix` and the commit type `fix`.

**Hotfix and align conflict:** the Manuale says the above, but recent PRs differ. Hotfixes go from `hotfix/PIN-XXX_<description>` into `hotfix/x.y.z`, then into `main`. Align branches are named `align/x.y.z`, and some target an open `release/x.y.z`. To see the current practice, run `gh pr list --state merged --search "head:align"`. Before creating one of these branches, show the user both options and wait.

## Commits

```text
<type>: <short English description> (<JIRA-KEY[, OTHER-KEY]>)
```

- Allowed types: `feature`, `fix`, `hotfix`, `refactor`, `perf`, `docs`, `build`, `ci`, `chore`, `align`, `test`.
- Maximum 100 characters, with the Jira key in uppercase.
- Subject line only. The user decided this: the PR description carries the detail, even though the Manuale suggests a body. Add a body only with permission, for a `BREAKING CHANGE` or `perf` metrics.
- One commit per logical task.

Choosing the type:

- `feature`: new externally visible capability, minor version.
- `fix`: bug in `develop`, QA, or a feature or release branch, patch version.
- `hotfix`: urgent production fix from `main`, patch version.
- `refactor`: no external behavior change.
- `perf`: latency, memory, or I/O. Put baseline and new metric in the body, with permission.
- `docs`: README, guides, runbooks.
- `build`: Dockerfile, Makefile, toolchain, dependencies.
- `ci`: GitHub Actions and pipelines.
- `chore`: cleanup with no behavior change.
- `test`: test-only changes.
- `align`: aligns `develop` with `main`.

## Force Push

The user decides, every time.

- Never force-push to fix a pushed commit message. Feature branches reach `develop` by squash, so the message rarely matters. Ask.
- Avoid it on a branch with an approving review.
- If authorized: `--force-with-lease`, feature branches only. Rewriting drops GPG/SSH signatures, and commits then show as `Unverified`.
- Before any push, confirm it is fast-forward: `git merge-base --is-ancestor <remote> HEAD`. If it is not, stop and ask.

## Pull Requests

Open every PR as Draft. Check `gh auth status` first. If `gh` or Jira fails, stop and ask.

The title equals the final commit subject. Several issues: `(PIN-1, PIN-2)`.

The body uses the Manuale template, with these exact names and lines. Most merged PRs do not follow it, so never copy the structure from an existing PR.

```markdown
## Jira Issue
[PIN-XXX](https://pagopa.atlassian.net/browse/PIN-XXX)

## Context/Why
- <what is the problem and why this change is needed>

## Services Impacted
- <package or service>

## Key Changes
- <main change>

## Traceability Checklist
- [ ] Branch name contain the Jira key
- [ ] PR title is compliant with the standard (type(scope): descrizione (KEY))
- [ ] Service labels applied
- [ ] Fix Version set in Jira (if required)
- [ ] API and integration tests updated (if required)
- [ ] OpenAPI spec updated (if required)
- [ ] Bruno endpoint definition updated
```

Keep the title as `type: description (KEY)`, the Manuale's commit format, even though the checklist line mentions `type(scope)`.

What goes in each section:

- **Context/Why:** the problem in plain language and what it breaks, in one or two bullets. If a bullet answers "where" or "how", cut it. If it answers "what is broken" or "what does it break", keep it.
- **Services Impacted:** package names only.
- **Key Changes:** one bullet per change, in plain words at a conceptual level: what now behaves differently and for whom (endpoint or flow, before and after). Do not list files or functions; name an endpoint, field, or error only when it makes the change clear. Only real code changes: no rebase or verification notes. This is the only section with detail. Example: "Interface upload (BFF, M2M Gateway): when the WSDL has no server address, the response is now 400 with the specific error instead of 500."
- **Traceability Checklist:** verbatim. Tick what is true. Do not add, remove, or annotate items.
- Nothing outside these sections.

Everything on GitHub is in English, even when the ticket and the chat are in Italian.

If merging `develop` brings behavior opposite to your ticket, the PR keeps following its ticket. Leave the other ticket's unused code in place, note it in the PR, and tell the user.

## Labels

Required on every PR. Never invent a label: check with `gh label list`.

- **Type:** one that exists: `bug`, `hotfix`, `chore`, `refactor`, `feature-branch`.
- **Activity:** for example `archiviazione`, `api-v3`, `catalogo-privato`.
- **Service:** for example `catalog-process`, `bff`.
- **Release:** `x.y.z`, only when the ticket already has a Fix Version. See "Fix Version And Release Label".

A PR to `develop` fails `.github/workflows/release-version-validation.yaml` unless the release label equals `ALLOWED_RELEASE_TOWARDS_DEVELOP_BRANCH`. Read the value in that file on `develop`. Only a PR titled exactly `align/x.y.z` skips this check.

## Fix Version And Release Label

Source: page `3418685526` (phases 0 and 5) and the team lead. The Manuale's step "Imposta Fix Version su Jira" for the developer is outdated.

- Never set or change the Fix Version on Jira. The Mixed Activities Manager sets it, after the evaluation with QA and the insertion in a sprint.
- The ticket already has a Fix Version: add the release label with that exact value.
- The ticket has no Fix Version: add no release label, even if the release gate fails. Whoever sets the Fix Version adds the label. Report the failing gate as expected, never fix it.
- Never pick the release from `ALLOWED_RELEASE_TOWARDS_DEVELOP_BRANCH`. A Fix Version different from it is the team's decision: report it.
- An urgent bug needs a release now: stop and tell the user to agree it first with the team lead.

## Jira And Merge

- Link the branch and the PR to the issue.
- Fix Version is mandatory before merge and matches the release label.
- Approval: at least one PagoPA Codeowner, or one external Codeowner plus one internal developer not involved in the feature (the internal developer can be skipped for an urgent or very small PR). A Codeowner does not approve PRs of their own feature team. This also applies to merges into a `feature/*` branch.
- Move the PR from Draft to ready only when verification is complete (see "Draft To Ready").
- After answering or fixing review comments, request the review again from each reviewer: `gh pr edit <n> --repo pagopa/<repo> --add-reviewer <login>`.

## Draft To Ready

The Manuale opens every PR as a draft and moves it to review "if everything is ok". Ready means a reviewer can approve it without asking for changes you could have caught. All of these must hold:

- Every check on the last commit passed. Only exception: the release label gate, when the ticket has no Fix Version yet.
- The branch is up to date with its base and has no conflicts.
- Title, template, and labels are complete and final. The release label is present only if the ticket has a Fix Version.
- The diff contains only the ticket's scope: no debug code, no unrelated changes.
- The local verification is done: tests for the change, typecheck, lint, and `knip` or `lint:openapi` when they apply.

The `pdnd-work` skill waits for the checks and marks the PR ready when all of these hold. In any other task, the user marks it ready. Report which conditions are still missing.

## Checklist Before A Commit

1. `git branch --show-current`.
2. `pnpm lint:autofix` and `pnpm check` in each modified package.
3. No AI authorship text and no em dash in the message.
4. For a PR: template, labels, release label only if the ticket has a Fix Version.

## Output For Git Metadata

When the user asks for branch, commit, or PR text:

```text
Branch: bugfix/PIN-456_fix-thing
Commit / PR title: fix: fix thing (PIN-456)
Origin -> Merge: develop -> Squash Merge on develop
PR: Draft
Labels: type=<type>, activity=<activity>, service=<service>, release=<Fix Version, or none>
Fix Version: <from the ticket, or "not set: decided by the team">
```
