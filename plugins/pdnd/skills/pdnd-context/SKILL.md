---
name: pdnd-context
description: Use when working in the PagoPA PDND Interoperability backend monorepo on TypeScript services, process services, BFF, gateways, routers, validators, domain errors, OpenAPI/protobuf/schema changes, readmodel projections, SQL, Kafka consumers, M2M flows, importers, jobs, new packages, tests, Jira PIN tickets, Jira ticket status (In review, Ready to Dev, Dev) and aligning it with merged PRs, SRS/SERS pages, Confluence page IDs, branch names, commits, PRs, labels, draft PRs, release/hotfix/align branches, merge strategy, Fix Version, CODEOWNERS, bug reproduction, regression tests, failing flows, security/auth bugs, schema drift, or readmodel consistency issues.
---

# PDND Backend

Never add `Co-Authored-By` or AI authorship text to commits or PR descriptions. Check the message before every commit and PR submission.

This skill tells you the project's conventions, where things are, and what to do when sources disagree. It does not replace reading the code you change.

Before the first step that needs GitHub or Jira, run the preflight in `references/workflows.md`. If a dependency is missing, explain at once how to install it.

## Which Source Wins

Use the first source that answers the question:

1. An explicit instruction from the user in this conversation.
2. The ticket, for scope and expected behavior. Do only what it says.
3. The Manuale di Integrazione GitHub-Jira (Confluence page `1900806152`) for branches, commits, PRs, and Jira metadata. CI workflows in `.github/workflows` for what is enforced.
4. The closest sibling in the code: same package first, then the same package type. Follow the dominant pattern, not typos or isolated outliers.
5. This skill.

Decide from the code when a sibling already fixes the choice (body schema, names, limits), and cite the sibling. Do not ask.

Stop and ask when two sources at the same level disagree, when no source answers, or when the choice changes scope, a public contract, or data. Never assume. Ask as a multiple choice with an open option (`references/workflows.md`, "Asking").

For implementation conventions, follow the closest sibling when this skill disagrees with the code, and report the mismatch. Existing code does not override user constraints or the ticket's expected behavior, and may contain the bug being fixed.

## Writing Rule

This rule applies to every text in this project: PRs, commits, Jira, Confluence, code comments, error messages, docs, chat, and this skill.

The reader must understand the text at the first read, without effort.

- Follow the structure the Manuale requires. Inside it, write the minimum.
- One fact per line. One idea per sentence. Short, common words.
- Say the problem first, then what changes.
- Name the concrete thing: the endpoint, the field, the state. Avoid abstract nouns such as "propagation", "support", "chain".
- Say each thing once. No preamble, closing, filler, emphasis, or hedging.
- A rewrite for clarity must come out shorter.
- Never use an em dash. Use a comma, a colon, parentheses, or two sentences. Hyphens in compound words and filenames are fine.

## Code Rules

- Run `git branch --show-current` and `git status` before starting and before every commit. The user switches branches outside this session, and stacked PRs give the same file different content. If the code does not match the expected branch, stop and report it.
- Search `commons`, `models`, `commons-test`, and `readmodel` before writing a helper, schema, type, mock, or error. Reuse what exists.
- Do not use `any`. Use `unknown` at boundaries and parse with Zod.
- Use branded IDs from `pagopa-interop-models`, not raw strings. Derive types with `z.infer`.
- End every `ts-pattern` `match()` with `.exhaustive()`. Error mappers end with `.otherwise(...)`. Use another fallback only if the file already has one. Never satisfy `.exhaustive()` with `[]`, `undefined`, or a no-op just to compile.
- Never hardcode a string when a constant or enum exists: `attributeKind.certified`, not `"CERTIFIED"`.
- Build new state immutably.
- Use `event_version: 2` for new events. Exception: the attribute domain has only V1 events.
- Command already applied: copy the closest sibling endpoint. The default in process services is a typed error (`purposeAlreadyLinkedToClient` maps to 409). Return success with no event only if the ticket, the SRS, or the sibling says so. Never emit an event when nothing changed.
- Kafka consumers and jobs must be safe to repeat, because the same event or run can arrive twice.

## Traps

