# Snippets

Shapes for HTTP packages and scripts. Consumer and job shapes are in `references/package-types/`. Read the target package first: it wins over these.

## Package Scripts

Process service:

```json
{
  "scripts": {
    "test:api": "vitest --config vitest.api.config.ts",
    "test:integration": "vitest --config vitest.integration.config.ts",
    "test": "pnpm test:api && pnpm test:integration",
    "lint": "eslint .",
    "lint:autofix": "eslint . --fix",
    "start": "tsx -r 'dotenv-flow/config' --watch ./src/index.ts",
    "build": "tsc",
    "check": "tsc --project tsconfig.check.json"
  }
}
```

Job or consumer: the same, with `"test": "vitest"` and no `test:api` or `test:integration`.

## Config Composition

```ts
import {
  CommonHTTPServiceConfig,
  EventStoreConfig,
  ReadModelSQLDbConfig,
  ApplicationAuditProducerConfig,
} from "pagopa-interop-commons";
import { z } from "zod";

const MyServiceConfig = CommonHTTPServiceConfig.and(ReadModelSQLDbConfig)
  .and(EventStoreConfig)
  .and(ApplicationAuditProducerConfig)
  .and(
    z
      .object({ MY_REQUIRED_ENV: z.string() })
      .transform((c) => ({ myRequiredEnv: c.MY_REQUIRED_ENV }))
  );

export type MyServiceConfig = z.infer<typeof MyServiceConfig>;
export const config: MyServiceConfig = MyServiceConfig.parse(process.env);
```

## HTTP App

```ts
export async function createApp(service: MyService) {
  const serviceName = modelsServiceName.MY_SERVICE;
  const app = zodiosCtx.app();

  app.disable("x-powered-by");
  app.use(healthRouter(myApi.healthApi.api));
  app.use(contextMiddleware(serviceName));
  app.use(await applicationAuditBeginMiddleware(serviceName, config));
  app.use(await applicationAuditEndMiddleware(serviceName, config));
  app.use(authenticationMiddleware(config));
  app.use(loggerMiddleware(serviceName));
  app.use(myRouter(zodiosCtx, service));
  app.use(errorsToApiProblemsMiddleware);

  return app;
}
```

## HTTP Index

```ts
const db = makeDrizzleConnection(config);
const readModelServiceSQL = readModelServiceBuilderSQL(db);
const service = myServiceBuilder(
  initDB({
    username: config.eventStoreDbUsername,
    password: config.eventStoreDbPassword,
    host: config.eventStoreDbHost,
    port: config.eventStoreDbPort,
    database: config.eventStoreDbName,
    schema: config.eventStoreDbSchema,
    useSSL: config.eventStoreDbUseSSL,
  }),
  readModelServiceSQL
);

startServer(await createApp(service), config);
```

## Router Handler

```ts
router.post("/resources/:resourceId/action", async (req, res) => {
  const ctx = fromAppContext(req.ctx);

  try {
    validateAuthorization(ctx, [authRole.ADMIN_ROLE, authRole.API_ROLE]);

    const { data, metadata } = await service.doAction(
      unsafeBrandId(req.params.resourceId),
      req.body,
      ctx
    );

    setMetadataVersionHeader(res, metadata);
    return res.status(200).send(myApi.Resource.parse(toApiResource(data)));
  } catch (error) {
    const errorRes = makeApiProblem(error, doActionErrorMapper, ctx);
    return res.status(errorRes.status).send(errorRes);
  }
});
```

## Event Builder

```ts
export const toCreateEventResourceUpdated = (
  resource: Resource,
  version: number,
  correlationId: CorrelationId
): CreateEvent<ResourceEvent> => ({
  streamId: resource.id,
  version,
  event: {
    type: "ResourceUpdated",
    event_version: 2,
    data: { resource: toResourceV2(resource) },
  },
  correlationId,
});
```

A new stream passes `version: undefined`. An existing stream passes the current metadata version.

## Error Factory And Mapper

```ts
const errorCodes = {
  resourceNotFound: "0001",
  invalidResourceState: "0002",
};

export type ErrorCodes = keyof typeof errorCodes;
export const makeApiProblem = makeApiProblemBuilder(errorCodes);

export function resourceNotFound(id: ResourceId): ApiError<ErrorCodes> {
  return new ApiError({
    detail: `Resource ${id} not found`,
    code: "resourceNotFound",
    title: "Resource not found",
  });
}
```

```ts
export const doActionErrorMapper = (
  error: ApiError<LocalErrorCodes | CommonErrorCodes>
): number =>
  match(error.code)
    .with("resourceNotFound", () => constants.HTTP_STATUS_NOT_FOUND)
    .with("invalidResourceState", () => constants.HTTP_STATUS_BAD_REQUEST)
    .with("operationForbidden", () => constants.HTTP_STATUS_FORBIDDEN)
    .otherwise(() => constants.HTTP_STATUS_INTERNAL_SERVER_ERROR);
```
