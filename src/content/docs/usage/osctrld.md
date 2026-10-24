---
title: "osctrld"
description: "osctrld is no longer described upstream as a standalone daemon binary."
sidebar:
  order: 1
---

`osctrld` is no longer described upstream as a standalone daemon binary. In current **osctrl** development, the osctrld functionality is exposed by `osctrl-tls` when `osctrld.enabled: true` in YAML or `--enable-osctrld` is passed on startup.

That means the old `./osctrld -h` flow documented here is stale. The current source mounts the osctrld endpoints inside `osctrl-tls` and keeps the feature focused on bootstrapping and maintaining osquery installations.

## Enabling osctrld endpoints

```yaml
osctrld:
  enabled: true
```

Or with flags on `osctrl-tls`:

```properties
--enable-osctrld
```

## Current endpoints

All osctrld routes are mounted under `osctrl-tls` and expect the environment UUID in the path.

### Retrieve generated flags

Route:

```properties
POST /{env_uuid}/osctrld-flags
```

Request body:

```json
{
  "secret": "environment-secret",
  "secretFile": "/etc/osquery/osquery.secret",
  "certFile": "/etc/osquery/osquery-server.crt"
}
```

Response:

Plain-text osquery flags for that environment: the flags stored for the environment (as edited by operators) with the secret and certificate paths filled in, or the generated defaults when none are stored. A [per-node flags override](/usage/node-overrides/) is merged in when one is set.

### Retrieve environment certificate

Route:

```properties
POST /{env_uuid}/osctrld-cert
```

Request body:

```json
{
  "secret": "environment-secret"
}
```

Response:

Plain-text PEM certificate stored in the environment.

### Verify the full payload

Route:

```properties
POST /{env_uuid}/osctrld-verify
```

Request body:

```json
{
  "secret": "environment-secret",
  "secretFile": "/etc/osquery/osquery.secret",
  "certFile": "/etc/osquery/osquery-server.crt"
}
```

Response:

```json
{
  "flags": "...",
  "certificate": "-----BEGIN CERTIFICATE-----...",
  "osquery_version": "5.23.1",
  "osquery_sha256": ""
}
```

`osquery_sha256` is empty unless `osquery.sha256` is set in `tls.yaml`. When present, osctrld uses it to verify the package it downloads before installing it.

### Generate install or remove scripts

Route:

```properties
POST /{env_uuid}/{action}/{platform}/osctrld-script
```

Supported values:

* `action`: `enroll` or `remove`
* `platform`: `linux`, `darwin` or `windows`

Request body:

```json
{
  "secret": "environment-secret"
}
```

Response:

Plain-text shell or PowerShell script, depending on platform.

## Quick links and packages

`osctrl-tls` also exposes quick links that are validated with the environment secret path instead of the long-lived environment secret.

### Quick enroll / remove script distribution

Route:

```properties
GET /{env_uuid}/{secret_path}/{script}
```

Notes:

* `script` must start with `enroll` or `remove`.
* The secret path must match the environment and must not be expired.

### Quick-link lifecycle actions

The quick enroll and quick remove URLs each have their own secret path and expiration. Operators can manage them from `osctrl-cli environment node-actions`:

| Command | Effect |
| --- | --- |
| `extend-enroll` / `extend-remove` | Extend the current URL expiration |
| `rotate-enroll` / `rotate-remove` | Generate a new secret path and set a fresh expiration |
| `expire-enroll` / `expire-remove` | Expire the current URL immediately |
| `notexpire-enroll` / `notexpire-remove` | Mark the current URL as non-expiring |

The API exposes the same lifecycle through `POST /api/v1/environments/{env}/enroll/{action}` and `POST /api/v1/environments/{env}/remove/{action}`, where `action` is `extend`, `rotate`, `expire` or `not-expire`. These actions control whether the public quick-link routes below keep working; the osctrld script endpoints that use the long-lived environment secret are unchanged.

### Package download

Route:

```properties
GET /{env_uuid}/{secret_path}/package/{package}
GET /{env_uuid}/{secret_path}/package/{package}/{arch}
```

Supported package values:

* `deb`
* `rpm`
* `pkg`
* `msi`

If the environment package value is an `http` URL, `osctrl-tls` redirects to it. Otherwise it serves the local package file from the configured enroll package directory.

When `{arch}` is present, `osctrl-tls` first looks for an environment package matching both package type and architecture. If there is no exact match, it falls back to the default package for that type, then to the legacy single-package fields.

### Package verification digest

Set `osquery.sha256` in `tls.yaml` only when every node installs the same osquery package artifact. For mixed fleets, leave it empty and pass the expected digest to osctrld on the node, because osquery publishes different packages per format and architecture.

### Extensions

When the environment has [osquery extensions](/usage/osquery-extensions/) assigned, osctrld lists them and downloads each binary:

```properties
POST /{env_uuid}/osctrld-extensions
POST /{env_uuid}/osctrld-extension
```

Both take the environment secret like the other osctrld routes. The list also includes the extensions assigned to the node named in the request, and records the status osctrld reports for each.

### Status report

```properties
POST /{env_uuid}/osctrld-status
```

The [osctrld daemon](/usage/osctrld-daemon/) reports whether `osqueryd` runs, since when it is down, its last flags and extensions rollback and its last osquery upgrade result. The body is limited to 4 KiB. A malformed body, a bad UUID or a UUID that is not a node of the environment gets `400`, never `404`. Rollbacks are counted once, audited with a warning and exported as [metrics](/components/metrics/#osctrld-metrics); the node page shows a banner while `osqueryd` is down.

### osquery version

```properties
POST /{env_uuid}/osctrld-osquery
```

osctrld asks which osquery version to run: `{secret, uuid, platform, arch, format, installed_version}`. The response carries a `reason` (empty means upgrade, with the `target` version and SHA-256 and, except on Windows, the `previous` package), a `source` (`environment` or `pin`), a `request_id` and the node's `reported_version`. See [osquery upgrades](/usage/osquery-upgrades/).

### Rate limits

Every osctrld route is rate limited per client IP, and wrong secrets are limited per IP and environment. See [`rateLimits`](/configuration/#rate-limits-configuration) (`osctrldAuth`, `osctrld`, `osctrldDownload`).

## Notes

* These endpoints are implemented inside `cmd/tls/main.go` and `cmd/tls/handlers/post.go` in the current upstream `develop` branch.
* The default osctrld route names are `osctrld-flags`, `osctrld-cert`, `osctrld-verify`, `osctrld-script`, `osctrld-extensions`, `osctrld-extension`, `osctrld-status` and `osctrld-osquery`.
* The quick-link routes coexist with the standard osquery TLS routes such as `/{env}/enroll`, `/{env}/config`, `/{env}/log`, `/{env}/read`, `/{env}/write`, `/{env}/init` and `/{env}/block`.