- **Runtime:** check `node -v` first. The repo has no `.nvmrc`: the version is in `engines` of the root `package.json` and in the `FROM` line of each Dockerfile.
- **Infra:** Docker, Testcontainers, `pnpm infra:*`, Drizzle pull, and integration tests need direct host access. Do not try them in a sandbox first.
- **Generated files:** never edit `packages/api-clients/src/generated/*`, `packages/models/src/gen/*`, or `packages/readmodel-models/src/drizzle/schema.ts`. Edit the source and regenerate.
- **Generation runs by itself:** `turbo.json` runs `generate-model` and `generate-protobuf` before `check` and `build`, and `build` before `lint`. On the wrong branch this corrupts the local build.
- **After generating:** rebuild `pagopa-interop-api-clients`, because consumers import `dist`. Then trust `npx tsc --noEmit`, not the editor, whose cache stays stale.
- **Readmodel is async:** a process service cannot read back what it just wrote.
- **Package manager files:** `pnpm-workspace.yaml`, `pnpm-lock.yaml`, and dependency entries in `package.json` need explicit permission. Propose the diff and wait, even if `pnpm install` or a build fails.
- **Command output:** preserve the complete output of long commands. Never pipe them into `grep`, `tail`, `head`, or another filter, even after `tee`. Use `| tee <logfile>` alone, then inspect the log with a separate command.

## Where To Go

Read only the files for your task.

- Find which package owns a change: `references/package-map.md`.
- Know how one package runs and tests: `references/package-index.md` (one row per package).
- Understand the system and its vocabulary: `references/architecture.md`.
- Understand a business concept: `references/domain-context.md`.
- Change a package: the guide named in the index, under `references/package-types/`:
  - `references/package-types/process-service.md`
  - `references/package-types/bff.md`
  - `references/package-types/gateway.md`
  - `references/package-types/sql-readmodel-writer.md`
  - `references/package-types/platformstate-writer.md`
  - `references/package-types/outbound-writer.md`
  - `references/package-types/event-dispatcher.md`
  - `references/package-types/importer.md`
  - `references/package-types/scheduled-job.md`
  - `references/package-types/shared-libraries.md`
- Change OpenAPI, protobuf, events, readmodel, a field across layers, or add an endpoint: `references/change-surfaces.md`.
- Create a package: `references/scaffolding.md`, with code shapes in `references/snippets.md`.
- Naming, errors, config, feature flags: `references/code-conventions.md`.
- Auth, roles, ownership, delegation, M2M, tokens, DPoP: `references/security.md`.
- Write tests: `references/testing.md`.
- Fix a bug, regression, or failing test: `references/debugging.md`.
- Start from a Jira key, SRS, or Confluence page: `references/atlassian-context.md`.
- Write a Jira ticket: `references/jira-tickets.md`.
- Jira statuses, or align ticket statuses with their PRs: `references/jira-status.md`.
- Run a workflow (`pdnd-work`, `pdnd-align`, `pdnd-sync`, `pdnd-merge`, `pdnd-status`), check dependencies, find the user's PRs across repositories: `references/workflows.md`.
- Branch, commit, PR, labels, release, hotfix, align: `references/git-github.md`.

## Commands

Run these without asking. Test commands run from the package directory; the index says which config the package has.

```bash
pnpm check --filter <package>
pnpm lint --filter <package>
pnpm build --filter <package>
npx eslint <paths>
npx tsc --noEmit
npx vitest run test/<file>.test.ts                                                    # default
npx vitest run --config vitest.api.config.ts test/api/<file>.test.ts                  # api
npx vitest run --config vitest.integration.config.ts test/integration/<file>.test.ts  # integration
```

Add `-t "<test name>"` to run one test. Run at most five test files; beyond that you are running the suite.

Ask first, give the exact command, and say why. A whole package suite takes about 30 minutes on the user's machine:

```bash
pnpm test --filter <package>
pnpm test:api --filter <package>
pnpm test:integration --filter <package>
pnpm test
pnpm test:api
pnpm test:integration
npx vitest run
```

Other commands:

```bash
pnpm generate-model
pnpm generate-protobuf
pnpm infra:start
pnpm infra:stop
pnpm knip                                                # CI gate: unused code and dependencies
pnpm --filter pagopa-interop-api-clients lint:openapi    # CI gate: OpenAPI files
```

After a package is added or removed, or when this skill disagrees with the code, run from the repo root. `<skill-dir>` is the folder that contains this `SKILL.md`:

```bash
node <skill-dir>/scripts/skill-check.mjs generate
node <skill-dir>/scripts/skill-check.mjs check
```

The first rebuilds the index from the current branch. The second reports packages, paths, and symbols the skill names that no longer exist.

## Before Saying Done

- The branch was confirmed before the work and before committing.
- Generated files and rebuilt packages match their source.
- Every new domain error has a code, a factory, a mapper entry, and a test.
- Security changes keep authentication, role checks, ownership checks, and their tests.
- `pnpm knip` was run if exports or dependencies changed, `lint:openapi` if OpenAPI changed.
- A bug fix has a regression test, or you state why no deterministic test is possible.
- You state exactly what was run and what was not. Nothing unrun is reported as passing.
- The diff contains only what was asked.
