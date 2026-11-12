# Platform State Writer (`*-platformstate-writer`)

A Kafka consumer that writes to DynamoDB the data needed to issue access tokens. It keeps two sets of records in step: platform-states and token-generation-states. No business logic, no HTTP.

Examples: `catalog-platformstate-writer`, `purpose-platformstate-writer`, `agreement-platformstate-writer`, `authorization-platformstate-writer`.

## Files

- `src/config/config.ts`: `PlatformStateWriterConfig.and(<Domain>TopicConfig)`.
- `src/index.ts`: creates `new DynamoDBClient()`, decodes each event, and dispatches on `event_version`. Same shape as the SQL writer.
- `src/consumerServiceV2.ts`, `src/consumerServiceV1.ts`: match on event `type`.
- `src/utils.ts`: builds keys, reads and writes both record sets.

```ts
.with({ type: "<Entity>Published" }, async (msg) => {
  const { entity } = parse<Entity>(msg.data.<entity>, msg.type);
  const pk = makePlatformStates<Entity>PK({ /* ids */ });
  const existing = await readEntry(pk, dynamoDBClient);
  if (existing && existing.version > msg.version) { return; }

  await upsertPlatformStatesEntry(/* ... */);
  await updateInfoInTokenGenerationStatesTable(/* ... */);
})
```

## Tasks

- New event type: add a case. If it does not affect tokens, add it to the do-nothing case on purpose.
- New record shape: update the key builder and type in `models`, then both write paths in `utils.ts`.

## Mistakes To Avoid

- Skipping the version check: an older event must not overwrite a newer record.
- Updating only one of the two record sets, which makes token issuing inconsistent.
- Building keys by hand. Use the `makePlatformStates*PK` and `makeGSIPK*` functions from `models`.
- Assuming transactions: DynamoDB has none across tables here. Rely on the version check.

## Tests

Pass events through the handlers against the DynamoDB test setup. Check both record sets, that an older event is ignored, and the do-nothing events.
