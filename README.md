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

Default Grafana login is **admin** / **admin** (set in [`docker-compose.yml`](docker-compose.yml) under the `grafana` service). Change `GF_SECURITY_ADMIN_USER` and `GF_SECURITY_ADMIN_PASSWORD` there before exposing the stack beyond localhost.

The dashboard **Cadvisor exporter** (Grafana.com [14282](https://grafana.com/grafana/dashboards/14282-docker-monitoring-with-prometheus/), vendored as `grafana/dashboards/docker-monitoring.json`) should appear under **Dashboards** after startup.

## Customizing ports and credentials

All defaults live in [`docker-compose.yml`](docker-compose.yml); there is no `.env` file.

- **Host ports** — edit the `ports` mapping for each service (left side is the host port). For example, use `"3001:3000"` for Grafana if **3000** is already taken.
- **Grafana admin** — edit `GF_SECURITY_ADMIN_USER` and `GF_SECURITY_ADMIN_PASSWORD` under the `grafana` → `environment` block.

After changes, run `docker compose up -d` again so containers are recreated as needed.

## Verify

1. Prometheus: **Status → Targets** — job `docker-containers` targeting cAdvisor should be **UP**.
2. Grafana: **Connections → Data sources** — **Prometheus** should exist (UID `prometheus`).
3. Grafana: open the provisioned Docker/cAdvisor dashboard and confirm panels load.

### Ports often used elsewhere (example Briq dev host)

If you remap observability ports, avoid colliding with services already on the host, for example:

| Host ports | Typical use |
|------------|----------------|
| 5173, 5174 | Frontends |
| 2130       | alya |
| 8000–8001, 8004–8005 | briqpilot / pilot services |
| 8085       | centauri-core (maps container 8080) |
| 9050, 9051 | alioth-major, stella-polaris |
| 5540       | redisinsight |

Default stack ports **3000 / 9090 / 8080** are outside that set.

## macOS / Docker Desktop

cAdvisor runs in the Linux VM. The bundled volume mounts work for many setups; if targets stay **DOWN** or container lists are empty, check Docker Desktop resources and [cAdvisor](https://github.com/google/cadvisor/blob/master/docs/running.md) run requirements.

## Security

Do not publish Grafana, Prometheus, or cAdvisor on untrusted networks without TLS, strong passwords, and network controls. Set a strong Grafana password in `docker-compose.yml` before doing so.

## Scope

This repo covers **metrics** only (availability-ish signals, CPU/memory, etc.). Logs (e.g. Loki) and traces (e.g. Tempo) are not included.
