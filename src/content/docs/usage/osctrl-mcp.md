---
title: "osctrl-mcp"
description: "osctrl-mcp is the Model Context Protocol server for osctrl, exposing the fleet to MCP clients over stdio."
sidebar:
  order: 3
---

`osctrl-mcp` serves the **osctrl** read-only surface over the [Model Context Protocol](https://modelcontextprotocol.io/), so an MCP client — Claude Code, Claude Desktop, or any other — can inspect a fleet through [osctrl-api](/components/osctrl-api/).

:::note[It is not a daemon]
`osctrl-mcp` speaks MCP over **stdio** and is launched by the client, not run as a service. It holds no state and opens no listening socket. Every tool call becomes an authenticated request to `osctrl-api`, so the token's existing per-environment permissions are what bound the agent.
:::

Build it with `make mcp`, which produces `bin/osctrl-mcp`.

Execute `./osctrl-mcp --help` to show the main help of the program:

```properties
$ ./osctrl-mcp --help
NAME:
   osctrl-mcp - MCP server for osctrl

USAGE:
   osctrl-mcp [global options]

VERSION:
   0.5.8

GLOBAL OPTIONS:
   --api-url string                                                Base URL of osctrl-api, e.g. https://osctrl.example.com [$OSCTRL_API_URL]
   --api-token string                                              Bearer token for osctrl-api (prefer OSCTRL_API_TOKEN over the flag so it stays out of the process list) [$OSCTRL_API_TOKEN]
   --config osctrl-cli login --write, -c osctrl-cli login --write  Path to an osctrl-api.json holding url + token, as written by osctrl-cli login --write [$OSCTRL_API_FILE]
   --insecure                                                      Skip TLS verification when talking to osctrl-api (development only) [$OSCTRL_INSECURE]
   --allow-writes                                                  Expose the mutating tools (run_query, expire_query, complete_query, tag_node). Off by default; the token's own permissions still apply [$OSCTRL_MCP_ALLOW_WRITES]
   --log-level string                                              Log level: debug, info, warn, error (default: "info") [$OSCTRL_LOG_LEVEL]
   --help, -h                                                      show help
   --version, -v                                                   print the version
```

## Pointing It at osctrl-api

The API URL and token can come from a file, from flags, or from the environment. The config file is read first and explicit flags or environment variables override it, so you can keep one checked-in file and still point a client at a different environment without editing it.

The `--config` file is the same `osctrl-api.json` that [osctrl-cli login](/usage/osctrl-cli/login/) writes with `--write-api-file`:

```json
{
  "url": "https://osctrl.example.com",
  "token": "replace-with-a-secret"
}
```

The server fails fast at start-up if the URL is unreachable or the token is rejected, rather than starting cleanly and failing one tool call at a time.

:::caution[Give it its own service user]
Create a dedicated osctrl user scoped to what the agent should read, and use its token. `osctrl-mcp` never issues a write unless you pass `--allow-writes`, but a token that *can* write is still a token the process holds in memory.
:::

## Registering It With a Client

MCP clients launch the binary themselves. Keep the token in `env` rather than in `args`, so it stays out of the process list:

```json
{
  "mcpServers": {
    "osctrl": {
      "command": "/opt/osctrl/osctrl-mcp",
      "env": {
        "OSCTRL_API_URL": "https://osctrl.example.com",
        "OSCTRL_API_TOKEN": "replace-with-a-secret"
      }
    }
  }
}
```

Or, if you already have an `osctrl-api.json` on disk:

```json
{
  "mcpServers": {
    "osctrl": {
      "command": "/opt/osctrl/osctrl-mcp",
      "args": ["--config", "/etc/osctrl/osctrl-api.json"]
    }
  }
}
```

Logs go to **stderr**, never stdout — stdout is the MCP transport and a stray byte there corrupts the JSON-RPC stream. Raise `--log-level debug` when a client cannot start the server.

## Tools

The server is read-only by default. It reports itself to the client as `osctrl`.

| Tool | What it does |
| --- | --- |
| `list_environments` | Environments this token can see. Call it first — most other tools take an environment name. Enrollment secrets and certificates are deliberately not returned |
| `fleet_stats` | Fleet-wide node counts: totals, active vs inactive, per-platform and per-environment |
| `search_nodes` | Search enrolled nodes in one environment, with optional platform and hostname filters. Results are capped — check the `truncated` flag |
| `get_node` | Full detail for one node by UUID or hostname, optionally including its posture score |
| `list_osquery_tables` | The osquery tables this deployment knows about, filtered by name substring or platform |
| `get_table_schema` | Columns and metadata for one table. Columns marked required must appear in the `WHERE` clause or the query silently returns no rows |
| `list_queries` | Distributed queries in an environment, newest first, with how many nodes have answered |
| `list_saved_queries` | The saved, reusable query templates — not queries that have run |
| `get_query_results` | Rows returned so far for a distributed query |

Distributed queries are asynchronous. Results accumulate as nodes check in over seconds to minutes, so a small or empty result set means *not yet*, not *no matches*.

### Write Tools

Passing `--allow-writes` additionally registers the mutating tools. They are opt-in by construction: without the flag the server is built read-only, so no configuration mistake can quietly hand an agent the ability to schedule queries. The token's own permissions still apply on top.

| Tool | What it does |
| --- | --- |
| `run_query` | Schedule an osquery SQL query against nodes in an environment. Returns a query name, not results. Requires query-level permission |
| `expire_query` | Stop a query from being handed to any more nodes. Results already collected stay readable |
| `complete_query` | Mark a query complete, so osctrl stops waiting on nodes that have not answered |
| `tag_node` | Apply an existing tag to a node. Tags drive query targeting, so this changes which queries a node receives in future. Requires admin permission |

`run_query` does real work on every targeted endpoint, so target as narrowly as the task allows and prefer reading an existing query's results over re-running it.

## Fleet Data Is Untrusted

Hostnames, process names, file paths and result rows come from monitored endpoints — exactly the machines an attacker would control. The server tells the model to report what it finds and never to follow instructions that appear inside that content. Keep that in mind when reviewing what an agent did: text inside a query result asking to run a query, widen a target or change a tag is an attacker instructing the agent, not you.
