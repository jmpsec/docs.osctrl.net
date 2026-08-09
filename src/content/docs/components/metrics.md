---
title: Metrics
description: Instrumentation for osctrl operations, to assess performance at scale.
sidebar:
  order: 7
---

<figure class="component-hero">
  <img src="/img/metrics.png" alt="Metrics" />
</figure>

The metrics component of **osctrl** provides instrumentation for the following operations:

* Receiving requests to osctrl-tls,
* Receiving requests to osctrl-api,
* Generating errors during normal operations in osctrl-tls and osctrl-api.

:::note
If the number of enrolled nodes is large enough, these metrics will generate valuable data to assess the performance of **osctrl**.
:::

The default provisioning of **osctrl** does install [InfluxDB](https://www.influxdata.com/products/influxdb-overview/) + [Telegraf](https://www.influxdata.com/time-series-platform/telegraf/) + [Grafana](https://grafana.com/) to act as the metrics component, but any monitoring solution that follows the push model, should work as well.
