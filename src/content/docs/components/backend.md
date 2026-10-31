---
title: Backend
description: The centralized datastore where all osctrl data is stored.
sidebar:
  order: 7
---

<figure class="component-hero diagram-hero">
  <img class="only-light" src="/img/components-backend-light.svg" alt="osctrl-tls and osctrl-api reading and writing PostgreSQL and Redis" />
  <img class="only-dark" src="/img/components-backend-dark.svg" alt="" aria-hidden="true" />
</figure>

The backend component provides a centralized place where all the **osctrl** data is stored. In the diagram above, it is displayed with a [PostgreSQL](https://www.postgresql.org/) logo, and in fact, the default provisioning of **osctrl** does install PostgreSQL as backend. However, the code that handles database access uses [GORM](https://gorm.io/) so the changes to make **osctrl** work with a different backend solution would be minimal.

Each component uses the backend differently but all of them have read/write operations.

## Read replica

The database can optionally have a read replica. When it is configured, [osctrl-api](/components/osctrl-api/) sends its read-heavy, staleness-tolerant queries (dashboard, node lists, audit log, posture and vulnerability views) to the replica and keeps everything else on the primary: writes, authentication, enrollment, and the query and carve lifecycle. Replication lag therefore only affects what the dashboard and lists display, never authorization or a read inside a write flow.

Replicas are not supported with SQLite. See [Read replica](/configuration/#read-replica) for the settings.
