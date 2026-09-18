---
name: arnatech-service
description: Build or modify an Arnatech backend service while preserving SSO, shared-pool tenancy, Commerce entitlements, File Manager ownership, and Pulsar contracts. Use for backend API, worker, migration, or service-integration work.
---

# Arnatech Service

Before changing a boundary, read [the platform contract](../arnatech-platform/references/platform-contract.md). Apply it to the service's actual responsibility; do not turn every service into a platform gateway.

For tenant-owned data in the shared pool, persist `organization_id` and `tenant_id`, scope every list/detail/update/delete query by both, and include those fields in relevant uniqueness constraints and tests. Use the authenticated context derived from a verified token or trusted service principal; never accept browser-provided organization, tenant, user, role, plan, or permission headers as authority.

Protected HTTP APIs must verify Arna SSO RS256 JWTs with the configured public key/JWKS, expiry, issuer, audience, token type, and required claims. Do not decode an unverified payload, use a shared HMAC secret, or make `AllowAny` a substitute for middleware. Service-to-service calls use narrowly scoped service tokens with the target service audience.

For a kiosk, POS, photobooth, or other public terminal, keep the visitor UI public but authenticate the underlying device with an SSO-issued device token. Require `token_type=device`, the expected audience and scope, `device_id`, `organization_id`, and `tenant_id`; then verify the active SSO device registration and the device's assignment to the requested resource or event. A device has one active tenant assignment and must be paired again to change it. Device credentials do not authorize direct calls to Commerce or File Manager: the owning backend uses its own narrowly scoped service token. Do not accept a copied personal app token or a perpetual bearer token as device identity.

Use Commerce runtime entitlements for package-controlled behavior. Store File Manager IDs or stable URLs rather than object-store credentials. Use HTTP for immediate commands and queries; publish versioned Pulsar facts for asynchronous fan-out. Do not access another service's database.

For new APIs, use `/api/v1`, a stable OpenAPI contract, request IDs, pagination where applicable, and the standard error envelope. Maintain an adapter or compatibility window for an existing legacy route. Add authorization, cross-tenant, idempotency, migration, and dependency-failure tests in proportion to the change.

Follow the existing deployment target only after checking the actual runtime command, service port, health endpoints, secrets, and rollback behavior. Do not deploy or mutate production merely because the implementation is ready.
