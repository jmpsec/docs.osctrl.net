---
title: osctrl-mcp
description: The Model Context Protocol component that exposes osctrl operations to MCP clients.
sidebar:
  order: 3
---

<figure class="component-hero diagram-hero">
  <img class="only-light" src="/img/components-osctrl-mcp-light.svg" alt="An MCP client spawns osctrl-mcp over stdio on one host, and osctrl-mcp calls osctrl-api" />
  <img class="only-dark" src="/img/components-osctrl-mcp-dark.svg" alt="" aria-hidden="true" />
</figure>

The osctrl-mcp component is the Model Context Protocol bridge for **osctrl**. It gives MCP-capable assistants and automation tools a safer, structured way to work with osctrl data through [osctrl-api](/components/osctrl-api/) instead of connecting directly to the backend database.

Use it when you want an assistant to help with operator workflows: finding nodes, inspecting environment state, checking posture signals, preparing queries, or summarizing what is happening across the fleet. The component should be treated like any other privileged integration: give it narrowly scoped credentials, and register it only with MCP clients you trust.

## How It Fits

`osctrl-mcp` sits beside the human and script-facing interfaces:

* [osctrl-frontend](/components/osctrl-frontend/) is for browser operators,
* [osctrl-cli](/components/osctrl-cli/) is for shell automation,
* `osctrl-mcp` is for MCP clients that need typed tools and context.

All three should go through [osctrl-api](/components/osctrl-api/) for platform actions. That keeps authentication, authorization, audit logging, rate limiting and API validation in one place.

## Deployment Notes

`osctrl-mcp` is not a daemon. It speaks MCP over stdio and is launched by the MCP client itself, so there is no listener to expose, no port to publish and no state to persist. It needs only network access to the `osctrl-api` endpoint, and no direct access to PostgreSQL.

It is read-only unless started with `--allow-writes`. Create a dedicated osctrl service account for MCP usage, scope it to what the agent should read, keep its token out of the frontend, and rotate it like any other integration secret.

See the [usage of osctrl-mcp](/usage/osctrl-mcp/) for the flags, the tool list and client registration examples.
