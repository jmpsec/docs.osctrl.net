---
title: "Roles and permissions"
description: "Global roles, per-environment query and carve tiers, tag grants, sensitive nodes and sensitive carve paths."
sidebar:
  order: 3
---

osctrl checks every request in osctrl-api against two layers: **global roles**, which apply across all environments, and **per-environment access**, which says what a user can do inside one environment. The [frontend](/components/osctrl-frontend/#roles-and-access) hides what a user can't do, but the server is authoritative.

On upgrade nothing changes: existing global admins become superusers, existing query and carve grants read as the **standard** tier, and no tag or path is sensitive until someone marks it.

## Global roles

| Role | Can do |
| --- | --- |
| **Superuser** (the `admin` flag) | Everything. Sole owner of service config, settings, log sinks, auth providers, the extensions catalog, global osquery versions and vulnerability sync. |
| **User management** | Create, edit and delete users; set their environment permissions and non-superuser roles; manage other users' tokens. Cannot grant superuser, or edit, delete or re-token a superuser. |
| **Environment management** | Create environments (becoming env admin of each one it creates). Rename, update and delete only environments where it is env admin. |
| **Admin read-only** | Global read views: users, the full audit log, events, health, environment map, platforms, all tags, node lookup, vulnerability feeds, alerts. Reads nodes, queries and carves at the standard tier in every environment, and never runs anything. |
| **Alert admin** | Manage alert rules and channels. |

User management and environment management also include admin read-only. Alert admin does not.

Log-content alert rules (`result_log`, `query_log`, `status_log`) need the **full** query tier in the rule's environment and are never global, except for superusers. Channels used by those rules, and revealing channel secrets, are superuser-only.

Set roles with `osctrl-cli user add` / `user edit` (`--user-admin`, `--env-manager`, `--read-only-admin`, `--alert-admin`, and `--no-…` on edit), from the **Users** page, or through `POST /api/v1/users/{username}/edit`. See [osctrl-cli user](/usage/osctrl-cli/user/).

## Per-environment access

Each user has, per environment, a user level (paged node list and node logs), an env admin flag, and a **tier** for queries and another for carves:

| Tier | Nodes reached |
| --- | --- |
| `none` | No queries or carves. |
| `tags` | Only nodes carrying one of the user's **scope tags** in that environment. Sensitive nodes are excluded. |
| `standard` | Every node in the environment except sensitive ones. |
| `full` | Every node, including sensitive nodes and sensitive carve paths. Env admins and superusers are always full. |

The tier decides which nodes a query or carve is sent to, and which nodes the user sees in node lists, node detail, posture and query results (JSON and CSV, from the DB and S3 log readers). A node outside the caller's tier returns the same 404 as a node that doesn't exist, and naming one explicitly as a target creates nothing. A query that spans environments is judged against the caller's **lowest** tier among them.

Set tiers with `osctrl-cli user change-permissions` / `reset-permissions` (`--query-tier`, `--carve-tier`, `--scope-tags`), from the **Users** page, or with `query_tier`, `carve_tier` and `scope_tags` on `POST /api/v1/users/{username}/permissions`. The legacy on/off flags still work: on grants standard (or keeps the current tier) and off sets none.

Scope tags must belong to that environment and can't be sensitive. Deleting a tag removes it from every grant and cancels pending queries whose creator reached the node only through that tag.

## Sensitive nodes

A tag marked **sensitive** makes every node carrying it reachable only at the full tier: domain controllers, key-management hosts, executive laptops. A node stops being sensitive as soon as the tag is removed from it or deleted.

Env admins set the flag on the **Tags** page, with `osctrl-cli tag add|edit --sensitive`, or with `"sensitive": true` on the tag API. Marking a tag sensitive removes it from every scope grant and cancels pending queries and carves from users who can no longer reach those nodes.

Aggregate counts (dashboard stats, platform and version counts, query execution counts) still include sensitive nodes. The audit log and alert history stay fully visible to read-only admins.

## Sensitive carve paths

Each environment has a list of path prefixes that only full-tier users can carve, for example key material, credential stores or the Windows SAM hive. Below full carve tier, a carve whose path touches the list is refused (403 `path is sensitive`), and existing carves of listed paths are hidden, including carves taken before the path was listed. Adding a path cancels pending carves of it from users below full.

Matching only looks at the path text:

* Paths are normalized (Unicode NFC, always case-insensitive, `\` treated as `/`, cleaned, trailing dots and spaces trimmed per segment).
* A path **touches** a prefix when it is the prefix, lies under it, or is a parent of it, so carving `/etc` is refused when `/etc/shadow` is listed. A root touches everything on its volume; `/dev` and `/proc` touch whenever the list is non-empty.
* On macOS, paths under `/private/` and `/System/Volumes/Data/` are also checked with that prefix stripped.
* Paths with `..`, `~` (8.3 short names), NUL bytes, UNC or device prefixes, or NTFS alternate data streams are refused outright.
* Symlinks, hard links and junctions are not resolved.

Manage the list from the **Sensitive paths** tab of the environment config page, with [`osctrl-cli environment sensitive-paths`](/usage/osctrl-cli/environment/#sensitive-carve-paths), or through `GET`/`POST /api/v1/environments/sensitive-paths/{env}` and `DELETE /api/v1/environments/sensitive-paths/{env}/{id}`. Paths must be absolute and at most 512 bytes after normalization.

:::note
`osctrl-cli --db` applies the same validation, but does not cancel pending queries or carves when a tag or path becomes sensitive. Results are still hidden when read.
:::
