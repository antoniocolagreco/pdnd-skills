# Importer (`*-importer`)

A one-shot job that reads an outside source, compares it with current state, and applies the difference by calling `*-process` services through generated clients. It never writes the database directly, so business rules still run and events are still saved.

Examples, by source:

- `ipa-certified-attributes-importer`: public dataset over HTTP.
- `anac-certified-attributes-importer`: CSV over SFTP.
- `ivass-certified-attributes-importer`: zipped CSV over HTTP.
- `istat-certified-attributes-importer`, `private-certified-attributes-importer`.

`selfcare-onboarding-consumer` also drives process services, but it is a consumer: see `references/package-types/event-dispatcher.md`.

Folder names vary (`src/service` or `src/services`, `src/model` or none): follow the package.

## Files

- `src/config/config.ts`, `src/config/<source>Config.ts`: readmodel, token, process URLs, polling, and source settings.
- `src/index.ts`: see below.
- `src/services/<name>ImporterService.ts` or `src/service/processor.ts`: computes the difference, applies it, and polls.
- `src/services/openDataService.ts`, `src/service/sftpService.ts`, `src/service/fileDownloader.ts`: fetch and parse the source.
- `src/services/readModelServiceSQL.ts` or `src/service/readmodelQueriesServiceSQL.ts`: current state.
- `src/service/tenantProcessService.ts`: thin wrapper around the generated client.

```ts
const { db, cleanup } = makeDrizzleConnectionWithCleanup(config);
const correlationId = generateId<CorrelationId>();
const loggerInstance = logger({ serviceName, correlationId });

const refreshableToken = new RefreshableInteropToken(new InteropTokenGenerator(config));
await refreshableToken.init();

try {
  const registryData = await getRegistryData(/* the outside source */);
  const current = await readModelService.get<State>();
  const diff = compute<Diff>(registryData, current);

  const token = (await refreshableToken.get()).serialized;
  const headers = getInteropHeaders({ token, correlationId });
  await apply<Changes>(diff, processClient, headers, loggerInstance, {
    defaultPollingMaxRetries: config.defaultPollingMaxRetries,
    defaultPollingRetryDelay: config.defaultPollingRetryDelay,
  });
} catch (error) {
  loggerInstance.error(error);
  process.exitCode = 1;
} finally {
  await cleanup();
}
```

## Tasks

- New source field: extend the parsed row and the parser, then the difference functions.
- New importer: copy the one with the closest source and keep the read, compare, apply, wait order.

## Mistakes To Avoid

- Acting on the source alone. Every action comes from the difference, so a second run does nothing.
- Reading back from the readmodel without polling.
- Letting raw CSV or JSON reach the difference logic. Parse at the edge.
- Skipping `cleanup()` in `finally`.
- Stopping the whole run for one bad row. Log it and continue, unless the ticket says otherwise.

## Tests

Test the parser, the difference functions, and the processor with a fake process client. Include a second run that does nothing, polling, and missing or partial data.
