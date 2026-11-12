# Shared Libraries

Search here before writing any helper, type, error, config, mock, or ID. Import from the package entry point, never from a deep path. If something is close to what you need, extend it in its own style instead of copying it. A change here affects dependents: run their `pnpm check`.

## models (`pagopa-interop-models`)

Domain types, IDs, events, protobuf converters, errors, service names.

- `brandedIds.ts`: branded IDs, `unsafeBrandId`, `generateId`.
- `errors.ts`: `ApiError`, `makeApiProblemBuilder`, `genericInternalError`, `missingKafkaMessageDataError`, `operationForbidden`, `eventConflictError`, `emptyErrorMapper`.
- `constants.ts`, `utils.ts`, `services.ts`: constants, helpers, `serviceName`.

Each domain folder (`agreement`, `attribute`, `eservice`, `eservice-template`, `purpose`, `purpose-template`, `tenant`, `authorization`, `delegation`, `notification-config`, `m2m-event`, and others) contains:

- `<domain>.ts`: Zod schemas and types.
- `<domain>Events.ts`: event unions.
- `protobufConverterFromV1.ts`, `protobufConverterFromV2.ts`, `protobufConverterToV2.ts`. `attribute` has only `protobufConverterFromV1.ts` and `protobufConverterToV1.ts`.

Also: `events/`, `protobuf/`, `read-models/`, `token-generation-readmodel/` (`makePlatformStates*PK`), `risk-analysis/`.

## commons (`pagopa-interop-commons`)

- `auth/`: `authenticationMiddleware`, `validateAuthorization`, `authRole`, `setMetadataVersionHeader`, JWT and JWK.
- `config/`: Zod config pieces such as `CommonHTTPServiceConfig`, `EventStoreConfig`, `ReadModelSQLDbConfig`, `ReadModelWriterConfigSQL`, `PlatformStateWriterConfig`, `KafkaConsumerConfig`, `KafkaProducerConfig`, topic configs, feature flags.
- `context/`: `fromAppContext`, `zodiosCtx`, `getInteropHeaders`.
- `logging/`: `logger`, `loggerMiddleware`.
- `repositories/`: `eventRepository`, `initDB`.
- `kafka/`: `decodeKafkaMessage`.
- `router/`: `healthRouter`, `startServer`, error-to-response middleware, `multerMiddleware`, `fromFilesToBodyMiddleware`.
- `file-manager/` (`initFileManager`), `sqs/`, `dynamo-db-client/`, `safe-storage/`.
- `rate-limiter/`, `interop-token/` (`InteropTokenGenerator`, `RefreshableInteropToken`), `email-manager/`, `templating/`, `risk-analysis/`, `eservice-documents/`, `notification/`.
- `utils/`: `date`, `arrays`, `delay` (`retry`), `getAll`, `getAllFromPaginated`, `createPollingByCondition`, `apiUtils`.
- `types/`: `Exact`.

## commons-test (`pagopa-interop-commons-test`)

Test helpers: see `references/testing.md`. Also `setupDynamoDBtables.ts` for platform state tests, and `protobufConvertersToV1/` for V1 events. Extend the `getMock*` factories instead of hand-making fixtures.

## api-clients (`pagopa-interop-api-clients`)

- `open-api/*.yml`: API definitions. Edit these.
- `src/generated/*`: generated clients and schemas.
- `src/<service>Api.ts`: wrappers with `createProcessApiClient` and `QueryParametersByAlias`. This is what services import.
- `src/zodiosWithMetadata.ts`: `WithMaybeMetadata`.

## readmodel (`pagopa-interop-readmodel`)

One folder per domain:

- `splitters.ts`: `split<Entity>IntoObjectsSQL`, one entity into table rows.
- `aggregators.ts`: `aggregate<Entity>*`, rows into one entity.
- `<entity>ReadModelService.ts`: read helpers. Reuse these instead of writing SQL.

Also `utils.ts` (`checkMetadataVersion`, `makeDrizzleConnection`, `makeDrizzleConnectionWithCleanup`) and `testUtils.ts` (`upsert*`).

## readmodel-models (`pagopa-interop-readmodel-models`)

The generated `src/drizzle/schema.ts`, with tables named `<entity>InReadmodel<Domain>`, plus connection and transaction types.

## Others

- `kafka-iam-auth`: `runConsumer`, `runBatchConsumer`, `initProducer`.
- `eslint-config`: shared ESLint settings.
- `agreement-lifecycle`: shared agreement rules and filters.
- `notification-commons`: recipients, retrievers, scheduled delivery.
- `client-assertion-validation`, `dpop-validation`: auth checks.
- `application-audit`: audit middleware.
- `kpi-models`: KPI models.
- `m2m-event-db-models`, `in-app-notification-db-models`, `scheduled-notification-db-models`, `tenant-kind-history-db-models`: tables of dedicated stores.
