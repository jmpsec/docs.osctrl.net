---
title: Components
description: Diagram of all the osctrl components in action and how they interact.
sidebar:
  order: 0
  label: Overview
---

<figure class="component-hero diagram-hero">
  <img class="only-light" src="/img/components-light.svg" alt="Current osctrl architecture diagram" />
  <img class="only-dark" src="/img/components-dark.svg" alt="" aria-hidden="true" />
</figure>

Diagram of the current osctrl components in action. `osctrl-mcp` sits beside `osctrl-cli` as another automation interface backed by `osctrl-api`.

## The components

| Component | Role |
| --- | --- |
| [osctrl-tls](/components/osctrl-tls/) | TLS endpoint implementing the osquery remote API |
| [osctrl-api](/components/osctrl-api/) | REST API for nodes and for osctrl itself |
| [osctrl-mcp](/components/osctrl-mcp/) | Model Context Protocol bridge for assistant and automation workflows |
| [osctrl-frontend](/components/osctrl-frontend/) | Browser operator interface backed by osctrl-api |
| [osctrl-cli](/components/osctrl-cli/) | Command line interface for automation |
| [nginx](/components/nginx/) | TLS termination and load balancing |
| [Backend](/components/backend/) | Centralized storage for all osctrl data |
| [Metrics](/components/metrics/) | Instrumentation for osctrl operations |
| [osctrld](/components/osctrld/) | Bootstrap and maintenance of osquery installations |
