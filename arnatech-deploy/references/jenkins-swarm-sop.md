# Arnatech Jenkins and Docker Swarm release SOP

This is the reusable release procedure for Arnatech backend APIs, workers, and event publishers. Frontends hosted on Vercel retain that deployment target; use the coordination rules below. Do not migrate hosting, initialize a Swarm, or provision shared platform services as a side effect of a release.

## Ownership and accountability

One person may hold multiple responsibilities; record their identity and evidence rather than requiring extra approvals for routine, already authorized work.

| Responsibility | Accountable for | Evidence |
| --- | --- | --- |
| Service maintainer | Runtime commands, config schema, tests, compatible migrations/contracts | Commit, checks, migration and compatibility notes |
| Platform operator | Agent/tooling, manager access, registry, network, secret delivery, recovery | Agent identity, target inventory, credential references, previous release |
| Release operator | Execute the authorized release and stop failed gates | Job/build URL, digest, stage outcomes, timestamped probes |
| Product/config owner | Package policy, tenant settings, entitlement changes | Owner API/CLI change reference and reason; only when these change |

Copy [release-record.yaml](../assets/release-record.yaml) into the appropriate service's operational records or fill its fields in the release report. Link restricted Jenkins logs rather than publishing raw environment dumps. Record actor, UTC timestamps, and the requested scope. A Jenkins green build proves only stages actually executed: `DEPLOY=false` is a tested/published release, not a deployed one.

## 1. Establish the release inputs

Inspect source and read-only runtime state before adapting another service's pipeline.

| Input | Resolve from |
| --- | --- |
| Repository, production branch, full commit | Actual Git remote and job checkout; `main` by default, explicit legacy branch where required |
| Agent label, runtime/tools, Docker context | Jenkins node configuration, running container, service requirements |
| Registry repository and registry credential ID | Job configuration and approved registry |
| Manager SSH host/user, key credential, trusted host keys | Existing infrastructure inventory; no guessed IPs |
| Stack, service roles, internal ports, overlay and proxy | Existing Swarm/service specs and deployed manifests |
| Runtime file/key credential IDs and required config groups | Application settings, runtime validator, Jenkins credential bindings |
| Migration, API, worker, publisher commands | Dockerfile/entrypoint and application code |
| Probe paths, expected replicas, enabled features | Deployment manifest and implemented endpoints |
| Previous digest/configuration, backup and compatibility | Running service specs, previous release and migration review |

For a new service, create these service-owned artifacts where appropriate:

```text
Jenkinsfile                      # orchestration, credential bindings, gates
Dockerfile / .dockerignore       # pinned build/test/runtime; exclude secrets
deploy/ci.sh                    # reproducible checks and isolated dependencies
deploy/build.sh                 # runtime build/push, optional Build Cloud
deploy/remote.sh                # secure staging/transfer to the existing manager
deploy/swarm.sh                 # validate, migrate, roll, verify, recover
deploy/stack.yml                # roles, network, secrets, resources, health
deploy/runtime.env.example      # config names and safe placeholders only
deploy/validate_runtime_env.*    # required groups and semantic validation
deploy/healthcheck.*            # bounded readiness check with nonzero failure
```

These names are a layout convention, not a mandatory rewrite of existing equivalent artifacts. Keep a thin Jenkinsfile and executable service scripts; document the source if a legacy job still uses an inline pipeline. Verify ports and health routes rather than copying a historical pipeline's mismatched values.

## 2. Prepare the agent and pipeline

Apply the [agent SOP](jenkins-agent-sop.md) for agent changes. Check tools where they execute: a Docker CLI on the agent does not make it a Swarm manager. Before SSH deployment, the agent must have `ssh`, `scp`, trusted `known_hosts`, and its scoped Jenkins SSH credential. The manager must support the actual `docker stack` subcommands used.

Pipeline defaults:

- Explicit checkout, timestamps, bounded total/stage timeouts, build retention and no concurrent deploys to the same stack. `disableConcurrentBuilds()` covers one job only; jobs sharing a stack need the existing cross-job lock or equivalent serialization.
- Deployment is opt-in (`DEPLOY=false` for validation/build jobs) or controlled by the existing approved release job. Verify the checked-out full commit equals the selected remote production branch before deployment, including single-branch jobs without `BRANCH_NAME`.
- Build a release ID from Jenkins build number, commit prefix, and a job-name hash, for example `b<build>-<sha12>-<jobhash8>`. This prevents colliding build numbers across jobs. Pin deployment to the registry digest even if the tag is moved later.
- Make event publishing/build capabilities explicit. Enable a Pulsar publisher only when its client dependency and backend configuration are present; this is deployment capability, not tenant policy.
- Extract JUnit after tests even when tests fail. Early setup failures can legitimately produce no XML; report the original failure, not merely a missing-report message. An empty report is not a passing test suite.

Do not relax a gate to make the pipeline green. Fix the cause or state a documented compatibility limitation.

## 3. Validate the source, then build and publish

Run checks against the exact checkout in a reproducible test stage. For Python/Django services, the proven pattern is lint/format, framework checks, migration drift, generated OpenAPI validation/contract diff, unit tests, and isolated PostgreSQL integration tests. Select equivalent meaningful checks for the stack; use synthetic data and release-specific containers/networks, never production credentials or customer data.

Check shell syntax and manifest compatibility. Older Docker CLIs may lack `docker stack config` even though `docker stack deploy` exists. Use a tested compatible schema/Compose validator in CI and render with `docker stack config --compose-file ...` on the deployment manager before migration. Do not replace this with an unvalidated deploy, and do not mistake local Compose acceptance for Swarm runtime compatibility.

Build guidance:

- Pin base runtime version and reviewed digest; pin application dependencies. Separate test dependencies from the non-root runtime image. Exclude `.env`, private keys, `.git`, local credentials and operational exports using `.dockerignore` and a clean build context.
- Run API, worker and publisher as roles of the same tested release image where the service supports it. Set resource limits, graceful shutdown and bounded log retention.
- Authenticate registry access with Jenkins credentials and `docker login --password-stdin`, not command-line passwords. Use a restricted, per-build `DOCKER_CONFIG`, and clean it on both agent and remote target on success/failure.
- Use `withCredentials` plus a single-quoted Groovy shell block and shell variable expansion. Disable shell tracing around secrets; Jenkins masking is not a substitute for safe handling.
- Docker Build Cloud is an optional existing builder. Select supported build platforms from production nodes and dependencies. A local fallback is acceptable only if its published architecture covers the actual deployment target. Record fallback and verify manifest platforms; do not claim a multi-platform build after a single-platform fallback.
- Resolve the pushed repository/tag to its registry digest. For a manifest list, record the index digest and platform variants; compare platform-specific task digests correctly rather than assuming every node reports the same child digest.

## 4. Stage and validate configuration

Transfer reviewed release scripts/manifests and the bound runtime credential using SSH/SCP with `BatchMode=yes`, bounded connection timeout and `StrictHostKeyChecking=yes`. Provision trusted host fingerprints from approved inventory; never disable host-key checks to fix a connectivity error. Validate/quote remote path components before constructing remote shell commands. Create a release-specific directory with restrictive permissions.

The durable source of truth is the Jenkins runtime credential (or the explicitly established secret manager) plus the versioned deployment manifest. Append/edit required keys in the existing complete runtime configuration; never replace it with a small snippet and lose database/SSO values. Keep an audited configuration revision or secret reference, not secret content, in release evidence.

Validate before migration:

- Required fields, placeholder rejection, production debug/host/proxy settings, internal database endpoint, existing DB name/user, and credentials. `postgres` may be the configured database user; this SOP does not mandate a new role. A login error needs credential verification, not automatic role/grant changes.
- SSO public key/JWKS, issuer/audience/type and the consumer's real token contract; never copy another service's audience or private signing key. Public key files still need correct mount path and runtime permissions.
- Complete config groups for enabled integrations: Commerce service authentication, File Manager, provider inference, Pulsar, website context/customer authentication. Validate optional groups only when used, but do not silently ship a required product capability disabled because every field is empty.
- Public website integration resolves host/tenant through ArnaSite on the backend. SSO owns membership/identity; Commerce owns organization-level entitlements; File Manager owns storage; Pulsar owns asynchronous facts. New tenants must not require a new Vercel secret or global environment entry. Browser `NEXT_PUBLIC_*` variables contain only public configuration.
- Existing manager/control access, overlay network, registry access, disk/memory capacity, and reachable service dependencies. Checks against third parties must be bounded and avoid triggering messages, invoices, or AI usage unless explicitly part of the authorized test.

