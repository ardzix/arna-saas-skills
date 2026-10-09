# Arnatech containerized Jenkins agent SOP

Use this procedure for Docker-hosted agents, including Docker Desktop on Windows. Match the selected Jenkins node to the physical host, Compose project/service, and running container before editing anything. Remote desktop display names are not reliable Jenkins node identifiers.

## Inventory and capability checks

Record Jenkins node name/label, host, Compose directory/files/project/service, Docker context, current container/image ID, agent base image and controller-compatible Java version. Locate the actual Dockerfile and build context; do not assume two hosts use the same files. Read relevant files without printing agent secrets or dumping `docker inspect` environment values.

Required capabilities depend on the job:

| Capability | Where to verify |
| --- | --- |
| Java, agent connection and writable work directory | Running agent container and Jenkins node |
| Git, CA certificates, Bash/core tools used by scripts | Running agent container |
| Docker CLI plus daemon access | Container tools and selected Docker context/socket |
| Buildx/Compose versions and supported drivers/subcommands | Container plugins; Build Cloud only where selected |
| OpenSSH client (`ssh` **and** `scp`) | Running container, not the Windows host |
| SSH key binding and pinned host keys | Job credentials scope and mounted/provisioned `known_hosts` |
| Language/toolchain versions | Repository requirements; use test containers when possible |

Read-only Linux-container checks, using the verified Compose service name:

```bash
docker compose exec -T <agent-service> sh -ec '
  java -version
  git --version
  command -v bash
  command -v ssh
  command -v scp
  ssh -V
  docker version
'
```

Add `docker buildx version` / `docker compose version` only for jobs using them. Check `docker stack config --help` on the manager where rendering occurs. Do not enable Docker daemon access, elevate users or change socket ACLs as an incidental fix; retain the approved agent execution model and restrict it to trusted jobs. Docker socket access is effectively host administrative access.

## Repair the image durably

For `scp: not found`, install the OpenSSH **client** in the agent image, not an SSH server. A Debian/Ubuntu Dockerfile package fragment is:

```dockerfile
USER root
RUN apt-get update \
    && apt-get install -y --no-install-recommends openssh-client ca-certificates \
    && rm -rf /var/lib/apt/lists/* \
    && command -v ssh && command -v scp
# Return to the existing approved runtime user after installing packages.
```

This is a fragment, not a complete agent Dockerfile. Preserve the existing pinned/reviewed base, compatible Java, Python/native build tools when jobs require them, Docker CLI/plugins, working directory, mounted workspace, controller address and distinct node identity. Prefer pinned agent images and test-container language dependencies to an unrelated global runtime.

Pass the agent connection secret at runtime using the established credentials mechanism; do not bake it into a Dockerfile, track it in Compose, or paste it into release evidence. Preserve the existing entrypoint/inbound connection method. Any downloaded agent JAR uses the trusted controller's HTTPS URL, fails startup on download failure, and lives outside a workspace mount that could hide it. Do not install tools only with `docker exec apt-get`: that repair disappears at recreation.

Inspect Compose interpolation/configuration locally with care: `docker compose config` can reveal resolved secrets. Do not archive or print its full output. Check that the service uses `build:` with the corrected Dockerfile or an explicitly rebuilt/published image. A service with only `image:` will not build a local Dockerfile simply because `--build` is supplied.

## Rebuild, recreate, reconnect

Wait for active builds to finish or coordinate the already authorized interruption; rebuilding an agent may disconnect running jobs. Preserve the old image and Compose configuration for recovery. Operate on one identified agent at a time so the controller keeps available capacity.

For a Compose service built locally, run on the **actual host**, in the verified project directory/context:

```powershell
Set-Location -LiteralPath 'C:\<verified-compose-directory>'
docker compose -p <verified-project> -f <verified-compose-file> build <agent-service>
docker compose -p <verified-project> -f <verified-compose-file> up -d --no-deps --force-recreate <agent-service>
```

For Linux hosts, use the equivalent `cd` and the same explicit Compose project/file/service. Alternatively `up -d --build --no-deps --force-recreate <agent-service>` combines the two operations. If Compose uses a registry image only, build/push the reviewed image separately, update its immutable reference, then pull/recreate the service. Do not run `down -v` or delete workspaces to apply an image change.

Docker Desktop can show container status, image identity, logs and an Exec terminal. **Restart** reruns the existing image; it does not rebuild or recreate. Use an available terminal on that host for the Compose operation, or a verified equivalent build/recreate workflow. If available tools cannot reach the host, report the exact unapplied step; never infer that a remote command ran from an edited local file.

After recreation:

1. Verify the container's new image ID and creation time, not only its tag. Record sanitized values without exposing environment variables.
2. Repeat capability checks inside that running container, including both `ssh` and `scp`; verify work directory/daemon access when changed.
3. Confirm the correct Jenkins node reconnects and is online with the expected name/labels. Review redacted logs if disconnected.
4. Run a harmless preflight job on that node or rerun the already authorized failed pipeline. Installing OpenSSH does not itself authorize a new production deployment.
5. Record each host/node separately. Success on one host does not prove another agent uses the same image.

If the new agent fails, recreate it with the previous known-good image/config and verify reconnection. Do not overwrite the node's connection secret or registration to conceal a tooling error.

## Acceptance evidence

Record four separate states: configuration edited; image built; container recreated with that image; Jenkins node verified online/tooling verified. A completed repair includes all four, plus the relevant preflight/pipeline result. If one is missing, state it as pending and identify its owner.
