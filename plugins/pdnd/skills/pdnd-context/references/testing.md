# Testing

Commands and permission rules are in `SKILL.md`. The Vitest configs of each package are in `references/package-index.md`.

## Which Test

- Validator, converter, mapper, state transition: unit test.
- Status, auth, request validation, headers, Problem payload: API test (`test/api`).
- Business rule, saved event, version, readmodel query: integration test (`test/integration`).
- Already-applied command: test the behavior its sibling has, error or no-op.
- Projection: integration test, including the same event delivered twice.
- BFF shape and enrichment: converter and service tests.
- M2M visibility, delegation, emitted rows: dispatcher and manager tests.
- Importer or job: repeated run, duplicates, missing external data, partial failure.

## API Tests

`test/vitest.api.setup.ts` replaces the audit and auth middleware and exports `api` and the mocked service.

```ts
import { generateToken, getMockDelegation, getMockWithMetadata } from "pagopa-interop-commons-test";
import request from "supertest";
import { api, delegationService } from "../vitest.api.setup.js";

const makeRequest = async (token: string) =>
  request(api)
    .post(`/consumer/delegations/${id}/approve`)
    .set("Authorization", `Bearer ${token}`)
    .set("X-Correlation-Id", generateId())
    .send();

it.each(authorizedRoles)("returns 200 for role %s", async (role) => {
  const res = await makeRequest(generateToken(role));
  expect(res.status).toBe(200);
  expect(res.headers["x-metadata-version"]).toBe(version.toString());
});
```

Cover allowed roles, forbidden roles (`Object.values(authRole)` minus the allowed ones), invalid input, each mapped error, and the metadata header.

## Integration Tests

`test/integrationUtils.ts` starts the containers with `setupTestContainersVitest`, builds the real service, and runs `afterEach(cleanup)`. `test/vitestIntegrationGlobalSetup.ts` calls `setupTestContainersVitestGlobal`.

Helpers from `pagopa-interop-commons-test`:

- Context: `getMockContext`, `getMockContextM2M`, `getMockContextM2MAdmin`, `getMockContextInternal`, `getMockContextMaintenance`, `getMockAuthData`.
- Entities: `getMock<Entity>` (`getMockEService`, `getMockDescriptor`, `getMockTenant`, `getMockAgreement`, `getMockPurpose`) and `getMockWithMetadata`.
- API payloads: the `getMocked*` factories in `packages/commons-test/src/apiMocks.ts`.
- Event store: `writeInEventstore`, `readLastEventByStreamId`, `decodeProtobufPayload`.
- Tokens: `generateToken(role)`.

Fill the readmodel with the `upsert*` helpers from `pagopa-interop-readmodel/testUtils`.

Check the saved event type and payload, the new version, and the expected errors.

## Style

- Test the business outcome, not call counts.
- Use fixed clocks and IDs when time or randomness matters. No sleeps.
- One test verifies one thing. Repeated setup is acceptable; merged concerns are not.
- Cover the happy path and the negative path that justifies the change.
- Annotate spread-built objects so the compiler checks them: `const x: attributeRegistryApi.Attribute = { ...getMockedApiAttribute(), id };`. Reviewers enforce this.
- A new OpenAPI response field needs a test that parses the converter output with its schema: `SomeApi.SomeType.parse(converter(domainObject))`. This catches a field the converter never fills.

## Feature Flags

A test of a flagged feature sets the flag in `beforeEach`: `config.featureFlag<Name> = true`. Without it, inherited tests run the "off" path and pass for the wrong reason. After adding a flag on a branch, check every integration test that touches the feature.

After `git rebase --onto` on a stacked branch, target-branch tests do not know downstream flags. Ask to run the full suite of the touched package.
