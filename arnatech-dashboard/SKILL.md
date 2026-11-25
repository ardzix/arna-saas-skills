---
name: arnatech-dashboard
description: Design, implement, or review authenticated Arnatech dashboards and business workspaces using the approved BNK dashboard UI/UX standard. Use for dashboard layouts, navigation, shared components, responsive tables, themes, localization, and data states. Public marketing sites retain their own design unless explicitly included.
---

# Arnatech Dashboard

Use the BNK dashboard accepted on 26 November 2026 as the visual and interaction baseline. Carry its reusable patterns into Arnatech products; choose content and workflows for the product being changed. User requirements take precedence over these conventions.

Read [the BNK baseline](references/bnk-baseline.md) for exact tokens, dimensions, component locations, and implementation examples. Reuse the target repository's stack and components; the BNK implementation does not require a framework migration.

For session, BFF, or tenant integration, also use [arnatech-web-sso](../arnatech-web-sso/SKILL.md). For changes to service ownership or authorization contracts, read [the platform contract](../arnatech-platform/references/platform-contract.md). This skill defines UI conventions; it does not grant deployment, access-management, or other external-action authority.

## Establish the page contract

Before changing the page, identify its main user task, active organization/tenant, applicable roles and entitlements, existing routes, and authoritative data sources. Inspect the current shared shell and components before adding alternatives.

Keep existing business behavior and deep links working. Define which part of the screen answers “what needs my attention?” and the next action it supports. A working queue should dominate an operational screen; a reporting screen may give charts the main space. Avoid an oversized welcome or tutorial banner that pushes useful work below the fold.

## Shell and visual hierarchy

- Use a neutral canvas, bordered panels, restrained accents, and the existing light/dark theme. Scope dashboard tokens to the dashboard shell so tenant websites keep their own styling.
- Use one consistent page header: title, concise context or description, and one primary action. Let secondary actions wrap. Keep organization/tenant scope visible in the shared shell.
- Follow the reference spacing, panel radius, and width as defaults. Adjust density to the task and target viewport rather than creating a new visual system per module.
- Group sidebar items by user intent. Make the active destination obvious; separate destination links from submenu toggles. Keep settings predictable and only show destinations relevant to the user's verified access. Hiding navigation is not authorization.
- Use local module navigation when the workflow needs it; avoid repeating the same expanded navigation at multiple levels. On mobile, use an accessible drawer with focus containment, Escape, focus restoration, and no hidden tabbable controls.
- Reuse shared headers, panels, controls, tables, and feature components. A feature shown on both Home and its module should share behavior and data contracts, with a compact Home variant when useful.

## Data and interactions

- Show real measures with understandable labels, units, scope, and periods. Do not invent trends, activity, or progress. Unknown values are not zero. Distinguish source data from any explicitly requested demonstration data.
- Provide loading, empty, failed, and denied states appropriate to each panel, with an actionable recovery path. Let independently available panels remain usable when another service fails.
- Resolve verified subject, organization, tenant, roles, and entitlements before protected reads. Scope query caches to that context and clear stale views when it changes. Final access checks remain on the server.
- Keep search, filters, pagination, and selection when opening or closing a record. Use a contextual drawer for focused record work and a full page for complex workflows; retain canonical URLs and identifiers.
- Keep retry behavior consistent with the API's idempotency contract. A retry button for a failed read must not repeat a business mutation.
- Tables should prioritize human names and readable status/amount comparisons. Wrap descriptive text, keep full identifiers accessible when needed, and contain horizontal scrolling inside the table region.
- If a requested feature belongs on Home, render its useful state and actions there. A sidebar entry or shortcut alone does not fulfill that requirement. In BNK this applies to LevelUp on `/dashboard`; other products choose their own Home features.

## Accessibility, locale, and rendering

- Use semantic controls, visible keyboard focus, accessible labels, and sufficient contrast against the actual surface in both themes. Do not communicate status through color alone.
- Keep one active UI language. Add strings through the product's dictionaries and format dates, numbers, currency, and plural forms with its locale utilities. Preserve proper names and identifiers.
- Support narrow screens without page-wide horizontal overflow. Allow titles and actions to wrap; keep essential actions and record identity available on mobile.
- Keep server output and the first client render consistent. Route loading boundaries should render stable placeholders rather than mounting a second live dashboard. When a shared cache or browser-only context causes a mismatch, isolate the affected section until hydration; do not hide the entire application or suppress hydration errors.
- Treat 3D and motion as optional enhancements. Retain equivalent information and actions in an accessible 2D/text view, honor reduced motion, and avoid loading heavy effects on the critical path. A decorative scene must not imply unrecorded business growth.

## Verify the actual change

Use the repository's relevant checks and targeted tests. Review affected layouts at desktop and narrow widths (the BNK reference includes 320–390 px), both themes and supported locales where changed. Exercise keyboard navigation, wrapping, and table overflow. Allow layout to settle after viewport changes before measuring it.

For data or access changes, exercise applicable loading, empty, partial-error, denied, reader/writer, and tenant-switch states. Check for hydration and runtime errors. Test shared feature variants where they appear, including Home when affected.

Report the changed behavior and evidence precisely: source inspection, local checks, synthetic browser fixtures, production HTTP probes, authenticated browser checks, and user acceptance are different kinds of evidence. The accepted visual baseline does not establish usability for every SME or prove full accessibility conformance. Record remaining limitations without claiming checks that were not run.
