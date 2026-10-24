---
title: "Per-node overrides"
description: "Overriding the osquery configuration or flags of a single node, on top of its environment's."
sidebar:
  order: 5
---

An environment's osquery configuration and flags apply to every node in it. A **per-node override** changes them for one node without touching the rest. Both overrides are managed by environment admins, from the node detail page or the API, and can carry an expiry (`expires_at`); an expired override is no longer served.

## Configuration override

`GET`, `PUT` and `DELETE` on `/api/v1/nodes/{env}/node/{uuid}/config`. The body of a `PUT` is `{"override": "<json>", "expires_at": null}`.

- The override is **deep-merged** over the environment configuration: objects merge key by key; any other value, arrays included, replaces the environment's.
- It must be a JSON object. Known sections (such as `options`) must be objects, and `null` values are refused.
- Options that control how osquery talks to osctrl are refused, since changing them could cut the host off with no way to undo it from osctrl.
- The `GET` response shows the environment configuration, the override and the configuration the node is actually served.

## Flags override

`GET`, `PUT` and `DELETE` on `/api/v1/nodes/{env}/node/{uuid}/flags`. The override is one osquery flag per line; blank lines and `#` comments are ignored.

- Each override flag **replaces** every environment line with the same name, in place; flags the environment lacks are appended.
- Flags that control the connection to osctrl (and their `no` negations), repeated flags and an empty override are refused.
- A flag that is also a key of `options` in the environment or node configuration is reported as a conflict, because the same option would be set twice.
- Flags reach the host through [osctrld](/usage/osctrld/), so the node needs osctrld to apply them. The `GET` response includes the last fetch and the result osctrld reported; those apply fields come from the host and are untrusted.

## Deleting a node

`osctrl-cli node delete` also removes the node's configuration and flags overrides, osctrld status and extension assignments.
