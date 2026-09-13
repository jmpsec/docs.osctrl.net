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

Plain-text osquery flags generated for that environment.

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

## Notes

* These endpoints are implemented inside `cmd/tls/main.go` and `cmd/tls/handlers/post.go` in the current upstream `develop` branch.
* The default osctrld route names are `osctrld-flags`, `osctrld-cert`, `osctrld-verify` and `osctrld-script`.
* The quick-link routes coexist with the standard osquery TLS routes such as `/{env}/enroll`, `/{env}/config`, `/{env}/log`, `/{env}/read`, `/{env}/write`, `/{env}/init` and `/{env}/block`.
