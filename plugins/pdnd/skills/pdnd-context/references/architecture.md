# Architecture

A monorepo of backend services and shared libraries under `packages/*`, in TypeScript on Node.js. It uses pnpm (package manager) and Turbo (runs tasks across packages).

## Tools

- **Express** and **Zodios**: HTTP servers and typed routes.
- **Zod**: data shapes and runtime validation.
- **ts-pattern**: `match(...)` handles each case of a value explicitly.
- **Drizzle**: typed SQL queries on PostgreSQL.
- **Kafka** with **KafkaJS**: event stream between services.
- **Debezium**: copies new database rows into Kafka.
- **PostgreSQL**, **DynamoDB**, **S3**, **SQS**, **Redis**: storage and queues.
- **Vitest** and **Testcontainers**: tests, with real databases in containers.

## Terms

- **Event sourcing:** a change is saved as a record of what happened (an **event**, such as `EServiceAdded`) in the **event store**, a PostgreSQL database. Current state is the result of replaying events.
- **CQRS:** writing and reading use different code and storage.
- **Command side:** the `*-process` services. They check rules and append events.
- **Query side:** the **readmodel**, PostgreSQL tables with current state.
- **Projection:** code that listens to events and writes the readmodel.
- **Eventual consistency:** the readmodel updates a moment after the event is saved, so a process service cannot read back its own write.

## How A Change Flows

```text
HTTP request to change data
   -> *-process service            validate, apply rules, append an event
      -> Debezium -> Kafka         copy the event into the stream
         -> *-readmodel-writer-sql -> SQL readmodel, read by BFF and gateways
         -> *-platformstate-writer -> DynamoDB, used to issue access tokens
         -> *-outbound-writer      -> public events for outside systems
         -> *-dispatcher           -> notifications, emails, M2M rows
```

## Source Files And Generated Files

Edit these by hand:

- `packages/api-clients/open-api/*`: HTTP APIs (OpenAPI).
- `packages/models/proto/v1` and `packages/models/proto/v2`: stored event shapes (Protobuf).
- `docker/readmodel-db/*`: readmodel tables (SQL).

Everything derived from them is listed under "Generated files" in `SKILL.md`, and the regeneration steps are in `references/change-surfaces.md`.

## Package Kinds

The suffix of a package name does not always tell you how it runs. `references/package-index.md` lists the real kind of each package:

- **http:** a server, such as a `*-process` service, the BFF, a gateway, or a `*-manager`.
- **consumer:** runs until stopped and reacts to Kafka or SQS messages. This includes most `-updater`, `-archiver`, and `-persister` packages, for example `client-purpose-updater`, `delegation-items-archiver`, and `token-details-persister`.
- **job:** runs once and exits, such as `-cleaner` and `-checker` packages, importers, `dtd-catalog-exporter`, and `email-digest-dispatcher`.
- **library:** no process, imported by other packages.
