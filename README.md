<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="assets/icon-dark.svg">
    <img src="assets/icon.svg" alt="NetBird logo" width="128">
  </picture>
</p>

<h1 align="center">aac-docker-to-netbird</h1>

<p align="center">Self-hosted deployment definition for docker-to-netbird, a reconciler that publishes labelled Docker containers as NetBird reverse-proxy services.</p>

<p align="center">
  <a href="https://github.com/netbirdio/netbird"><img alt="Upstream" src="https://img.shields.io/badge/upstream-netbirdio%2Fnetbird-blue?logo=github"></a>
  <img alt="Image" src="https://img.shields.io/badge/image-docker--to--netbird%3A0.1.1-2496ED?logo=docker&logoColor=white">
</p>

## About

docker-to-netbird watches the Docker host for containers that carry `netbird.expose.*` labels
and reconciles them into NetBird reverse-proxy services through the NetBird management API.
A container that is labelled for exposure is published, and services for containers that no
longer carry the labels are removed again. The reconciler only makes outbound calls to the
NetBird management API; it exposes no ports and needs no ingress or DNS integration itself.
The NetBird peer name is derived from the Docker host name automatically.

Key features:

- Watches container labels on the local Docker daemon (read-only socket access)
- Reconciles labelled containers into NetBird reverse-proxy services
- Periodic checks and full reconciliation at configurable intervals
- No inbound network surface (no published ports)

## Upstream

- NetBird (target platform): <https://github.com/netbirdio/netbird>
- NetBird documentation: <https://docs.netbird.io>
- Container image: `docker-to-netbird:0.1.1`
- docker-to-netbird is a platform-internal tool; it has no separate public upstream project.

## Quick start (Docker Compose)

The Compose file is rendered from `service.yml` by the App-as-Code template engine. This
definition does not declare any external Docker networks, so none need to be created for it.

```bash
# Provide NETBIRD_API_URL and NETBIRD_API_TOKEN (see "Configuration")
docker compose up -d
```

The management API credentials are secrets and are not stored in this repository. Provide
them through the deployment pipeline or in a `stack.env` file next to the Compose file. Do not
commit them.

## Configuration

Environment variables (defaults from `service.yml`; `CHANGE_ME` must be set per deployment):

| Variable | Default | Description |
|---|---|---|
| `NETBIRD_API_URL` | `CHANGE_ME` (secret) | Base URL of the NetBird management API |
| `NETBIRD_API_TOKEN` | `CHANGE_ME` (secret) | API token with permission to manage reverse-proxy services |
| `NETBIRD_LABEL_PREFIX` | `netbird.expose` | Container label prefix the reconciler watches |
| `CHECK_INTERVAL` | `60` | Interval of the periodic check |
| `RECONCILE_INTERVAL` | `300` | Interval of the full reconciliation |
| `LOG_LEVEL` | `INFO` | Log verbosity |
| `TZ` | `Europe/Berlin` | Time zone |

Ports: none. The service does not listen on any port.

Volumes:

| Mount | Description |
|---|---|
| `/var/run/docker.sock` (host) -> `/var/run/docker.sock` (read-only) | Docker socket, used to watch container labels |

## Deploy with App-as-Code

1. Edit `service.yml`. It is the single source of truth for image, environment, volumes and
   per-stage overrides. Do not edit the rendered output.
2. Pushing to `main` (or a configured stage branch) runs the service pipeline, which renders
   the manifests with `aac-template-engine` and triggers the deployment.
3. Releases of this definition are published under <https://github.com/app-as-code/aac-docker-to-netbird>.

## Credits and license

NetBird is a project of its own developers and is licensed by them; see its repository for the
license terms. This repository only contains the deployment definition for docker-to-netbird
and is not a fork of NetBird. The icon is taken from the selfh.st icon collection; see
[`assets/ICON_SOURCE.md`](assets/ICON_SOURCE.md) for sources and licenses.
