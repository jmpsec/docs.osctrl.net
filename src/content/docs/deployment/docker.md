---
title: "Using Docker"
description: "Run osctrl and all its components locally with Docker Compose using the docker-compose-dev.yml stack."
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
