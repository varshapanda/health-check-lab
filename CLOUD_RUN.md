# Cloud Run Health Check Mapping

## Local Health Checks

The Orders API implements two separate health endpoints:

* `GET /health` is the liveness check. It only verifies that the Node.js application process is running and does not depend on the database.
* `GET /ready` is the readiness check. It verifies that the application can reach PostgreSQL by executing a simple database query. It returns `200 READY` when the database is reachable and `503 NOT READY` when the database cannot be reached.

## Container Health Check

Docker Compose uses the `/health` endpoint as the container health check:

```yaml
healthcheck:
  test: ["CMD", "wget", "--spider", "-q", "http://localhost:3000/health"]
  interval: 10s
  timeout: 3s
  start_period: 20s
  retries: 3
```

The `orders-api` service also uses:

```yaml
restart: unless-stopped
```

This allows the local container runtime to restart the service when appropriate.

## Cloud Run Mapping

| Local implementation         | Cloud Run concept                                            |
| ---------------------------- | ------------------------------------------------------------ |
| `/health`                    | Cloud Run liveness probe                                     |
| `/ready`                     | Cloud Run readiness/startup dependency check                 |
| HTTP 200 from `/health`      | Instance is alive                                            |
| HTTP 503 from `/ready`       | Service is not ready to serve dependency-dependent traffic   |
| Docker Compose `healthcheck` | Container-level health checking                              |
| `restart: unless-stopped`    | Cloud Run manages unhealthy instances and instance lifecycle |

The local `/health` endpoint is intentionally shallow because liveness should determine whether the application process itself is functioning.

The `/ready` endpoint is intentionally deeper because readiness verifies the database dependency before considering the service ready.

## Deployment Note

This document is documentation only. No Cloud Run deployment is performed as part of this assignment.
