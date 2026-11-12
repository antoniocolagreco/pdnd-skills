# Package Map

Find which package owns a change. For how a package runs and tests, see `references/package-index.md`.

## By Task

- Change a core domain aggregate: the domain `*-process` package.
- Change an HTTP contract: `packages/api-clients/open-api`, then the owning process, BFF, or gateway.
- UI-facing composition, labels, grouping, deep links: `backend-for-frontend`.
- Public REST API: `api-gateway`.
- External M2M API: `m2m-gateway` and `m2m-gateway-v3`.
- Reading persisted M2M events: `m2m-event-manager`.
- Creating M2M event rows from domain events: `m2m-event-dispatcher`.
- Readmodel: `packages/readmodel`, `packages/readmodel-models`, `docker/readmodel-db`, and the `*-readmodel-writer-sql` of the domain.
- Token or platform state: the `*-platformstate-writer` of the domain.
- Public events for outside systems: the `*-outbound-writer` of the domain.
- External registry ingestion: an `*-importer`.
- Shared domain types and events: `models`. Utilities, config, auth, server, Kafka helpers: `commons`. Test helpers: `commons-test`.
- User-facing error messages for error codes: `packages/error-messages/error-messages.csv`, read by `backend-for-frontend`.
- Local development dashboard: the `dev-dashboard-*` packages.

## By Domain

Catalog and e-services:

- `catalog-process`: e-services, descriptors, documents, risk analysis, template instances.
- `catalog-readmodel-writer-sql`, `catalog-platformstate-writer`, `catalog-outbound-writer`: projections.
- `dtd-catalog-exporter`: public catalog export.
- `eservice-descriptors-archiver`: archives descriptors when agreements are archived.
- `eservice-descriptors-scheduled-archiver`: scheduled descriptor archiving.

Agreements:

- `agreement-process`: agreement lifecycle.
- `agreement-readmodel-writer-sql`, `agreement-platformstate-writer`, `agreement-outbound-writer`: projections.
- `agreement-lifecycle`: shared validation and filters.
- `compute-agreements-consumer`: recomputes agreements from events.

Purposes:

- `purpose-process`: purposes, risk analysis documents, quotas.
- `purpose-readmodel-writer-sql`, `purpose-platformstate-writer`, `purpose-outbound-writer`: projections.
- `client-purpose-updater`: reacts to purpose and client changes.
- `risk-analysis-processing`: risk analysis processing job.

Purpose templates:

- `purpose-template-process`: templates and documents.
- `purpose-template-readmodel-writer-sql`, `purpose-template-outbound-writer`: projections.

E-service templates:

- `eservice-template-process`: templates and versions.
- `eservice-template-readmodel-writer-sql`, `eservice-template-outbound-writer`: projections.
- `eservice-template-instances-updater`: updates instances when a template changes.

Tenants and attributes:

- `tenant-process`: tenants, mails, certified, declared, and verified attributes.
- `attribute-registry-process`: attribute registry.
- `tenant-readmodel-writer-sql`, `tenant-outbound-writer`, `attribute-registry-readmodel-writer-sql`: projections.
- `ipa-certified-attributes-importer`, `anac-certified-attributes-importer`, `ivass-certified-attributes-importer`, `istat-certified-attributes-importer`, `private-certified-attributes-importer`: certified attribute ingestion.
- `tenant-kind-history-consumer`, `tenant-kind-history-db-models`: history of tenant kind changes.

Authorization, clients, keys:

- `authorization-process`: clients, keys, producer keychains.
- `authorization-server`: issues tokens.
- `authorization-platformstate-writer`, `producer-keychain-platformstate-writer`: DynamoDB token state.
- `client-readmodel-writer-sql`, `key-readmodel-writer-sql`, `producer-key-readmodel-writer-sql`, `producer-keychain-readmodel-writer-sql`: projections.
- `producer-key-events-writer`: producer key events.
- `client-assertion-validation`, `dpop-validation`: validation libraries.
- `token-details-persister`: stores token details for audit.

Delegation:

- `delegation-process`: producer and consumer delegations.
- `delegation-readmodel-writer-sql`, `delegation-outbound-writer`: projections.
- `delegation-items-archiver`: archives purposes and agreements when a delegation is revoked.

Notifications:

- `notification-config-process`, `notification-config-readmodel-writer-sql`: notification preferences.
- `notification-commons`: shared recipients, retrievers, scheduled delivery.
- `in-app-notification-manager`: HTTP API for in-app notifications.
- `in-app-notification-dispatcher`, `email-notification-dispatcher`: create notifications from events.
- `in-app-notification-cleaner`, `in-app-notification-db-models`: cleanup and tables.
- `email-sender`, `notification-email-sender`, `certified-email-sender`: email delivery.
- `email-digest-dispatcher`: digest job.
- `notification-tenant-lifecycle-consumer`, `notification-user-lifecycle-consumer`: lifecycle consumers.
- `scheduled-notification-scheduler`: writes reminders into the `scheduled_notification` table.
- `email-scheduled-notification-dispatcher`, `in-app-scheduled-notification-dispatcher`: deliver ready reminders.
- `scheduled-notification-cleaner`, `scheduled-notification-db-models`: cleanup and tables.

APIs and M2M:

- `backend-for-frontend`: API for the web UI.
- `api-gateway`: public REST API.
- `m2m-gateway`, `m2m-gateway-v3`: M2M APIs.
- `m2m-event-dispatcher`, `m2m-event-manager`, `m2m-event-db-models`, `m2m-event-cleaner`: M2M events.

Audit, documents, storage:

- `application-audit`, `application-audit-fallback`: audit middleware and fallback queue.
- `audit-signer`, `events-signer`, `documents-generator`, `documents-signer`, `signed-objects-persister`: signing and documents.

Exports, analytics, checks, external systems:

- `datalake-data-export`, `datalake-interface-exporter`: datalake exports.
- `domains-analytics-writer`: analytics.
- `kpi-domains-readmodel-checker`, `token-generation-readmodel-checker`, `async-token-generation-readmodel-checker`, `check-selfcare-diff`: reconciliation checks.
- `pn-consumers`: one-shot job despite its name.
- `selfcare-onboarding-consumer`, `selfcare-client-users-updater`: Selfcare consumers.
- `one-trust-notices`: OneTrust notices job.
- `notifier-seeder`: notifier consumer.

Local development tools:

- `dev-dashboard-api`, `dev-dashboard-ui`: API and web UI of the local backend development dashboard.
- `dev-dashboard-contracts`: its OpenAPI contract and generated client.
- `dev-dashboard-bootstrap`: reads each package's dotenv files to start services locally.

Shared libraries (see `references/package-types/shared-libraries.md`):

- `models`, `commons`, `commons-test`, `api-clients`, `readmodel`, `readmodel-models`, `kafka-iam-auth`, `eslint-config`, `kpi-models`.
- `error-messages`: generates and validates typed error messages from `error-messages.csv`.
