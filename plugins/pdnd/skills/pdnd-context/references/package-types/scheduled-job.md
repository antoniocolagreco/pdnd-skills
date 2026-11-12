# Scheduled Or One-Shot Job

A program that does one batch task and exits. The schedule lives in the deployment system, not in the package. This guide covers every package marked `job` in `references/package-index.md`, except importers.

Examples:

- Cleanup: `m2m-event-cleaner`, `in-app-notification-cleaner`.
- Export: `dtd-catalog-exporter`, `datalake-data-export`.
- Check: `token-generation-readmodel-checker`, `kpi-domains-readmodel-checker`.
- Archive: `eservice-descriptors-scheduled-archiver`.
- Digest: `email-digest-dispatcher`.

The suffix is not enough: `delegation-items-archiver` and `datalake-interface-exporter` are consumers.

## Files

- `src/config/config.ts` (`src/configs/config.ts` in the checkers): store settings, limits, batch sizes.
- `src/index.ts`: wiring and the run lifecycle.
- `src/services/<job>Service.ts` or a task file such as `src/deleteOldM2MEvents.ts`: the task, as a function of its dependencies.
- `src/services/readModelServiceSQL.ts`: current state.
- `src/models/*`, `src/utils/*`: local types and helpers.

```ts
const { db, cleanup } = makeDrizzleConnectionWithCleanup(config);
const loggerInstance = logger({ serviceName, correlationId: generateId<CorrelationId>() });

try {
  await <job>ServiceBuilder({ /* dependencies */, loggerInstance }).run<Task>();
} catch (error) {
  loggerInstance.error(error);
  process.exitCode = 1;
} finally {
  await cleanup();
}
```

Many existing jobs call `process.exit(1)` in the `catch` instead. That also signals the failure, but `finally` does not run. Copy the closest sibling.

## Tasks

- New step: add it to the service function, not to `index.ts`.
- New limit: an env var in `config.ts`.
- New consistency check: read both sides in the service, compare deterministically, report the difference.

## Mistakes To Avoid

- Leaking connections: close them in `finally`.
- Hiding logic in `index.ts`, where it cannot be tested.
- A rerun that is not safe. A job can be retried.
- Exiting with success after a failure, so the scheduler records a false success.
- Loading unbounded rows. Use the configured batch size.

## Tests

Run the service function against a test store or a fake client. Check the effect, a repeated run, and the edge cases: nothing to do, partial data.
