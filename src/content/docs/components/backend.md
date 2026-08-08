---
title: Backend
description: The centralized datastore where all osctrl data is stored.
sidebar:
  order: 6
---

<figure class="component-hero">
  <img src="/img/backend.png" alt="Backend" />
</figure>

The backend component provides a centralized place where all the **osctrl** data is stored. In the diagram above, it is displayed with a [PostgreSQL](https://www.postgresql.org/) logo, and in fact, the default provisioning of **osctrl** does install PostgreSQL as backend. However, the code that handles database access uses [GORM](https://gorm.io/) so the changes to make **osctrl** work with a different backend solution would be minimal.

Each component uses the backend differently but all of them have read/write operations.
