---
title: "osquery extensions"
description: "Distributing osquery extension binaries to environments and nodes through osctrld."
sidebar:
  order: 6
---

osctrl keeps a library of osquery extension binaries and delivers them to hosts through [osctrld](/usage/osctrld/). Requires `osctrld.enabled` on `osctrl-tls`.

## Library

Managed by **super admins** in the **Extensions** page of [osctrl-frontend](/components/osctrl-frontend/) or through `/api/v1/extensions`:

| Method | Path | Purpose |
| --- | --- | --- |
| `GET` / `POST` | `/api/v1/extensions` | List the library / add an extension |
| `PATCH` / `DELETE` | `/api/v1/extensions/{id}` | Change / delete an extension |
| `PUT` / `DELETE` | `/api/v1/extensions/{id}/files/{platform}/{arch}` | Upload / remove the binary for one platform and architecture |

- Names match `^[A-Za-z0-9_-]{1,64}$` and are unique ignoring case, because the name becomes a file name on the host.
- Uploads are limited to 64 MiB. If you proxy the frontend with the bundled nginx configuration, `client_max_body_size` is `65m` to allow for multipart framing.
- An extension that is still assigned can't be deleted.

## Assignment

| Scope | Endpoints | Who |
| --- | --- | --- |
| Environment | `GET` `/api/v1/environments/extensions/{env}`, `PUT` / `DELETE` `/api/v1/environments/extensions/{env}/{id}` | Environment admins |
| Node | `GET` `/api/v1/nodes/{env}/node/{uuid}/extensions`, `PUT` / `DELETE` `/api/v1/nodes/{env}/node/{uuid}/extensions/{id}` | Environment admins |

An extension assigned to an environment runs on every node of it. **Command-execution** extensions (`command_execution: true`) can only be assigned to individual nodes, and ordinary ones only to environments.

## Delivery

osctrld asks `osctrl-tls` which extensions to run (the environment's plus its own node's), downloads each binary, and reports a status per extension. If a host can't remove an extension that was unassigned, it is reported as kept, and the library entry can't switch off command execution or be deleted while that is the case.

Downloads are rate limited per client IP (`rateLimits.osctrldDownload`).
