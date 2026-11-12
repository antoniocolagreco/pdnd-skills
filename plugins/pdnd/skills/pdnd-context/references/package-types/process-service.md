# Process Service (`*-process`)

The command side. It turns a valid request into a saved event. It does not write read tables: a writer does that later.

An **aggregate** is the entity it manages with everything that belongs to it, such as an e-service with its descriptors and documents. One process package owns one aggregate.

Examples: `catalog-process`, `agreement-process`, `purpose-process`, `tenant-process`.

A **manager** (`in-app-notification-manager`, `m2m-event-manager`) has the same router, service, and error shape but writes to its own database. It has no event store, no `model/domain/` folder, and no `toEvent.ts`.

## Files, In Data Order

- `packages/api-clients/open-api/<domain>Api.yml`: the HTTP API. See `references/change-surfaces.md`.
- `src/config/config.ts`: env parsed with Zod.
- `src/index.ts`: wires databases, readmodel, file manager, service, and server.
- `src/app.ts`: middleware stack. See `references/snippets.md`.
- `src/routers/<Domain>Router.ts`: one handler per operation. Roles are checked here.
- `src/services/<domain>Service.ts`: business logic, one `async` method per command.
- `src/services/validators.ts`: `assert*` rule checks. `agreement-process` uses `src/model/domain/agreement-validators.ts`.
- `src/services/readModelServiceSQL.ts`: loads current state.
- `src/model/domain/toEvent.ts`: `toCreateEvent*` builders.
- `src/model/domain/apiConverter.ts`: domain to API and back.
- `src/model/domain/errors.ts`: `errorCodes`, `makeApiProblem`, factories.
- `src/utilities/errorMappers.ts`: per-operation error to status.
- `src/model/domain/models.ts`: types local to this service.

Not every package has every file: check the sibling.

## Service Method

```ts
async update<Entity>Field(
  entityId: <Entity>Id,
  newValue: <ValueType>,
  { authData, correlationId, logger }: WithLogger<AppContext<UIAuthData | M2MAdminAuthData>>
): Promise<WithMetadata<<Entity>>> {
  logger.info(`Updating <Entity> ${entityId}`);
  const current = await retrieve<Entity>(entityId, readModelService);

  await assertRequesterIsOwner(current.data, authData, readModelService);
  assert<BusinessRule>(current.data);

  const updated: <Entity> = { ...current.data, field: newValue };

  const event = await repository.createEvent(
    toCreateEvent<Entity>FieldUpdated(current.metadata.version, updated, correlationId)
  );
  return { data: updated, metadata: { version: event.newVersion } };
}
```

The builder wires the repository once:

```ts
export function <domain>ServiceBuilder(dbInstance: DB, readModelService: ReadModelServiceSQL, fileManager: FileManager) {
  const repository = eventRepository(dbInstance, <domain>EventToBinaryData);
  return { /* the async command methods */ };
}
```

Permissions are checked twice: the role in the router with `validateAuthorization`, the ownership in the service with `assertRequesterIs*`.

Already-applied commands: see "Code Rules" in `SKILL.md`. Example: `addClientPurpose` throws `purposeAlreadyLinkedToClient`, mapped to 409.

Steps for a new endpoint are in `references/change-surfaces.md`.

## Mistakes To Avoid

- Reading a value from the readmodel right after saving it.
- Passing raw strings as IDs. Brand them with `unsafeBrandId` in the router.
- Filling a new field in one converter and forgetting another.
- Leaving a flag off in a test, so it passes for the wrong reason.

`attribute-registry-process` is the exception to V2: it emits only `event_version: 1`.
