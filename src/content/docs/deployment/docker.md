---
title: "Using Docker"
description: "Run osctrl and all its components with Docker Compose: the development stack and the staging stack built from published images."
sidebar:
  order: 2
---

You can use docker to run **osctrl** and all the components are defined in the `docker-compose-dev.yml` that ties all the components together, to serve a functional deployment.

The Docker stack is meant for development and quick evaluation. It starts nginx, `osctrl-tls`, `osctrl-api`, the React operator frontend, PostgreSQL, Redis, `osctrl-cli`, and a few osquery containers already enrolled into the dev environment.

## Start the Stack

From the **osctrl** repository:

```bash
cp .env.example .env
make docker_dev_certs
make docker_dev
```

The first command creates the environment file used by Docker Compose. The second command generates a local self-signed certificate under `deploy/docker/conf/tls/`. The last command builds the images and starts the full `docker-compose-dev.yml` stack.

Once it is running, open:

```text
https://localhost:8444
```

Your browser will warn about the self-signed certificate. That is expected for the dev stack.

## What Runs

The compose file keeps the services on an internal Docker network and exposes only the ports you need on localhost:

| Service | Purpose |
| --- | --- |
| `osctrl-nginx` | Public entry point for the frontend and API |
| `osctrl-tls` | osquery TLS endpoint |
| `osctrl-api` | API used by the operator frontend |
| `osctrl-frontend` | React operator UI |
| `osctrl-postgres` | PostgreSQL database |
| `osctrl-redis` | Redis cache |
| `osctrl-cli` | Creates the initial environment and user data |
| `osquery-1`, `osquery-2`, `osquery-3` | Test osquery clients |

The stack does not include [osctrl-mcp](/components/osctrl-mcp/), and does not need to: it is launched by your MCP client over stdio rather than run as a container. Point it at the stack's `osctrl-api` endpoint with a token from `osctrl-cli`.

## Useful Commands

```bash
make docker_dev_down
```

Stops the stack.

```bash
make docker_dev_logs_api
make docker_dev_logs_tls
make docker_dev_logs_frontend
make docker_dev_logs_nginx
```

Follow logs for the main services.

```bash
make docker_dev_clean
```

Removes osctrl Docker images and volumes. Use it when you want a fresh database and clean containers.

For production deployments, prefer the native Ubuntu path with `provision.sh` or adapt the compose file to your own orchestration, secrets, certificates, and persistence model.

## Staging (published release)

`docker-compose-staging.yml` runs the four published Docker Hub images (`osctrl-tls`, `osctrl-api`, `osctrl-cli`, `osctrl-frontend`) instead of building from source. Use a release that publishes all four, such as `0.6.2`.

```bash
cp .env.staging.example .env.staging
make docker_staging_certs   # local self-signed certificate; refuses to overwrite
make docker_staging
```

Edit `.env.staging` first: set `OSCTRL_VERSION` (the image tag, without a leading `v`), independent random values for `JWT_SECRET` (at least 32 bytes), `POSTGRES_DB_PASSWORD` and `OSCTRL_PASS`, and `OSCTRL_USER`. Empty required values fail before Compose starts. Generate secrets with `openssl rand -hex 32`.

| Surface | URL |
| --- | --- |
| Operator frontend and `/api/*` | `https://localhost:9444` |
| osquery TLS endpoint | `https://localhost:9443` |
| PostgreSQL | `localhost:15432` |
| Redis | `localhost:16379` |

Everything binds to loopback by default. For a remote host set `STAGING_BIND_ADDRESS`, `STAGING_TLS_HOST`, `STAGING_FRONTEND_HOST` and a matching certificate (`STAGING_TLS_CERT`, `STAGING_TLS_KEY`) **before the first bootstrap**, since existing environment settings are preserved on later starts.

The bootstrap container creates the `staging` environment and the administrator if absent, then exits; later starts keep the data, and changing `OSCTRL_PASS` does not reset an existing password. The stack runs in the `osctrl-staging` Compose project, so it can run next to the development stack.

```bash
make docker_staging_pull    # download the selected images
make docker_staging_up      # pull and start in the background
make docker_staging_logs
make docker_staging_down    # stop, keep volumes
make docker_staging_check   # deployment contract tests, no daemon needed
```

To change release, back up the database, edit `OSCTRL_VERSION` and run `make docker_staging`. Startup migrates the schema, so going back to an older tag is not a rollback; restore a backup instead. Don't use `docker_dev_clean` while keeping staging data: it matches every osctrl image and volume.
