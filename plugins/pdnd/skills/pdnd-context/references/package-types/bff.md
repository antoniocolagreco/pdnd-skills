# Backend For Frontend (`backend-for-frontend`)

The backend of the web UI. A handler calls several services, combines their answers, reshapes them for the UI, and validates the result against a `bffApi` schema. It does not change core data, and it changes together with the frontend.

## Files, In Data Order

- `packages/api-clients/open-api/bffApi.yml`: the BFF API.
- `src/config/config.ts`: server settings and the URL of every backend.
- `src/clients/clientsProvider.ts`: `getInteropBeClients()` returns the `PagoPAInteropBeClients` bundle.
- `src/routers/<domain>Router.ts`: reads context with `fromBffAppContext(req.ctx, req.headers)`, calls the service, validates with `bffApi`, maps errors with `makeApiProblem`.
- `src/services/<domain>Service.ts`: the combining logic. Main file.
- `src/api/<domain>ApiConverter.ts`: backend shapes to UI shapes. One way only.
- `src/model/errors.ts`: the errors of the BFF.
- `src/model/modelMappingUtils.ts`: shared mapping helpers.
- `src/utilities/errorMappers.ts`: one mapper per operation.
- `src/utilities/context.ts`: `fromBffAppContext`, which carries headers to backend calls.

## Tasks

- New endpoint: `references/change-surfaces.md`.
- New data in a response: extend the converter and the service. Do not push UI shaping into process services.

## Mistakes To Avoid

- Copying backend error codes into the BFF. Forward them with `emptyErrorMapper`.
- Skipping `fromBffAppContext`, which loses auth and tracing headers.
- Changing a `bffApi` shape without treating it as a contract change for the frontend.
- Expecting a value changed by a command to appear at once: the backend readmodel updates later.

## Tests

`vitest.config.ts` runs `test/*.test.ts` with fake backend clients. `vitest.api.config.ts` runs `test/api`: permissions, validation, status, and shape.
