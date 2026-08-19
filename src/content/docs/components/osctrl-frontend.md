---
title: osctrl-frontend
description: The React operator frontend for managing osctrl through osctrl-api.
sidebar:
  order: 4
---

The osctrl-frontend component is the operator UI for **osctrl**. It provides the browser experience for managing environments, nodes, queries, saved queries, carves, tags, users, settings and posture data.

It replaces the old `osctrl-admin` interface. New operator-facing improvements are built here, as a React single-page application that talks to [osctrl-api](/components/osctrl-api/) instead of rendering pages from a separate admin service.

In production, the frontend is built into static files and served by nginx or another static web server. API calls go to `osctrl-api`, usually through the same nginx entry point, so the browser can use same-origin cookies and keep the deployment simple.

## Tech Stack

The frontend is built with:

* React 19 and TypeScript 7,
* Vite 8 for development and production builds,
* TanStack Router, Query and Table for routing, API state and data grids,
* Tailwind CSS 4 and Radix UI primitives for the interface,
* react-hook-form and zod for forms and validation,
* Monaco Editor for query editing,
* Vitest, Testing Library and Playwright for tests.

For local development, the Vite server runs from the `frontend/` directory and proxies `/api/*` requests to `osctrl-api`. In the Docker development stack, nginx exposes the frontend at `https://localhost:8444`.

## Node console

Each node detail page can open a live interactive console that runs osquery against that single node in real time. It's gated by the `osquery.console` [configuration](/configuration/#osquery-configuration) key on `osctrl-api` (and `query: true`) — disabled by default.

Besides `pwd`, `cd <path>`, `ls [path]`, `get <path>` (carve a file), `ps`, `sql <select ...>`, `osquery` (switch to raw query mode), `help` and `clear`, the console supports these built-in commands:

| Command | Arguments | What it queries |
|---|---|---|
| `hash` | `<path>` | File hashes (`hash`) |
| `mime` / `magic` | `<path>` | File type (`magic`) |
| `curl` | `<url>` | HTTP(S) request (`curl`) |
| `ports` | | Listening ports |
| `sockets` | | Open sockets (`process_open_sockets`) |
| `lsof` | `<pid>` | Open files for a process |
| `env` | `<pid>` | Environment variables for a process |
| `routes` | | Routing table |
| `dns` | | DNS resolvers |
| `users` | | Local users |
| `loggedin` | | Logged-in users |
| `sudoers` | | Sudoers |
| `autoruns` | | Autostart executables |
| `cron` | | Crontab entries |
| `launchd` | | launchd jobs (macOS) |
| `services` | | Services (Windows) |
| `tasks` | | Scheduled tasks (Windows) |
| `certs` | | Installed certificates |
| `diskenc` | | Disk encryption status |
| `os` | | OS version |
| `sysinfo` | | System info |
| `uptime` | | System uptime |
| `packages` | | Installed packages (platform-dependent) |
| `extensions` | | Browser extensions |
| `firewall` | | Firewall rules (platform-dependent) |

Type `.tables`/`.exit` inside `osquery` mode to list tables or return to the console.

## File explorer

Each node detail page also offers a **File Explorer** tab: a tree view of that node's filesystem, browsed live over osquery. It's gated by the `osquery.fileExplorer` [configuration](/configuration/#osquery-configuration) key on `osctrl-api` (and `query: true`) — disabled by default.

Opening the tab creates a file explorer session for that node, rooted at the platform default (`/` on Linux/macOS, `C:\` on Windows), and closes it when you leave the page. Expanding a directory dispatches a hidden distributed query (`select ... from file where directory = ...`) and renders the results as child rows; selecting a file dispatches a `stat`-style query (`select ... from file where path = ...`) and shows its path, type, size, mode, uid/gid, and modified/accessed/changed times in the details pane.

From the details pane you can carve the selected file directly — it starts a regular [carve](/usage/osctrl-cli/carve/) for that path on the node and links to the resulting carve once it's created.

The file explorer reuses the same session-priming and query-acceleration mechanism as the [node console](#node-console): opening the tab also dispatches a lightweight `osquery_info` query so live version/uptime metadata shows in the header, and its presence in the node's queue can trigger accelerated polling when `osquery.accelerated` is enabled.
