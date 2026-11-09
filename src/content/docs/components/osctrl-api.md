---
title: osctrl-api
description: The REST API to interact with enrolled nodes and with osctrl itself.
sidebar:
  order: 2
---

<figure class="component-hero diagram-hero">
  <img class="only-light" src="/img/components-osctrl-api-light.svg" alt="osctrl-api between the frontend, CLI and MCP bridge and the backend" />
  <img class="only-dark" src="/img/components-osctrl-api-dark.svg" alt="" aria-hidden="true" />
</figure>

The osctrl-api component of **osctrl** provides an API interface to interact with its enrolled nodes and with **osctrl** itself.

You can find the updated [API documentation](/api/) for **osctrl**, generated using [Starlight OpenAPI](https://starlight-openapi.vercel.app/).

It also exposes two newer operator-facing surfaces when enabled:

* `/api/v1/events` streams best-effort Server-Sent Events hints for live frontend updates. The stream does not replace REST snapshots; clients still refetch the affected data after `resource.changed`.
* `/api/v1/santa/{env}/...` manages the [Santa sync server](/usage/santa/) (config, token, rules, machines, events) when `santaEnabled` is true.
* `/api/v1/vulnerabilities` serves [vulnerability monitoring](/usage/vulnerabilities/) findings when `vulnEnabled` is true.
* Optionally reads dashboard, node list, audit log, posture and vulnerability data from a [database read replica](/configuration/#read-replica).
* `/api/v1/mcp` hosts the authenticated MCP endpoint when `mcp.enabled` is true, using the caller's normal token, permissions and audit trail.

Access to every route is checked against the caller's [roles and per-environment tiers](/usage/rbac/).

## Request limits

* Every authenticated request body is capped at **8 MiB** (large enough for an environment configuration with packs). Handlers with a smaller cap keep it; multipart uploads enforce their own limit.
* Query text is limited to **64 KiB** when running an on-demand query and when creating or updating a saved query (400 `query too long`). MCP and the CLI's API mode go through the same handlers; `osctrl-cli --db` is not limited.
* Login and MFA bodies are capped at 64 KiB, usernames at 256 bytes and passwords at 1024 bytes. A malformed login body returns 400.
