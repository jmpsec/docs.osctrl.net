---
title: "Install on Ubuntu"
description: "Deploy osctrl on Ubuntu with provision.sh, using nginx, PostgreSQL, Redis, systemd services, and the bundled operator frontend."
sidebar:
  order: 3
---

The fastest native deployment path is the `deploy/provision.sh` script from the **osctrl** repository. It installs the host dependencies, builds the binaries and frontend, writes service configuration, prepares PostgreSQL and Redis, and starts the services through systemd.

This tutorial walks through a single-server Ubuntu install. It is a good default for a lab, a small deployment, or the first production host you want to understand before automating it.

## What You Will Build

The command below installs:

* `osctrl-tls`, listening internally on port `9000` and exposed by nginx on `443`,
* `osctrl-api`, listening internally on port `9002` and exposed by nginx on `8444`,
* the React operator frontend, served by nginx on port `8443`,
* PostgreSQL as the backend database,
* Redis as the cache,
* an initial osctrl environment,
* an initial admin user if you pass `--admin`.

The script stores binaries and configuration under `/opt/osctrl` by default.

## Before You Start

Use a fresh Ubuntu 22.04 or 24.04 server with a sudo-capable user. The script installs packages and writes system files, so do not run it on a machine where nginx, PostgreSQL, or Redis are already carefully hand-managed unless you have reviewed the script first.

Open these ports to the host:

| Port | Purpose |
| --- | --- |
| `443` | osquery TLS endpoint |
| `8443` | operator frontend |
| `8444` | osctrl API |

Pick a hostname for the server. In the examples below, replace `osctrl.example.com` with your DNS name or server IP address.

## 1. Clone osctrl

Install `git` if the host does not have it yet, then clone the repository:

```bash
sudo apt-get update
sudo apt-get install -y git
git clone https://github.com/jmpsec/osctrl.git
cd osctrl
```

The provisioning script auto-detects this source path, but passing `--source "$PWD"` makes the command easier to read and repeat.

## 2. Run the Provisioning Script

For a first Ubuntu deployment, let the script install nginx, PostgreSQL, and Redis for you:

```bash
./deploy/provision.sh \
  --mode dev \
  --source "$PWD" \
  --dest /opt/osctrl \
  --part all \
  --nginx \
  --postgres \
  --redis \
  --all-hostname osctrl.example.com \
  --admin
```

In `dev` mode, the script creates self-signed certificates and a starter environment with short osquery intervals. That makes the first install easy to test. For production, use the production example later in this page.

The script may take a little while when nginx generates `dhparam.pem`. That pause is normal.

When the install finishes, it prints the operator URL and the generated admin credentials. Save the password somewhere safe; it is generated once.

## 3. What the Script Does

The script performs the boring setup work in the right order:

1. Detects Ubuntu and installs required packages such as `git`, `curl`, `make`, `gcc`, `openssl`, `bc`, and `rsync`.
2. Installs `yq`, Go, and Node.js through `nvm` when they are not already present.
3. Installs and starts PostgreSQL when `--postgres` is set.
4. Creates the `osctrl` PostgreSQL database and user.
5. Installs and starts Redis when `--redis` is set, using the default password from the script configuration.
6. Builds and installs `osctrl-cli`, `osctrl-tls`, `osctrl-api`, and the frontend.
7. Generates YAML configuration files in `/opt/osctrl/config`.
8. Verifies the generated service configuration.
9. Creates and starts systemd services for `osctrl-tls` and `osctrl-api`.
10. Configures nginx as the public TLS entry point.

That is why the one command is long: each flag makes one piece explicit.

## 4. Verify the Install

Check the two osctrl services:

```bash
systemctl status osctrl-tls
systemctl status osctrl-api
```

Check nginx:

```bash
sudo nginx -t
systemctl status nginx
```

Then open the operator frontend:

```text
https://osctrl.example.com:8443
```

Because `dev` mode uses a self-signed certificate, your browser will warn about the certificate. For a test host, accept the warning. For production, use your own certificate.

You can also check the TLS endpoint directly:

```bash
curl -k -I https://osctrl.example.com
```

A ready deployment returns an HTTP response instead of timing out or refusing the connection.

## 5. Find Logs and Configuration

The generated configuration lives here:

```text
/opt/osctrl/config/tls.yml
/opt/osctrl/config/api.yml
```

The installed binaries live here:

```text
/opt/osctrl/bin/
```

Use `journalctl` for service logs:

```bash
sudo journalctl -u osctrl-tls -f
sudo journalctl -u osctrl-api -f
```

nginx configuration is written under `/etc/nginx`, with certificates under `/etc/nginx/certs`.

## 6. Enroll the Server Itself

If you want the server to enroll itself with osquery during provisioning, add `--enroll`:

```bash
./deploy/provision.sh \
  --mode dev \
  --source "$PWD" \
  --dest /opt/osctrl \
  --part all \
  --nginx \
  --postgres \
  --redis \
  --all-hostname osctrl.example.com \
  --admin \
  --enroll
```

This asks `osctrl-cli` to generate a quick-add enrollment command for the initial environment and execute it on the host.

## 7. Use Your Own Certificate

For a production-style install, put your certificate and key on the host and run with `--mode prod --type own`:

```bash
./deploy/provision.sh \
  --mode prod \
  --type own \
  --certfile /etc/certs/osctrl.crt \
  --keyfile /etc/certs/osctrl.key \
  --source "$PWD" \
  --dest /opt/osctrl \
  --part all \
  --nginx \
  --postgres \
  --redis \
  --all-hostname osctrl.example.com \
  --admin
```

Make sure the certificate covers the hostname you pass with `--all-hostname`.

## 8. Upgrade Later

To rebuild from the latest code and restart the installed services, use `--upgrade`:

```bash
./deploy/provision.sh \
  --upgrade \
  --source "$PWD" \
  --dest /opt/osctrl \
  --part all
```

The upgrade path refuses to continue if the source checkout has local changes. Commit, stash, or discard those changes before upgrading.

## Useful Options

| Option | Use |
| --- | --- |
| `--mode dev` | Self-signed certificates and development-friendly defaults |
| `--mode prod` | Production mode |
| `--type own` | Use the certificate passed with `--certfile` and `--keyfile` |
| `--part tls` | Deploy only the TLS endpoint |
| `--part api` | Deploy only the API and frontend path |
| `--all-hostname` | Use the same hostname for all services |
| `--public-tls-port` | Change the public nginx port for the TLS endpoint |
| `--public-admin-port` | Change the public nginx port for the operator frontend; the option keeps the script's legacy name |
| `--public-api-port` | Change the public nginx port for `osctrl-api` |
| `--dest` | Change the install directory from `/opt/osctrl` |
| `--admin` | Create an initial admin user and print its password |
| `--password` | Set the initial admin password instead of generating one |

For the full option reference, see [the provision.sh usage page](/usage/provision.sh/).
