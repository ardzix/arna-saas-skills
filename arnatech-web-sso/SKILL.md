---
name: arnatech-web-sso
description: Build or modify Arnatech web frontends and their SSO integration. Use for Next.js/React app flows involving central sign-in, tenant context, Commerce gating, ArnaSite, or File Manager.
---

# Arnatech Web Sso

Read [the platform contract](../arnatech-platform/references/platform-contract.md) before changing authentication, tenant context, payment, or API clients.

Provide single sign-on through the SSO authorization-code flow with PKCE. On an app with no local session, redirect to SSO; an active SSO session must return immediately without asking for credentials. Exchange the one-time code server-side and establish an app-local, host-only `HttpOnly` session. Do not try to share a browser-readable JWT across apps or registrable domains; `.arnatech.id` cookies cannot cover `bisnisnaikkelas.com`.

For a kiosk or device screen that cannot safely host the browser flow, use SSO's OAuth Device Authorization Grant. The screen displays a short-lived QR/verification code only; the approving operator uses their normal SSO browser session, verifies the device and tenant, and approves the requested scopes. Never render an access token, refresh credential, or reusable personal app token into the QR or public UI. A public visitor screen remains unauthenticated even though the installed device is authenticated; keep its device credential outside browser-accessible storage.

Keep browser state non-authoritative. Resolve tenant identity from the trusted host/resource context and verified SSO organization membership. Never treat cookies, query parameters, or local storage values for organization/tenant/role/plan as proof of access. Send the Bearer token only through the approved app session or BFF path.

Render package UX from Commerce runtime entitlements, but leave final authorization and limits to backend services. Initiate checkout via ArnaSite/Commerce server APIs; never expose payment-provider secrets. Handle payment propagation as a pending state and refresh entitlement state rather than assuming an immediate upgrade.

For public websites, preserve tenant-domain routing and keep public endpoints separate from dashboard routes. For files, use File Manager stable URLs or presigned-upload flows. Match existing frontend behavior only where it conforms to these boundaries; document a compatibility step when replacing legacy token or route handling.
