# Code Conventions

The rules in `SKILL.md` come first. This file adds naming, errors, config, and tooling.

## TypeScript And Readability

- Code a junior cannot follow is badly written. Prefer plain steps, named intermediate values, and guard clauses to clever one-liners and chained ternaries.
- Duplicate twice before extracting a shared helper.
- A helper used by one function lives inside that function, so it reads the parent's values directly. Lift it when a second, unrelated caller needs it.
- Zod domain properties are `camelCase`. `snake_case` appears only in SQL columns and in the first argument of Drizzle's `timestamp("column_name", ...)`.
- A union alias used by two or more packages belongs in `packages/models/src`.

## Naming

Before naming an `operationId`, service method, or error mapper, grep the siblings and follow the dominant pattern, such as `maintenanceTenant{Verb}`. Ignore typos and isolated outliers.

M2M v3 reject bodies are named `<Entity>Rejection` and mirror the field name and limits of the process payload: `AgreementRejection`, `DelegationRejection`.

## Scope

- A single-verb instruction ("rebase", "push", "commit") means that verb only. If a prerequisite is missing, ask.
- Mention a related improvement in one sentence and wait. Do not make it.
- Before applying a convention suggested in review, grep the codebase to confirm it. AI reviewers see only the diff and can be wrong about what siblings do.
- Verify each step. Batch verification at the end only if the user says so.

## Services

Service methods stay flat, in this order: log with IDs, load state, resolve delegation, check ownership, validate, build new state, create the event with `toEvent.ts`, persist with `repository.createEvent()`, return `WithMetadata<Entity>`.

Put branching and domain rules in validators. `assert*` functions throw domain errors; predicates return booleans.

Process services do not contain the other layers' logic: see `references/package-types/process-service.md`.

## Converters

- Process converters go both ways: domain to API and API to domain.
- BFF converters go one way and may flatten, group, enrich, or rename.
- When a union, enum, state, or field changes, update every converter in the same change.
- A hardcoded default for a field in the OpenAPI schema (`field: []`) hides a missing domain field. Model the field, or add a comment saying why the default is intentional.

Protobuf enums: the first member is `ENUM_NAME_UNSPECIFIED = 0`, and real values start at `1`. Events are persisted, so this encoding is permanent. `from*V2` converters throw `genericInternalError` on the sentinel. `to*V2` converters never emit it.

## Errors

Domain errors live in `src/model/domain/errors.ts` (managers, BFF, and M2M gateways use `src/model/errors.ts`, `api-gateway` uses `src/models/errors.ts`).

- Add numeric codes only. Never renumber or reuse one.
- One exported factory per error. The factory name equals the `errorCodes` key.
- `detail` names the entity, the IDs, and the failed condition.
- Expected outcomes map to 4xx. Unexpected and infrastructure failures map to 500.
- A new error needs a mapper entry and a test.
- Do not use `new Error()` for business failures.
- BFF mappers handle only errors the BFF raises. Use `emptyErrorMapper` to forward a backend error.

Status codes: 401 for missing or bad credentials, 403 for a forbidden role or ownership, 400 for invalid input, 404 for a missing resource, 409 for a state conflict.

**Version conflicts:** `repository.createEvent()` throws `eventConflictError` (409) when another write already took that version. The process rolls back, so repeating the call is safe. In the BFF, use `retryOnEventConflict` from `src/utilities/eventConflict.ts` when the ticket asks for it. Other 409 responses are business conflicts and are never retried.

## Config

- Parse the environment with Zod in `src/config/config.ts`. Compose schemas from `commons` with `.and()`.
- Convert env vars to `camelCase` with `.transform`.
- Never read `process.env` outside config.

## Feature Flags

Every new feature goes behind a feature flag, unless something prevents it (Confluence page `2136113416`, section 3.1). If the ticket or the SRS does not name the flag, ask.

1. Add `FeatureFlag<Name>Config` to `packages/commons/src/config/featureFlagsConfig.ts`, with env var `FEATURE_FLAG_<NAME>`, default `false`, and key `featureFlag<Name>`. CODEOWNERS restricts this file to the core auth team.
2. Compose it with `.and(FeatureFlag<Name>Config)` in the config of each package that reads it.
3. Add `FEATURE_FLAG_<NAME>=true` to the `.env` of that package.
4. Read it with `isFeatureFlagEnabled(config, "featureFlag<Name>")`, or reject with `assertFeatureFlagEnabled`.

For tests, see `references/testing.md`.

## Tooling

The root `tsconfig.json` extends `@tsconfig/node-lts` and `@tsconfig/strictest`. Biome uses 2-space indentation, 80-column lines, and ES5 trailing commas. Packages lint with ESLint through `pagopa-interop-eslint-config`.
