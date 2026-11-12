# Event Dispatcher And Consumer

A consumer that reacts to events and produces something new: notifications, emails, M2M rows, analytics, signed documents, or calls to process services. This guide covers every package marked `consumer` in `references/package-index.md` that has no more specific guide, including most `-updater`, `-archiver`, and `-persister` packages.

Examples: `in-app-notification-dispatcher`, `m2m-event-dispatcher`, `email-notification-dispatcher`, `client-purpose-updater`, `delegation-items-archiver`.

Check `src/index.ts`: most use `runConsumer` from `kafka-iam-auth`, `token-details-persister` and `events-signer` use `runBatchConsumer`, and `audit-signer` and `application-audit-fallback` read from SQS.

## Files

- `src/config/config.ts`: Kafka config, one topic config per domain, readmodel config, output config.
- `src/index.ts`: picks a handler by topic, decodes, runs it, and saves the results.
- `src/handlers/<domain>/handle<Domain>Event.ts`: matches on `event_version` and `type`. `m2m-event-dispatcher` keeps handlers directly in `src/handlers/`.
- `src/handlers/<domain>/handle<Case>To<Audience>.ts`: one rule per file. The audience logic lives here.
- `src/services/readModelServiceSQL.ts`: finds recipients and context.
- `src/services/<output>ServiceSQL.ts`: writes to the package's own database.
- `src/models/*`: local types and, in M2M, event adapters.

```ts
function processMessage(topicNames: TopicNames) {
  return async (payload: EachMessagePayload): Promise<void> => {
    const handleWith = (eventSchema, handler) => {
      const decoded = decodeKafkaMessage(payload.message, eventSchema);
      const loggerInstance = logger({ serviceName, eventType: decoded.type, /* ...ids */ });
      return handler(decoded, loggerInstance, readModelService);
    };

    const results = await match(payload.topic)
      .with(agreementTopic, () => handleWith(AgreementEvent, handleAgreementEvent))
      .otherwise(() => { throw genericInternalError(`Unknown topic: ${payload.topic}`); });

    await outputService.persist(results);
  };
}
```

```ts
export async function handle<Domain>Event(decoded, logger, readModelService): Promise<<Result>[]> {
  return match(decoded)
    .with({ event_version: 1 }, () => { logger.info(`Skipping V1 event ${decoded.type}`); return []; })
    .with({ type: P.union("<Type1>", "<Type2>") }, (msg) => handle<Case>(msg, logger, readModelService))
    .otherwise(() => []);
}
```

## Tasks

- React to a new event: add a case and a new `handle<Case>To<Audience>.ts`.
- Listen to a new topic: topic config, read helper, case in `index.ts`, topic in `runConsumer`, new handler.
- Change who receives something: edit only that rule file.

## Mistakes To Avoid

- Letting V1 events reach V2 logic. Skip them explicitly. Exception: attribute events exist only as V1, and `m2m-event-dispatcher` handles them with `fromAttributeV1`.
- A wrong audience, which leaks information. Work out producer, consumer, delegate, and user from the readmodel.
- Repeats creating duplicates. Use a unique key or an upsert.
- Assuming the readmodel already shows the change in the event you are processing.

## Tests

Pass events through the handlers with prepared readmodel data. Check the audience, the content, skipped V1 events, irrelevant events, and that a repeated event creates no duplicate.
