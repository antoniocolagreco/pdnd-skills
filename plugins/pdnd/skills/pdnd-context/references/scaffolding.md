# New Package

There is no generator. Copy the closest package of the same kind, then trim. Find one in `references/package-index.md` and read its guide in `references/package-types/`.

## Checklist

1. `packages/<name>/package.json`: name `pagopa-interop-<name>`, version `1.0.0`, license `MPL-2.0`, `"private": true`, `"main": "dist"`, `"type": "module"`, and scripts from `references/snippets.md`.
2. `tsconfig.json`:

   ```json
   {
     "extends": "../../tsconfig.json",
     "compilerOptions": { "outDir": "dist" },
     "include": ["src"]
   }
   ```

3. With tests, `tsconfig.check.json` and `test/tsconfig.json`:

   ```json
   {
     "extends": "../../tsconfig.json",
     "compilerOptions": { "noEmit": true },
     "include": ["test"]
   }
   ```

   ```json
   { "extends": "../tsconfig.json", "include": ["."] }
   ```

4. `eslint.config.mjs`:

   ```js
   import node from "pagopa-interop-eslint-config/node";

   export default node;
   ```

5. Copy `.env`, `AUTHORS`, and `LICENSE` from the closest package.
6. `src/index.ts` and the modules of the package type.
7. `Dockerfile` for a deployable package (rules below).
8. A `start:<name>` script in the root `package.json`, if it needs to run locally.
9. A new package changes `pnpm-lock.yaml`, which needs permission. Propose the diff and wait before running `pnpm install` or committing it.
10. `pnpm build --filter <package>`, `pnpm check --filter <package>`, `pnpm lint --filter <package>`, and focused tests.
11. Add the package to `references/package-map.md` and regenerate the index with `scripts/skill-check.mjs`.

## By Kind

**Process service:** OpenAPI, then `packages/models` for a new aggregate or event, then errors, converters, validators, `toEvent.ts`, service, router, `app.ts`, `index.ts`, tests.

**Manager or gateway:** no event store. Use it for HTTP surfaces that do not own events. Do not create a process service for read-only aggregation or scheduled work.

**SQL readmodel writer:** SQL tables, Drizzle generation, then `packages/readmodel/src/<domain>/` (splitters, aggregators, read service), then the writer service with a version check, then `consumerServiceV2.ts`. New domains are V2 only.

**Consumer or dispatcher:**

1. Compose `KafkaConsumerConfig`, one `<Domain>TopicConfig` per topic (`CatalogTopicConfig`), and DB or API configs.
2. Call `runConsumer(config, [topic], processMessage, serviceName)` in `index.ts`.
3. Decode with `decodeKafkaMessage(message, DomainEvent)`.
4. Log `eventType`, `eventVersion`, `streamId`, `streamVersion`, `correlationId`.
5. Dispatch with `match` on `event_version` and `type`.
6. Make processing safe to repeat.

**Importer or job:**

1. Parse config, generate a `CorrelationId`, build the logger, create clients in `index.ts`.
2. Put the work in a service function, not in `index.ts`.
3. Catch and log at the top level, set `process.exitCode = 1`, and close connections in `finally`.
4. An importer parses external data with Zod, computes what to create, update, or revoke against current state, and calls process APIs through generated clients. It polls the readmodel only when a later step needs the result.

## Dockerfile

Copy from the closest deployable package and keep its `NODE_REGISTRY`, pinned Node image, `corepack enable`, `/out` copy, and `touch --date=@0` lines.

It must include:

- Root `package.json`, `pnpm-lock.yaml`, `pnpm-workspace.yaml`, `.npmrc`, then `pnpm install --frozen-lockfile`.
- `package.json` and source of this package and of every internal dependency.
- Root `tsconfig.json` and `turbo.json`, then `pnpm build`.
- Runtime copy of `node_modules`, the package `dist`, and internal dependency folders.
- Non-root `app` user, `WORKDIR /app/packages/<package>`, `CMD ["node", "."]`.
