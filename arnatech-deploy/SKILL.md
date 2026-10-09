---
name: arnatech-deploy
description: Prepare, implement, review, or troubleshoot Arnatech Jenkins CI/CD, Docker agents, and Docker Swarm releases. Use for Jenkinsfiles, agent Dockerfile/Compose changes, build credentials, runtime configuration, migrations, release verification, hotfix reconciliation, and rollback. Also covers coordinated frontend/backend releases while preserving Arnatech tenancy and SSO contracts.
---

# Arnatech Deployment

Read the [platform contract](../arnatech-platform/references/platform-contract.md) and the [Jenkins/Swarm SOP](references/jenkins-swarm-sop.md). For missing agent tools, Docker Desktop, or agent image/Compose changes, also read the [agent SOP](references/jenkins-agent-sop.md). Use the [release record](assets/release-record.yaml) to record concrete results and evidence; never mark an unperformed check as passed.

Discover the actual repository, branch, Jenkins job/agent, target manager, service names, networks, ports, credentials references, and deployment source of truth before changing them. Base adaptations on implemented commands and the active runtime, not an older README or an arbitrary sibling service. Keep service-specific names, audiences, credential IDs, replicas, feature flags, and provider settings outside this generic skill.

Work within the user's authorized scope. Creating a pipeline, SOP, or skill does not itself authorize production rollout, credential rotation, tenant grants, or sending a message. Continue an already authorized deployment without asking again. If a required access or release decision is missing, complete independent preparation and report the precise missing input.

Preserve these release gates:

- Test the exact commit before publishing its runtime image. Deploy the approved branch commit by resolved registry digest; a successful push is not a successful deployment.
- Scope Jenkins credentials to their stage; use shell expansion rather than Groovy interpolation for secret values. Keep runtime secrets out of Git, build context, images, archived reports, and browser bundles.
- Validate enabled integrations and tenant-independent backend configuration. Use SSO, Commerce, File Manager, ArnaSite, and Pulsar through their owned contracts; do not provision per-tenant infrastructure variables or bypass entitlement checks to make a probe pass.
- Run migrations as a bounded, single one-off task with the same digest, credentials, and network as the release. A failed migration stops rollout. Do not create database roles, change grants, or rewrite credentials to compensate for a mistaken configuration.
- Roll existing services using compatible schema and API/event contracts. Verify every changed service's digest, task convergence, health/readiness, and relevant public/authenticated path before reporting completion.
- Preserve the previous working image/configuration for rollback. Application rollback does not reverse schema changes. Reconcile emergency runtime changes with the durable Jenkins credential or deployment manifest.

For agent repairs, distinguish editing, building, recreating, and reconnecting. Verify binaries and image identity inside the running agent and the Jenkins node's online state. Docker Desktop Restart alone cannot apply a Dockerfile change.

Report repository/commit, build/release identifiers, actual deployed digest and services, configuration source changes, tests/probes performed, and any remaining manual step. Separate prepared, published, deployed, verified, failed, and rolled-back states. Redact credentials and personal customer data from evidence.
