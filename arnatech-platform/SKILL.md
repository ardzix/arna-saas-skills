---
name: arnatech-platform
description: Define, review, or align Arnatech SaaS services and cross-service contracts. Use for architecture, tenancy, SSO, Commerce, storage, API, event, or deployment decisions spanning multiple Arnatech modules.
---

# Arnatech Platform

Use this skill when a request crosses service boundaries or establishes a platform convention. First read [the platform contract](references/platform-contract.md).

Start from implemented behavior, not an older README alone. Identify the owner of the capability, the authoritative identifiers, and every affected caller before proposing a change. Preserve legacy contracts during a documented migration; do not silently change a public route, token claim, event topic, or entitlement key.

Treat these as established platform decisions:

- ArnaSite uses shared-pool tenancy. Business data is scoped by `organization_id` and `tenant_id`; Commerce remains organization-scoped.
- Arna SSO supplies central browser sign-in. Applications use PKCE authorization code flow to obtain an app-local, `HttpOnly` session, so navigation between services does not require another login.
- Arna SSO also owns registered device identities. A public terminal is backed by a tenant-bound device credential obtained through an approved device-pairing flow; it is not an anonymous service or a long-lived personal app token.
- SSO, Commerce, File Manager, ArnaSite, Payment Router, and Pulsar own their respective platform capabilities. Do not duplicate them in a consumer service.

When a proposed change alters a product policy—such as entitlement scope, tenant ownership, data retention, or migration cutover—state the concrete alternatives and a recommendation, then ask the user to choose. Security validation and tenant isolation are not optional trade-offs.

For payment work, also read [the payment-event contract](../arnatech-payment-events/references/event-contract.md). For a single backend or frontend implementation, use `arnatech-service` or `arnatech-web-sso` alongside this skill.
