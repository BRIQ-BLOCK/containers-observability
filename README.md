# containers-observability

One-command **Docker container metrics** stack: **cAdvisor** → **Prometheus** → **Grafana**, with the Prometheus data source and a Docker/cAdvisor dashboard **provisioned from files** (no manual data source setup).

## Prerequisites

- Docker Engine and Docker Compose v2

## Quick start

```bash
docker compose up -d
```

Then open:

| Service    | URL              |
|-----------|------------------|
| Grafana   | http://localhost:3000  |
| Prometheus| http://localhost:9090  |
| cAdvisor  | http://localhost:8080  |

On a fresh volume Grafana starts with **admin** / **admin** and asks for a new password on first login. Users and settings persist in the `grafana-data` volume.

The dashboard **Cadvisor exporter** (Grafana.com [14282](https://grafana.com/grafana/dashboards/14282-docker-monitoring-with-prometheus/), vendored as `grafana/dashboards/docker-monitoring.json`) should appear under **Dashboards** after startup.

## Customizing ports and credentials

All defaults live in [`docker-compose.yml`](docker-compose.yml); there is no `.env` file.

- **Host ports** — edit the `ports` mapping for each service (left side is the host port). For example, use `"3001:3000"` for Grafana if **3000** is already taken.

After changes, run `docker compose up -d` again so containers are recreated as needed.

## Verify

1. Prometheus: **Status → Targets** — job `docker-containers` targeting cAdvisor should be **UP**.
2. Grafana: **Connections → Data sources** — **Prometheus** should exist (UID `prometheus`).
3. Grafana: open the provisioned Docker/cAdvisor dashboard and confirm panels load.

### Avoiding port conflicts

This stack binds **3000** (Grafana), **9090** (Prometheus), and **8080** (cAdvisor) on the host by default. If any of those are already in use (for example by a web app, another metrics stack, or a local proxy), change the **left** side of each `ports` entry in [`docker-compose.yml`](docker-compose.yml)—for example `"3001:3000"` maps host port 3001 to Grafana’s container port 3000. Use `docker compose ps` or your OS tools to see what is listening before you start.

## macOS / Docker Desktop

cAdvisor runs in the Linux VM. The bundled volume mounts work for many setups; if targets stay **DOWN** or container lists are empty, check Docker Desktop resources and [cAdvisor](https://github.com/google/cadvisor/blob/master/docs/running.md) run requirements.

### Docker Engine 29+ (API 1.44)

If cAdvisor logs show *client version 1.41 is too old … minimum … 1.44*, your Docker daemon is newer than the old `gcr.io/cadvisor/cadvisor` images support. This stack uses **`ghcr.io/google/cadvisor:0.56.2`**, which includes a compatible client. After changing the image, run `docker compose pull && docker compose up -d`.

## Security

Do not publish Grafana, Prometheus, or cAdvisor on untrusted networks without TLS, strong passwords, and network controls. Set a strong Grafana password in `docker-compose.yml` before doing so.

## Scope

This repo covers **metrics** only (availability-ish signals, CPU/memory, etc.). Logs (e.g. Loki) and traces (e.g. Tempo) are not included.

## License

Released into the public domain under [The Unlicense](https://unlicense.org/). See [LICENSE](LICENSE).

## Remote hosts (briq-large)

briq-large runs `node_exporter` (127.0.0.1:9100) and `cadvisor` (127.0.0.1:8089) as systemd services with `MemoryMax`/`CPUQuota` caps. Its nginx serves them only to briq's private IP, behind basic auth, under the internal name `metrics.briq-large.internal` (`/node/metrics`, `/cadvisor/metrics`). Prometheus resolves that name through `extra_hosts` and reads the password from `prometheus/secrets/briq-large.pass`, which is gitignored and lives only on the server.

All published ports bind to 127.0.0.1. Every service has a memory cap so the stack cannot starve application containers.
