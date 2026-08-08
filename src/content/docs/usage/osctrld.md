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
  "osquery_version": "5.23.0"
}
```

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

### Package download

Route:

```properties
GET /{env_uuid}/{secret_path}/package/{package}
```

Supported package values:

* `deb`
* `rpm`
* `pkg`
* `msi`

If the environment package value is an `http` URL, `osctrl-tls` redirects to it. Otherwise it serves the local package file from the configured enroll package directory.

## Notes

* These endpoints are implemented inside `cmd/tls/main.go` and `cmd/tls/handlers/post.go` in the current upstream `develop` branch.
* The default osctrld route names are `osctrld-flags`, `osctrld-cert`, `osctrld-verify` and `osctrld-script`.
* The quick-link routes coexist with the standard osquery TLS routes such as `/{env}/enroll`, `/{env}/config`, `/{env}/log`, `/{env}/read`, `/{env}/write`, `/{env}/init` and `/{env}/block`.
