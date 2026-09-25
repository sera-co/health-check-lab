# Cloud Run Mapping for the Health Check Lab

This project uses the same health-check model that managed runtimes like Cloud Run and Kubernetes rely on.

## Local Docker behavior

- `/health` is the liveness check.
  - It is shallow and only confirms the process is alive.
  - It should never depend on a shared dependency such as PostgreSQL.
- `/ready` is the readiness check.
  - It is deep and verifies the application can actually use its dependency.
  - In this lab, it runs a database ping before returning `200 OK`.

## Mapping to Cloud Run

Cloud Run exposes similar concepts through startup and liveness probes. The pattern is the same:

- Startup probe: allow a container time to boot before being considered failed.
- Liveness probe: detect a stuck process and restart it.
- Readiness / dependency checks: confirm required services are reachable before sending traffic.

## Why the liveness check must stay shallow

If `/health` were to query PostgreSQL, a short database blip would trigger a restart of every healthy app instance. That would turn a temporary dependency issue into an avoidable outage.

Instead:

- `/health` checks the app itself.
- `/ready` checks the database dependency.

This keeps traffic off the app during a dependency outage while avoiding unnecessary restarts.

## Equivalent design

- Liveness: `GET /health` → `200 OK` when the process is alive.
- Readiness: `GET /ready` → `200 OK` when the DB is reachable, otherwise `503`.
- Runtime: Docker Compose healthcheck + `restart: unless-stopped` provides automatic self-healing behavior.
