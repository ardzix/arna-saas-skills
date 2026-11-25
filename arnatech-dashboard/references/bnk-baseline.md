# BNK dashboard baseline

The user accepted the current BNK dashboard as the Arnatech dashboard standard on 26 November 2026. This reference pins the implementation rather than assuming a moving branch always has the same design.

- Application: `ardzix/arna_site_fe` (private repository; requires access).
- Source commit: [`96268280ca7792962b0bb931a961bc77b0c32663`](https://github.com/ardzix/arna_site_fe/commit/96268280ca7792962b0bb931a961bc77b0c32663).
- Home route: `/dashboard` on `www.bisnisnaikkelas.com`.
- Reference stack: Next.js 16, React 19, Tailwind CSS 4, and Base UI. Reuse the stack of the target product instead of installing these dependencies solely to follow the style.

## Tokens and geometry

The theme tokens are scoped to `.bnk-dashboard`. They must not retheme public tenant templates.

| Token | Light | Dark |
| --- | --- | --- |
| `--bnk-canvas` | `#f5f6f8` | `#10151e` |
| `--bnk-panel` | `#ffffff` | `#19212d` |
| `--bnk-sidebar` | `#fafbfc` | `#141d29` |
| `--bnk-line` | `#e3e7ee` | `#364354` |
| `--bnk-text` | `#1b293b` | `#edf2f9` |
| `--bnk-muted` | `#59697b` | `#b6c4d5` |
| `--bnk-soft` | `#edf2fc` | `#263955` |

The primary brand color is `#165DFF`. Selected small dark-theme links use `#A6C1FF`; this is not a blanket replacement for button backgrounds with white text. Check contrast for each foreground/surface combination.

| Element | Reference default |
| --- | --- |
| Desktop sidebar / topbar | 240 px / 72 px |
| Content container | 100% width, maximum 1440 px, `min-width: 0` on constrained children |
| Page padding | 20 px; 28 px from 768 px viewport width |
| Page section gap | 24 px |
| Panel | 1 px border, 12 px radius, panel background, no decorative shadow |
| Page title | 24 px, semibold; title and actions can wrap |
| Mobile navigation drawer | Maximum 280 px, constrained to viewport minus 32 px |

These are starting values for a consistent shell. Preserve equivalent established product tokens where possible; adapt data density and specialized workspaces to their actual task.

## Source map

Paths below are relative to the pinned application repository, not this skills repository. Start from the [pinned source tree](https://github.com/ardzix/arna_site_fe/tree/96268280ca7792962b0bb931a961bc77b0c32663).

| Concern | Files |
| --- | --- |
| Tokens and shared page/panel classes | `src/app/globals.css` |
| Shell, sidebar, scope switcher, topbar | `src/components/dashboard/dashboard-shell.tsx`, `app-sidebar.tsx`, `workspace-switcher.tsx`, `topbar.tsx` in the same directory |
| Page heading and wrapping actions | `src/components/dashboard/page-header.tsx` |
| Home work area and independent LevelUp panel | `src/components/dashboard/home/workspace-dashboard.tsx`, `levelup-card.tsx` in the same directory |
| Shared LevelUp display and data | `src/components/dashboard/levelup/stage-activity.tsx`, `overview-screen.tsx`, `levelup-provider.tsx` in the same directory |
| LevelUp access and localized copy | `src/hooks/use-levelup-access.ts`, `src/lib/levelup/types.ts`, `src/lib/levelup/copy.ts` |
| Locale dictionaries and provider | `src/lib/dashboard-locale.ts`, `src/providers/dashboard-locale-provider.tsx` |
| Accessible drawer primitive | `src/components/ui/sheet.tsx` |
| Stable route loading boundary | `src/app/(dashboard)/dashboard/loading.tsx` |
| Existing behavior checks | `scripts/test-dashboard-experience.mjs` |

## Product examples and their limits

**BNK Home.** A compact page heading precedes LevelUp, actual CRM indicators, the main work queue, and supporting opportunity/website panels. LevelUp is embedded on `/dashboard` while its detailed workspace remains at `/business-hub/overview`. The Home panel mounts outside the CRM provider, so a CRM error cannot hide it. Do not require CRM, LevelUp, or these exact routes in another product.

**LevelUp progress.** The 2D/3D activity view represents recorded completed tasks per stage. Its 3D effect uses CSS transforms on stage bars, not a Three.js scene or a time series. Zero remains zero. It does not claim revenue history or automatic classification of business size. The readiness threshold of 80% and reader/writer actions belong to BNK's existing contract; obtain the corresponding rule from the target product rather than copying that percentage.

**Operational and accounting screens.** Compact local navigation and tables support daily work. Accounting keeps selected period/account when a read fails and distinguishes an unknown balance from zero. These UI conventions do not establish that a backend calculation or integration has been fixed.

**Legacy patterns.** This source pin is a UI reference, not certification of every data contract. Some legacy cache keys, including `["accounting", ...]` and `["GET_CURRENT_TENANT_DETAIL"]`, lack explicit subject/org/tenant scoping. Do not copy those keys as integration examples; follow the verified-context and cache-isolation requirements in the skill.

**Hydration.** The implementation uses a static localized route placeholder. Home isolates context-dependent content behind a stable hydration snapshot (`useSyncExternalStore` with a false server snapshot and true client snapshot). Use that mechanism only when the target rendering/data model needs it; do not turn all pages into client-only screens.

**Evidence.** The baseline release had targeted automated and synthetic-browser checks. Those fixtures are not production customer data, authenticated production-browser verification, or SME usability research. New changes need evidence appropriate to their own scope; there is no required historical test count or requirement to deploy merely to use this skill.
