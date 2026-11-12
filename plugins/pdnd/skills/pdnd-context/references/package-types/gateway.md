# Gateways (`api-gateway`, `m2m-gateway`, `m2m-gateway-v3`)

The public APIs for outside companies (`m2m` means machine-to-machine). They must stay stable, because integrators cannot be updated on demand.

This guide describes the two M2M gateways. `api-gateway` differs: no polling, no `m2mAuthDataValidationMiddleware`, `src/models/errors.ts`, `src/utilities/*`, and one `vitest.config.ts`.

## Files, In Data Order

- `packages/api-clients/open-api/m2mGatewayApi.yml`, `m2mGatewayApiV3.yml`, `apiGatewayApi.yml`: the public APIs.
- `src/config/config.ts`: server, backend URLs, rate limit, polling, and token settings.
- `src/app.ts`: the middleware stack below.
- `src/clients/clientsProvider.ts`: backend clients.
- `src/routers/<domain>Router.ts`: checks permissions, calls the service, returns the public shape.
- `src/services/<domain>Service.ts`: calls the backend, then waits until the change is visible.
- `src/api/<domain>ApiConverter.ts`: backend shapes to public shapes.
- `src/utils/polling.ts`: `pollResourceWithMetadata`, `pollResourceUntilDeletion`.
- `src/utils/validators/*`: `assert*` preconditions.
- `src/utils/fileDownload.ts`, `src/utils/fileUpload.ts`, `src/utils/context.ts`: files and context.
- `src/model/errors.ts`, `src/utils/errorMappers.ts`: errors of the gateway.

```ts
app.use(multerMiddleware);
app.use(fromFilesToBodyMiddleware);
app.use(appBasePath,
  healthRouter(...), contextMiddleware(serviceName, false),
  await applicationAuditBeginMiddleware(...), await applicationAuditEndMiddleware(...),
  authenticationMiddleware(config),
  m2mAuthDataValidationMiddleware(clientService),
  rateLimiterMiddleware,
  <domain>Router(zodiosCtx, <domain>Service), /* ... */);
app.use(errorsToApiProblemsMiddleware);
```

## Waiting For The Readmodel

Outside callers expect to read back what they changed:

```ts
const pollResource = (response: WithMaybeMetadata<<Domain>>, headers) =>
  pollResourceWithMetadata(() => retrieve<Entity>ById(headers, response.data.id))({
    condition: isPolledVersionAtLeastResponseVersion(response),
  });

async create<Entity>(body, { headers }: WithLogger<<Gateway>AppContext>) {
  const created = await clients.<domain>ProcessClient.create<Entity>({ body, headers });
  const polled = await pollResource(created, headers);
  return to<Gateway>Api<Entity>(polled.data);
}
```

Steps for a new endpoint are in `references/change-surfaces.md`.

## Mistakes To Avoid

- Breaking the API. Add to it, or make a new version.
- Returning from a write before polling.
- Skipping auth or rate limiting on a new route.
- Leaking internal error codes. Translate them to the documented public errors.
- Handling uploads and downloads by hand.

## Tests

Use the config that matches the test folder (see `references/package-index.md`). Check the public shape, polling, caller permissions, rate limiting, and error mapping, with fake backend clients.
