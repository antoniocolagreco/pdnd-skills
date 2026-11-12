# SQL Readmodel Writer (`*-readmodel-writer-sql`)

A Kafka consumer that turns events into rows in the readmodel tables. No business rules, no HTTP.

Examples: `catalog-readmodel-writer-sql`, `agreement-readmodel-writer-sql`, `tenant-readmodel-writer-sql`.

## Files

Shared, outside the package:

- `docker/readmodel-db/*.sql`: table definitions, columns in `snake_case`.
- `packages/readmodel-models/src/drizzle/schema.ts`: generated tables, named `<entity>InReadmodel<Domain>`.
- `packages/readmodel/src/<domain>/splitters.ts`: `split<Entity>IntoObjectsSQL(entity, metadataVersion)` returns the rows for each table.

In the package:

- `src/config/config.ts`: `ReadModelWriterConfigSQL.and(<Domain>TopicConfig)`.
- `src/index.ts`: see below.
- `src/consumerServiceV2.ts`: `handleMessageV2` matches on event `type`.
- `src/consumerServiceV1.ts`: the same for V1, only in domains with old events.
- `src/<domain>WriterService.ts`: `upsert*` and `delete*` methods. `notification-config-readmodel-writer-sql` uses `src/readModelWriteService.ts`.

`attribute-registry-readmodel-writer-sql` is V1 only and uses `src/attributeRegistryConsumerService.ts`.

```ts
const db = makeDrizzleConnection(config);
const writerService = <domain>WriterServiceBuilder(db);

async function processMessage({ message, partition }: EachMessagePayload): Promise<void> {
  const decodedMessage = decodeKafkaMessage(message, <Domain>Event);
  const loggerInstance = logger({ serviceName, eventType: decodedMessage.type, /* ...ids */ });

  await match(decodedMessage)
    .with({ event_version: 1 }, (msg) => handleMessageV1(msg, writerService))
    .with({ event_version: 2 }, (msg) => handleMessageV2(msg, writerService))
    .exhaustive();

  loggerInstance.info(`Read model was updated. Partition: ${partition}. Offset: ${message.offset}`);
}

await runConsumer(config, [config.<domain>Topic], processMessage, "<service-name>");
```

```ts
export async function handleMessageV2(message: <Domain>EventEnvelopeV2, writerService: <Domain>WriterService): Promise<void> {
  await match(message)
    .with({ type: "<Entity>Deleted" }, async (msg) =>
      writerService.delete<Entity>ById(unsafeBrandId(msg.stream_id), msg.version))
    .with({ type: "<Entity>Added" }, { type: "<Entity>Updated" }, async (msg) => {
      const entity = msg.data.<entity>;
      if (!entity) {
        throw missingKafkaMessageDataError("<entity>", msg.type);
      }
      return writerService.upsert<Entity>(from<Entity>V2(entity), msg.version);
    })
    .exhaustive();
}
```

```ts
async upsert<Entity>(entity: <Entity>, metadataVersion: number): Promise<void> {
  await db.transaction(async (tx) => {
    const shouldUpsert = await checkMetadataVersion(tx, <entity>InReadmodel<Domain>, metadataVersion, entity.id);
    if (!shouldUpsert) { return; }

    await tx.delete(<entity>InReadmodel<Domain>).where(eq(<entity>InReadmodel<Domain>.id, entity.id));
    const { <entity>SQL, ...childRowsSQL } = split<Entity>IntoObjectsSQL(entity, metadataVersion);
    await tx.insert(<entity>InReadmodel<Domain>).values(<entity>SQL);
  });
}
```

Delete methods add `lte(table.metadataVersion, metadataVersion)` so an old event cannot undo a newer change.

## Tasks

- New event type: add a case in `handleMessageV2`. `.exhaustive()` does not compile until every type has one.
- New column or table: `references/change-surfaces.md`.

## Mistakes To Avoid

- Writing without `checkMetadataVersion` and `lte`: Kafka can deliver an event twice.
- Forgetting to raise `metadataVersion` on related tables.
- A splitter array with no insert in the writer: rows are lost silently.
- A do-nothing case added only to compile. Ignore an event only on purpose, with a comment.
- Skipping a missing event payload. Throw `missingKafkaMessageDataError`.

## Tests

`test/utils.ts` builds the writer against containers, and `test/vitestGlobalSetup.ts` starts the database. Build events with `to<Entity>V1` or `to<Entity>V2` and mock factories, run them through the handlers, and check the rows. Always test that a repeated event creates no duplicate and does not undo newer data.
