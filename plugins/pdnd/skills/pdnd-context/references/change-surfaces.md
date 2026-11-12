# Change Surfaces

Every layer a change must reach. The compiler catches few of these: a converter that never fills a field or a writer that never inserts a row compiles and loses data at runtime.

## HTTP Contract

The `.yml` is the source of truth. If a response field or the set of errors of an endpoint changes, the `.yml` changes in the same commit, even if you started from the code.

1. Confirm the branch: generation overwrites files from the checked-out `.yml`.
2. Edit `packages/api-clients/open-api/*.yml`.
3. Run `pnpm generate-model`, then rebuild `pagopa-interop-api-clients`.
4. Update routers, services, converters, errors, and mappers.
5. Update the Bruno request in `collections/<domain>/<Operation name>.bru`, which the Manuale checklist requires. Copy a sibling request: it uses `{{host-<domain>}}`, `Authorization: {{JWT}}`, and `X-Correlation-Id: {{correlation-id}}`.
6. Run `pnpm --filter pagopa-interop-api-clients lint:openapi`.
7. Grep for residual references to the old behavior.

## New Process Endpoint

1. Copy the closest endpoint in the same package.
2. Update OpenAPI as above.
3. Add the router handler: `fromAppContext`, `validateAuthorization`, branded params, converter, metadata header, `makeApiProblem`.
4. Add the service method, validators, event builder in `toEvent.ts`, errors, and mapper entries.
5. Read `references/security.md` if roles, ownership, or delegation are involved.
6. Add API tests and integration tests.

## New BFF Endpoint

1. Update `bffApi.yml` and regenerate.
2. Add the service method. Keep the router thin.
3. Put mapping in `src/api/*ApiConverter.ts` or `src/model/modelMappingUtils.ts`.
4. Add a client to `src/clients/clientsProvider.ts` and its URL to config if it is new.
5. Forward backend errors with `emptyErrorMapper`.
6. Test the shape, the enrichment, and backend errors.

## New M2M Endpoint

1. Read `references/security.md`. The ticket says which gateway changes, `m2m-gateway` or `m2m-gateway-v3`. If it does not, ask.
2. Update the gateway `.yml` by adding, not breaking, and regenerate.
3. Add converter, service with polling for writes, and router.
4. If event visibility changes, update `m2m-event-dispatcher`, `m2m-event-db-models`, and `m2m-event-manager`.
5. Test visibility for producer, consumer, delegate, delegator, and unrelated tenant.

## New Event

1. Add it to `packages/models/proto/v2/<domain>/` and run `pnpm generate-protobuf`. The attribute domain has only `proto/v1`, so ask before changing its events.
2. Update `packages/models/src/<domain>/*Events.ts` and the protobuf converters.
3. Add the builder in `toEvent.ts` of the process package.
4. Update every consumer, then run `pnpm check` on each: readmodel writer, platformstate writer, outbound writer, `m2m-event-dispatcher`, and the email, in-app, documents, datalake, and analytics handlers of that domain.
5. Add tests in every touched consumer.

## Readmodel Schema

The readmodel is database-first.

1. Edit `docker/readmodel-db/<domain>.sql`.
2. Generate the schema:

   ```bash
   pnpm infra:start
   cd packages/readmodel-models
   npx drizzle-kit pull --config=drizzle.config.ts
   pnpm lint:autofix
   pnpm build
   ```

3. Update `splitters.ts`, `aggregators.ts`, and `*ReadModelService.ts` in `packages/readmodel/src/<domain>/`.
4. Update the writer. Every array the splitter returns needs an insert in the writer.
5. A new FK-backed table joins the helper that updates `metadataVersion` on related tables, such as `updateTenantVersionInRelatedTable`.
6. Check `m2m-event-dispatcher`, BFF, and gateway queries if the data affects visibility or payloads.

Other DB model packages have their own setup: read their `package.json` and `drizzle.config.ts`. The root `generate-db-models` task runs only for `m2m-event-db-models`.

## New Field On An Aggregate

Walk the whole chain and do not stop when API and domain compile:

- OpenAPI schema and generated clients.
- Domain Zod schema in `packages/models/src/<domain>`.
- Protobuf message, generated output, and converters to and from V2.
- Process `apiConverter.ts`, validators, service, and `toEvent.ts`.
- Readmodel SQL, Drizzle schema, splitter, aggregator, read service, writer.
- BFF and gateway converters.
- Outbound and M2M models if the field is public.
- Tests at the lowest layer that can fail.

## New Readmodel Query

1. Add the method to `packages/readmodel/src/<domain>/*ReadModelService.ts`, or to `readModelServiceSQL.ts` of the package if only that package uses it.
2. Return the domain type or `WithMetadata<T>`.
3. Test filters, joins, pagination, missing data, and metadata.

## Notifications And Email

1. Identify the channel: in-app, email, digest, certified email, or config.
2. Add the handler in the dispatcher and update templates or payload.
3. Read recipients and context from the readmodel, using `notification-commons` first.
4. Respect admitted roles, blocklists, config, and delegation visibility.
5. Test recipients, payload, skipped cases, and repeated events.

## Outbound Models

The public event shape lives in the sibling repo `/home/ac/Projects/interop-outbound-models`. When a published protobuf changes, update it in step. Tag numbers must be identical in both repos, or the same bytes decode to different values.

1. **Development:** create branch `om-<feature>`, never commit to `main`. Push preliminary tags `<feature>-1.0`, `<feature>-1.1` so the monorepo can install them. Publication depends on the git tag, not on `version`.
2. **QA:** bump `version` to the target semver on the `om-` branch and push.
3. **Release:** merge to `main`, push the tag equal to `version`, add the `CHANGELOG.md` entry.
4. **Monorepo:** bump `@pagopa/interop-outbound-models` on the feature branch.

Do not use a pnpm dev-link: it works locally and breaks CI and QA.
