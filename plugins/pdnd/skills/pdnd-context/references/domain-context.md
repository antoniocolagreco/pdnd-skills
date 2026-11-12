# Domain Context

PDND Interoperability lets public and private entities publish and consume APIs through e-services, purposes, agreements, attributes, and OAuth/M2M access.

## E-Service And Descriptor

An e-service is the API published by a producer (erogatore). A descriptor is its version: interface, documents, voucher lifespan, required attributes, approval policy, thresholds, and state.

Descriptor states are in `descriptorState` in `packages/models/src/eservice/eservice.ts`.

## Attributes

- **Certified:** issued by certifying authorities or imported from registries.
- **Verified:** granted by producers.
- **Declared:** self-declared by tenants.

Requirements can be grouped. A change here touches catalog, agreement, tenant, attribute registry, readmodel, BFF, and M2M.

## Purpose

The legal and technical reason for consuming an e-service. It connects a consumer, an e-service, estimated load, risk analysis, and clients. Changes touch `purpose-process`, `agreement-process`, `backend-for-frontend`, and gateways.

## Agreement

The relationship between consumer and producer for a descriptor. It depends on descriptor state, attributes, delegation, approval policy, and purpose. Use domain validators and readmodel state, never raw IDs or UI state.

## Delegation

Changes who acts for a producer or consumer, and who sees events. A change here touches the delegation readmodel, process authorization, and M2M event dispatching.

## OAuth2 And M2M

M2M authentication uses client credentials with JWT client assertions, verified by public key lookup. Access tokens (vouchers) carry claims read by gateways and services. See `references/security.md`.

## Deliver And Receive

In deliver mode, the producer exposes a service and the consumer creates purposes and agreements. Receive mode reverses part of this. Read the domain model before assuming deliver-mode ownership.
