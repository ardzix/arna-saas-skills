# Arnatech SaaS AI Skills & Platform Reference

This repository serves two purposes:

1. **AI agent skills and reference context** for agents that design, review, or change Arnatech services.
2. **Human documentation** for engineers and product owners making cross-service architecture decisions.

It does not replace service source code, OpenAPI specifications, or production operating procedures. Treat it as the cross-service architecture contract that those artifacts must follow.

## Read as human documentation

For a change that crosses service boundaries, start with the [platform contract](arnatech-platform/references/platform-contract.md). It defines:

- shared-pool tenancy using `organization_id` and `tenant_id`;
- central SSO with PKCE and app-local sessions;
- device identities for kiosks, POS systems, scanners, and photobooths;
- ownership boundaries for Commerce, File Manager, Payment Router, and Pulsar;
- HTTP and deployment conventions.

For QRIS, payment webhooks, invoice reconciliation, or payment-event migration, then read the [payment-event contract](arnatech-payment-events/references/event-contract.md). Payment Router owns provider webhook ingress, Commerce owns invoice/order/entitlement state, and Pulsar carries versioned, idempotently processed payment facts.

For an implementation-ready guide to QR login, manual-code fallback, silent refresh, local kiosk logout, and account-dashboard revocation, read [Kiosk Device Login](KIOSK_DEVICE_LOGIN.md).

### Repository map

| Area | Read to understand | AI skill folder |
| --- | --- | --- |
| Cross-service architecture, tenancy, SSO, devices | [Platform contract](arnatech-platform/references/platform-contract.md) | `arnatech-platform` |
| Backend APIs, workers, and service integration | [Service skill](arnatech-service/SKILL.md) | `arnatech-service` |
| Web frontends, BFFs, SSO, and public routes | [Web SSO skill](arnatech-web-sso/SKILL.md) | `arnatech-web-sso` |
| QRIS/Xendit, Commerce, Payment Router, and Pulsar | [Payment-event contract](arnatech-payment-events/references/event-contract.md) | `arnatech-payment-events` |

### Example: a public photobooth