Create immutable, release-specific Swarm secrets for runtime credentials and mounted verification material; set UID/GID/mode for the application's non-root user. Validate the environment-file loader's quoting/interpolation behavior and precedence. The application must load the secret before starting framework settings or executing migrations. Do not print the resolved environment.

Render the stack with the resolved digest and intended secret names. Keep a previous sanitized service specification/configuration reference for every existing role before mutating it; protect any full specs containing literal environment secrets. Retain old secrets needed by current or previous service specs.

## 5. Run the migration gate

Run one temporary Swarm service with the **same release digest, overlay, environment loader and secret mounts** as the intended runtime. Use the service's actual migration command, `restart-condition=none`, an appropriate timeout/resource limit, and no API-specific HTTP health check.

Wait for its task to reach `complete` with exit code `0`. Treat `failed`, `rejected`, unexpected `shutdown`, nonzero exit and timeout as failures. Capture a redacted diagnostic summary and task ID before removing the transient service. A successful `docker service create` or a briefly `running` migration task is not success.

Stop before rollout if migration fails. Do not suppress its exit code, loop indefinitely, or retry a non-idempotent operation without examining state. Coordinate migrations across any jobs that share the same service/database.

Use expand/contract changes that old and new application versions can both handle during rolling updates. For a destructive or incompatible migration, resolve the explicit release/recovery decision and required backup or restore checkpoint before execution. Rolling back an image does not undo DDL or recover deleted data; do not run a reverse migration automatically.

## 6. Roll out and verify

Update the existing stack with registry authentication and resolved digest. Do not remove/recreate a healthy service for a normal update. Nginx Proxy Manager on the same overlay targets the service's DNS name and **internal listening port**, not a guessed host port. Confirm the actual proxy domain/TLS/routing configuration when it changes.

Starting point for stateless roles: parallelism `1`, `start-first`, delay `10s`, monitor `60s`, failure action `rollback`, and graceful stop longer than the application's drain timeout. Set replicas and CPU/memory from capacity and workload; these values are not universal. With insufficient headroom or a singleton/exclusive worker, use an explicit suitable update policy. Two replicas on one node do not create host-level high availability. Workers/publishers must tolerate overlap through idempotency/locking, or use a non-overlapping update.

Verify each changed role, including a deliberately disabled publisher with zero desired replicas:

1. Desired tasks converge to the intended digest/platform. Check running task states, absence of failed/rejected replacements, and stable replica count over multiple bounded polls.
2. Update status completes. `paused`, `rollback_started`, or `rollback_completed` is a failed requested release even if old tasks are healthy.
3. API containers become healthy; probe `/health/live` and `/health/ready` or documented compatible endpoints. Liveness detects a broken process, not an unavailable downstream service. Readiness tests required local dependencies without exposing secrets. Workers need their real role-appropriate signal, not an HTTP probe copied from the API.
4. Probe internally from the production overlay, then the public HTTPS route. Inspect response semantics, not only HTTP 200 or replica counts. Check the relevant SSO/session/tenant/Commerce path using authorized test identities and confirm unauthorized/cross-tenant rejection when that boundary changed.
5. Review redacted startup/error summaries and intended worker/event progress using synthetic read-only or explicitly authorized operations. An idle worker requires a process/role check; state its functional-test limitation honestly.

For UI/session changes, use an actual browser where authorized access is available: an active SSO session should open the service without a second credential prompt, and tenant navigation/public website routes should work. A backend service-token probe cannot prove the browser's cookie/session flow.

## 7. Recover, reconcile and close

