# Outbound Writer (`*-outbound-writer`)

A Kafka consumer that republishes internal events in a public form on a separate topic. It decides which events are public and which fields are safe. No business logic, no HTTP.

The public shape lives in another repository, `@pagopa/interop-outbound-models`. For that repo's workflow, see "Outbound Models" in `references/change-surfaces.md`.

Examples: `catalog-outbound-writer`, `agreement-outbound-writer`, `purpose-outbound-writer`.

## Files

- `src/config/config.ts`: `KafkaConsumerConfig.and(KafkaProducerConfig).and(<Domain>TopicConfig)` plus `<DOMAIN>_OUTBOUND_TOPIC`.
- `src/index.ts`: see below.
- `src/converters/toOutboundEventV2.ts`: internal to public, `undefined` for events not published.
- `src/converters/toOutboundEventV1.ts`: the same for V1, only where V1 events exist.

```ts
const producer = await initProducer(config, config.<domain>OutboundTopic);

async function processMessage({ message, partition }: EachMessagePayload): Promise<void> {
  const msg = decodeKafkaMessage(message, <Domain>Event);

  const outboundEvent: <Domain>OutboundEvent | undefined = match(msg)
    .with({ event_version: 1 }, (m) => toOutboundEventV1(m))
    .with({ event_version: 2 }, (m) => toOutboundEventV2(m))
    .exhaustive();

  if (outboundEvent) {
    await producer.send({ messages: [{ key: outboundEvent.stream_id, value: encodeOutbound<Domain>Event(outboundEvent) }] });
  }
}
```

```ts
function toOutbound<Entity>V2(entity: <Entity>V2): Exact<Outbound<Entity>V2, <Entity>V2> {
  return { ...entity, path: undefined, riskAnalysis: undefined };
}
```

`Exact<Outbound, Internal>` makes a new internal field a compile error here until you expose or strip it.

## Tasks

- Expose a field or event: change `interop-outbound-models`, then the converter here.
- Keep an event internal: add it to the `match` case that returns `undefined`.

## Mistakes To Avoid

- Leaking document paths, risk analyses, or other private fields.
- Different protobuf field numbers across the two repositories, which corrupts what outside systems decode.
- Landing the model change and the writer change separately.

## Tests

These packages have no test folder, Vitest config, or `check` script. Their only automatic checks are `tsc` with `Exact<>` and the exhaustive `match`. Report that runtime behavior is not tested. Adding tests is new infrastructure, so ask first.
