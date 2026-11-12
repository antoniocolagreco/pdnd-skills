# Security, Auth, And Visibility

## Layers

- **Authentication** proves who is calling. Use the existing middleware. Never parse JWTs, client assertions, DPoP proofs, or interop headers in feature code.
- **Role authorization** decides which actor classes may call an endpoint. Routers call `validateAuthorization` with explicit role arrays.
- **Ownership and visibility** decide if this caller may act on this resource. Services and validators check the tenant, producer or consumer side, client, key, purpose, user, and delegation. A valid role is never enough.
- **Proof binding** connects request, client, key, token, and HTTP operation for M2M and DPoP. Never bypass it for a new route.

`validateAuthorization` only checks the role and narrows `ctx.authData`. It does not prove ownership.

## Roles

`userRole` and `systemRole` are in `packages/commons/src/auth/roles.ts`. `authRole`, in `packages/commons/src/auth/authorization.ts`, joins them.

Never infer what a role may do: read the role array of the endpoint.

User roles, carried by `UIAuthData`:

- `ADMIN_ROLE` (`admin`): tenant administrator.
- `SECURITY_ROLE` (`security`): security operator on clients, keys, and keychains. Not equal to admin.
- `API_ROLE` (`api`): API operator.
- `SUPPORT_ROLE` (`support`): support and backoffice. Not equal to admin.
- `REVIEWER_ROLE` (`reviewer`): used by the purpose risk analysis review workflow.
- `VIEWER_ROLE` (`viewer`): admitted on selected endpoints.

System roles, carried by tokens with `systemRole`:

- `M2M_ROLE` (`m2m`): client token with `organizationId` and `clientId`, no `userId`. Fails on endpoints that admit only `M2M_ADMIN_ROLE`.
- `M2M_ADMIN_ROLE` (`m2m-admin`): client token bound to an admin user, with `organizationId`, `clientId`, and `userId`. For M2M writes.
- `INTERNAL_ROLE` (`internal`): service-to-service, no tenant or user identity. Only for internal routes.
- `MAINTENANCE_ROLE` (`maintenance`): no tenant or user identity. Only for maintenance endpoints and jobs.

## Where To Look

- Middleware, roles, JWT, context: `packages/commons/src/auth`, `packages/commons/src/context`.
- Clients, keys, keychains: `packages/authorization-process`.
- Token issuing: `packages/authorization-server`, the `*-platformstate-writer` packages.
- Assertions and DPoP: `packages/client-assertion-validation`, `packages/dpop-validation`.
- Public requests: `packages/api-gateway`, `packages/m2m-gateway`, `packages/m2m-gateway-v3`.
- Visibility: `packages/m2m-event-dispatcher`, outbound writers, notification dispatchers.

## Every New Or Changed Endpoint

1. Keep authentication in the app stack.
2. Use the narrowest role array in the router.
3. Check resource-level access in the service or validator.
4. Forward interop and auth headers through BFF and gateway calls.
5. Map failures with the status codes in `references/code-conventions.md`. Use 404 to hide a resource only if the package already does.

## M2M, DPoP, Tokens

- Keep client identity, key ownership, `kid` lookup, audience, and method and URL binding aligned with the existing validation.
- Never weaken replay protection, proof freshness, or token binding to pass a test.
- When client, key, purpose, keychain, or tenant data changes, check the platform state writers and token-generation readmodels.
- For public events, check producer, consumer, delegate, delegator, admin, and unrelated tenant.

## Tests

- API: allowed and forbidden roles, validation, status, Problem payload.
- Integration: tenant ownership, producer or consumer side, delegation, membership.
- Gateway: caller identity, DPoP when touched, rate limiting, error shape, visibility.
- A security regression test names the attacker, the target, and the exact status or error.