A guest-facing photobooth screen can remain login-free, but the installed device must be registered and assigned to exactly one organization and tenant. An authorized operator pairs the device through the SSO Device Authorization Grant. The device receives access only to the photobooth API; the photobooth backend, rather than the guest browser or device UI, creates Commerce orders with `organization_id`, `tenant_id`, `device_id`, and event context. See [Device identities and public terminals](arnatech-platform/references/platform-contract.md#device-identities-and-public-terminals).

## Use with an AI platform

Every `arnatech-*` directory is an independent Agent Skill. It contains `SKILL.md` with agent instructions and, when required, a `references/` directory with more detailed contracts. AI platforms that support filesystem Agent Skills should register the four directories individually; do not register the repository root as one oversized skill.

Platforms without native Skills support can still use this repository as authoritative context. Attach or link the documents relevant to the task and give the agent an explicit instruction such as:

```text
Use the Arnatech SaaS Platform Reference as a mandatory contract.
Read the platform contract first and then the service skill before implementing backend work.
Do not change tenancy, SSO, Commerce, File Manager, or Pulsar contracts without
describing the impact and a migration path.
```

For a cross-service task, attach the [platform contract](arnatech-platform/references/platform-contract.md). Add the [payment-event contract](arnatech-payment-events/references/event-contract.md) for payment work, and use the task-specific skill folder from the repository map above.

## Install as AI skills

### Codex

Codex discovers skills with `SKILL.md`. When available, invoke `$skill-installer` and ask it to install skills from this repository. Start a new session, or restart Codex if the skills do not appear.

For a manual global installation, copy each `arnatech-*` directory directly into Codex's user skill directory, `~/.agents/skills`. Do not copy the repository itself as a single skill.

#### Windows PowerShell

```powershell
$checkoutPath = Join-Path $env:TEMP 'arna-saas-skills'
$skillsPath = Join-Path $env:USERPROFILE '.agents\skills'

git clone git@github.com:ardzix/arna-saas-skills.git $checkoutPath
New-Item -ItemType Directory -Force $skillsPath
Copy-Item -Recurse $checkoutPath\arnatech-* $skillsPath
```

#### macOS / Linux

```bash
git clone git@github.com:ardzix/arna-saas-skills.git /tmp/arna-saas-skills
mkdir -p ~/.agents/skills
cp -R /tmp/arna-saas-skills/arnatech-* ~/.agents/skills/
```

Use an explicit invocation for important work:

```text
$arnatech-platform
Design the tenant and SSO contract for a new service.

$arnatech-service
Add a backend API that uses tenant context and Commerce.

$arnatech-web-sso
Implement a dashboard with central SSO.

$arnatech-payment-events
Design a QRIS flow and an idempotent Pulsar consumer.
```

In the ChatGPT desktop app, use the skill picker (`@`) when available. In Codex CLI or the IDE extension, use `$skill-name` or `/skills`. [OpenAI Docs: Build skills](https://learn.chatgpt.com/docs/build-skills)

### Claude Code

Claude Code uses the same `SKILL.md` folder format. Copy the skill directories to `~/.claude/skills/` for personal use in every project, or to `.claude/skills/` at a project root for a versioned team installation.

```bash
git clone git@github.com:ardzix/arna-saas-skills.git /tmp/arna-saas-skills
mkdir -p ~/.claude/skills
cp -R /tmp/arna-saas-skills/arnatech-* ~/.claude/skills/
```

Restart Claude Code after installation. Claude discovers skills automatically; name the skill in the prompt when you need to guarantee its use. [Claude Agent Skills](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)

### claude.ai

For Claude in the browser, upload each skill directory as a separate ZIP file through **Settings → Features → Custom Skills**. Select only the skills that are relevant to the work; do not upload the whole repository as one skill.

Custom Skills in claude.ai are per user. They must be uploaded separately for each Claude surface, including Claude Code and the API. Availability depends on a plan and code-execution configuration that supports Custom Skills. [Claude Agent Skills](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)

### Google Antigravity

Antigravity uses the same `SKILL.md` format. For project-scoped skills, copy the directories to `<project-root>/.agents/skills/`. For global skills across all workspaces, use `~/.gemini/antigravity/skills/`.

```bash
git clone git@github.com:ardzix/arna-saas-skills.git /tmp/arna-saas-skills
mkdir -p .agents/skills
cp -R /tmp/arna-saas-skills/arnatech-* .agents/skills/
```

Antigravity can select a skill from its description, or you can explicitly mention the skill name in a task. [Google Antigravity: Agent Skills](https://antigravity.google/docs/ide/skills)

### Other Agent Skills-compatible platforms

This repository follows the open Agent Skills structure: one capability per directory, a `SKILL.md` file with `name` and `description` frontmatter, and optional supporting resources. For another platform that adopts this format, follow that platform's instructions for its global or project skill directory and copy the four `arnatech-*` directories directly into it.

If a platform does not provide a Skills feature, use the repository as reference documentation instead. Attach the [platform contract](arnatech-platform/references/platform-contract.md) and any task-specific contract to the agent context. Do not assume that a skill is active until the platform displays or confirms that it discovered the skill.

## Change principles

- Do not create a second identity store, invoice system, entitlement engine, file store, or payment webhook receiver inside a consumer service.
- Do not trust `organization_id`, `tenant_id`, roles, or payment state received from a browser, public QR code, header, or provider callback without verifying it with the authoritative owner.
- Preserve live public contracts during a documented migration; payment event versioning and idempotency are mandatory.
- Do not commit secrets, private keys, personal app tokens, or production data to this repository.

When a contract changes, update the relevant reference document and the skill that guides its implementation in the same commit.
