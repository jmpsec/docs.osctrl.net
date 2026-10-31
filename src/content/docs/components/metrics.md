---
title: Metrics
description: Instrumentation for osctrl operations, to assess performance at scale.
sidebar:
  order: 8
---

<figure class="component-hero diagram-hero">
  <img class="only-light" src="/img/components-metrics-light.svg" alt="osctrl-tls and osctrl-api reporting metrics" />
  <img class="only-dark" src="/img/components-metrics-dark.svg" alt="" aria-hidden="true" />
</figure>

The metrics component of **osctrl** provides instrumentation for the following operations:

* Receiving requests to osctrl-tls,
* Receiving requests to osctrl-api,
* Generating errors during normal operations in osctrl-tls and osctrl-api.

:::note
If the number of enrolled nodes is large enough, these metrics will generate valuable data to assess the performance of **osctrl**.
:::

The default provisioning of **osctrl** does install [InfluxDB](https://www.influxdata.com/products/influxdb-overview/) + [Telegraf](https://www.influxdata.com/time-series-platform/telegraf/) + [Grafana](https://grafana.com/) to act as the metrics component, but any monitoring solution that follows the push model, should work as well.

## osctrld metrics

With `--metrics-enabled` and `osctrld.enabled`, `osctrl-tls` also exports gauges and counters about hosts running [osctrld](/usage/osctrld-daemon/):

| Metric | Type | Meaning | Aggregate across replicas |
| --- | --- | --- | --- |
| `osctrld_osquery_down{environment}` | gauge | Nodes whose report is under 15 minutes old and says `osqueryd` is down | `max by (environment)` |
| `osctrld_osquery_behind{environment}` | gauge | Nodes below their effective osquery target (registered when the version library is enabled) | `max by (environment)` |
| `osctrld_osquery_rollout_paused{environment}` | gauge | 1 while the environment's latest rollout is paused | `max by (environment)` |
| `osctrld_rollbacks_total{environment,kind,stage}` | counter | Flags/extensions rollbacks reported by hosts | `sum` |
| `osctrld_osquery_upgrade_rollbacks_total{environment}` | counter | osquery upgrades rolled back | `sum` |

The gauges are read from the database ([read replica](/configuration/#read-replica) when one is configured), so every replica exports the same value; a database error exports nothing for that scrape. Pair an alert on them with an `absent()` rule if a database outage must not resolve it.
