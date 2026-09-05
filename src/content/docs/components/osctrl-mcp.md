---
title: osctrl-mcp
description: The Model Context Protocol component that exposes osctrl operations to MCP clients.
sidebar:
  order: 3
---

<figure class="component-hero diagram-hero">
  <img class="only-light" src="/img/components-osctrl-mcp-light.svg" alt="MCP clients reaching osctrl through osctrl-mcp and osctrl-api" />
  <img class="only-dark" src="/img/components-osctrl-mcp-dark.svg" alt="" aria-hidden="true" />
</figure>

The osctrl-mcp component is the Model Context Protocol bridge for **osctrl**. It gives MCP-capable assistants and automation tools a safer, structured way to work with osctrl data through [osctrl-api](/components/osctrl-api/) instead of connecting directly to the backend database.

Use it when you want an assistant to help with operator workflows: finding nodes, inspecting environment state, checking posture signals, preparing queries, or summarizing what is happening across the fleet. The component should be treated like any other privileged integration: run it close to `osctrl-api`, give it narrowly scoped credentials, and expose it only to trusted MCP clients.

## How It Fits

`osctrl-mcp` sits beside the human and script-facing interfaces:

* [osctrl-frontend](/components/osctrl-frontend/) is for browser operators,
* [osctrl-cli](/components/osctrl-cli/) is for shell automation,
* `osctrl-mcp` is for MCP clients that need typed tools and context.

All three should go through [osctrl-api](/components/osctrl-api/) for platform actions. That keeps authentication, authorization, audit logging, rate limiting and API validation in one place.

## Deployment Notes

Deploy `osctrl-mcp` as an internal service next to `osctrl-api`, with network access to the API endpoint and no direct access to PostgreSQL unless you have a very specific operational reason. In Docker or Kubernetes deployments, this usually means adding it to the same private network as `osctrl-api` and publishing only the MCP transport endpoint needed by your trusted client.

For production, create a dedicated osctrl service account for MCP usage, keep its token out of the frontend, and rotate it like any other integration secret.
