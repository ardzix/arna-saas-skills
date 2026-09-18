# Arnatech SaaS platform contract

## Capability ownership

| Capability | Owner | Consumer rule |
| --- | --- | --- |
| Identity, organizations, roles, permissions, MFA, passkeys, device registrations | Arna SSO | Verify its signed tokens; do not create a parallel identity or device store. |
| Products, prices, subscriptions, invoices, entitlements | Arna Commerce | Ask for runtime entitlements; do not infer access from a plan label. |
| Files, metadata, access policy, object storage | Arna File Manager | Store file IDs/stable URLs, not raw object-store credentials. |
| Tenant, CMS, domains, website context | ArnaSite | Resolve tenant context from trusted host/resource context. |
| Provider payment webhook ingress | Payment Router | Consume payment facts through Pulsar. |
| Shared asynchronous facts | Pulsar | Use versioned, idempotently consumed events. |

## Shared-pool tenancy

ArnaSite is a shared-pool product. A tenant is a business workspace associated with one SSO organization. Its active implementation uses the `pool_shared` schema; shared-schema correctness comes from application-level scoping.

- Tenant-owned records must persist `organization_id` and `tenant_id` as immutable ownership fields.
- All read and mutation queries must constrain both identifiers. Include ownership in uniqueness constraints and cross-tenant tests.
- Commerce billing and entitlements are organization-level. A tenant may consume an org entitlement, but a service must still check tenant membership and resource ownership.
- Default business files are tenant-scoped. File Manager needs a tenant dimension in addition to `owner_org_id`; use an explicit organization-shared scope only for deliberately shared assets.
- Template ownership must use `source_tenant_id`, not `source_tenant_schema`; every shared tenant currently has schema name `pool_shared`.
- Browser-supplied `X-Arna-*` headers, local storage, and query strings are routing hints only. Verify them against the authenticated principal and resolved tenant.

## Central SSO and app sessions

Users sign in once at SSO. Apps then redirect to SSO using PKCE authorization code flow. With an active central SSO session, authorization is immediate and the user sees no second login form.

1. App creates PKCE state and verifier and redirects to an allowlisted SSO authorization endpoint.
2. SSO authenticates only if its own session is absent, then returns a short-lived one-time code to the exact registered callback URI.
3. The app backend verifies state and exchanges the code server-side.
4. The app creates a local, host-only `HttpOnly`, `Secure`, appropriate-`SameSite` session and rotates/ends it on logout.

This works across `ems.arnatech.id`, `site.arnatech.id`, and `www.bisnisnaikkelas.com`. Do not use a parent-domain JavaScript-readable JWT as the shared session mechanism. The current ArnaSite PKCE bridge is a useful foundation but currently returns token pairs to the browser; treat that as transition code and move the exchange/session write into the application backend/BFF.

All resource services validate RS256 signatures with SSO public keys/JWKS plus expiry, issuer, audience, token type, and identity claims. The Business Hub legacy HMAC/payload-only implementation must not be used in production. Service tokens require narrow scopes and the intended target audience; SSO's legacy hard-coded `storage` service audience must become an allowlisted requested audience.

## Device identities and public terminals

A public visitor screen may have no human login, but its underlying device is authenticated. Arna SSO owns `DeviceRegistration`, including the immutable `organization_id`, `tenant_id`, `device_id`, client/key identifier, scopes, active status, approver, and audit trail. A device has one active organization-and-tenant assignment; changing that assignment requires a new approval by a user authorized for device activation in the target tenant.

Use the OAuth 2.0 Device Authorization Grant for a kiosk, POS, photobooth, scanner, or other device without a suitable browser. The device obtains a short-lived, one-time device code and shows the verification URI as a QR code plus a human-readable code. An operator completes approval in the normal SSO browser session, verifies the displayed device identity and requested tenant/scopes, and explicitly approves or denies it. A manually entered code is an alternative presentation of that same one-time pairing flow; do not copy a personal API token or a durable bearer token into a device.

Issue a short-lived, audience-specific device access token and a rotated, revocable device refresh credential, rather than a permanently long-lived access token. Bind credentials to a device-held key with DPoP or mTLS where the hardware supports it, and retain the key reference in the registration. Device tokens must carry and be checked for `token_type=device`, `device_id`, `organization_id`, `tenant_id`, scope, issuer, expiry, and the intended resource audience. A resource service also verifies the registration is active and that the requested event/resource is assigned to that device. Revocation, replacement, reassignment, and suspicious pairing must be auditable and take effect no later than the short access-token lifetime.

Device access is not backend service access. A photobooth device, for example, is issued only an audience such as `photobooth-api`; its backend uses separately scoped service credentials for Commerce and File Manager. Never expose a device credential to the public visitor browser, and never derive device, organization, or tenant authority from a public QR, header, query parameter, or local storage.

## HTTP contract

Use HTTPS and version new APIs under `/api/v1`. Preserve existing `/api` and unversioned routes through a documented compatibility period.

Use a consistent error shape for new or migrated endpoints:

```json
{
  "error": "machine_readable_code",
  "detail": "Human-readable explanation.",
  "request_id": "trace-or-request-id"
}
```

Include OpenAPI, bounded timeouts, idempotency on harmful retries, and explicit authorization. Every service exposes `/health/live` and `/health/ready`; a liveness probe must not fail merely because a downstream dependency is unavailable.

## Delivery baseline

Use a pinned Docker runtime, immutable image tags, external secrets, health checks, resource limits, a rolling update and rollback policy on the production overlay network. Do not use an image-embedded secret, delete/recreate a healthy service for routine updates, or rely solely on `latest`.

Known implementation gaps to account for in a migration plan: services have mixed Django/Python versions and health-route conventions; File Manager's Jenkins port target does not match its Gunicorn port; Business Hub only builds its image; SSO lacks the other repositories' Jenkins deployment artifact.
