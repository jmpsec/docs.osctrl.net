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
* `/api/v1/mcp` hosts the authenticated MCP endpoint when `mcp.enabled` is true, using the caller's normal token, permissions and audit trail.
