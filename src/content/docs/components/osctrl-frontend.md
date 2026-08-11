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