On rollout/probe failure, compare the captured previous specs with the attempted release. Request rollback only for existing services actually changed by this release, accounting for an automatic rollback already in progress. Do not roll back unrelated stacks or repeated failures to arbitrary historical versions. First releases have no previous service version: stop/contain the new release and report recovery explicitly.

Wait for rollback convergence and re-run health/public probes before reporting recovery. Verify restoration of the previous digest **and configuration/secret references**. If a previous spec was overwritten by subsequent updates, restore the recorded known-good spec through the established process instead of assuming `docker service rollback` still targets it. Mark the release failed even if recovery succeeds. If rollback fails, record the live state and remaining recovery step; never claim success because the rollback command returned zero.

Remove transient migration/probe services and temporary credential files/registry auth. Keep versioned images, manifests and secrets needed by current/previous specs for the established retention period; prune only after verifying references and recovery retention. Record the cleanup outcome without dumping secrets.

Emergency `docker service update --env-add` is a runtime hotfix. Reconcile it with Jenkins credentials/versioned manifests before the next pipeline rollout, because stack deployment can remove overrides. Record whether durable reconciliation is complete; do not say configuration was saved just because the live container sees it.

Complete the release record with the exact commit, immutable digest, roles updated, gate results, probe timestamp/results, current state, recovery outcome, durable configuration revision and outstanding work.

## Frontend/backend coordination

Keep Vercel deployment separate from backend Jenkins deployment. Record repository and branch/commit for each changed component. If a backend response changes, first deploy a frontend/client version that accepts old and new shapes, verify it, then deploy the new backend; remove compatibility only after the transition. Apply the analogous ordering for SSO claims, callbacks and event consumers. Do not deploy an incompatible backend while the frontend is still awaiting a manual rollout.

Frontend server/BFF secrets stay server-only; static browser bundles may contain only public endpoint configuration. Backend global settings and tenant-owned policies remain backend-managed. Update both apex and `www`/tenant domains when their routing/session behavior is affected. A preview deployment does not prove the production domain runs the same commit.

## Troubleshooting gates

| Symptom | Check before changing anything | Resolution boundary |
| --- | --- | --- |
| `scp: not found`, exit 127 | Tool inside the running agent, image ID, job node | Rebuild/recreate the correct agent; see agent SOP |
| `unknown flag: --compose-file` | Exact command and CLI at that execution location | Supported CI validation and manager rendering; do not skip validation |
| Required config missing | Jenkins credential ID/scope, full file, loader and enabled groups | Correct durable configuration; no secret output |
| Migration exits 1 | Redacted task error, auth/connectivity, actual migration/schema state | Stop rollout, fix cause; no automatic DB-role rewrite |
| Tasks running but old image | Digest, update status, failures, desired task generation | Fix rollout or recover; do not call it deployed |
| Public 502/404 | Proxy overlay/DNS/listening port, route and deployed version | Fix the owning layer, preserve tenant routing |
| Frontend `crm_not_configured` or session 401 | Server-only endpoint config, deployed FE/BE/SSO contracts | Fix config/token validation at the owner; no auth bypass |
| Website works after manual env update only | Runtime override versus bound Jenkins config | Reconcile source of truth before redeploy |

## Reference implementation and limitations

This baseline was extracted on 2026-10-09 from the tested Jenkins design in [`ardzix/arna_crm`](https://github.com/ardzix/arna_crm), commit `aa62850`: `Jenkinsfile`, `Dockerfile`, `deploy/{ci,build,remote,swarm}.sh`, `deploy/jenkins-stack.yml`, and runtime/probe validators. Agent recovery lessons came from the existing containerized Jenkins agents. This source is an implementation example, not an instruction to copy its service names, provider configuration, credentials or deployment addresses.

The SOP strengthens generic application of that design: verify architecture after Build Cloud fallback, serialize separate deploy jobs, verify rollback completion, test required feature configuration/public paths, and reconcile hotfixes. Do not assume every legacy pipeline or even the reference implementation automatically enforces every gate; inspect actual code and document/adapt any gap for the task at hand.
