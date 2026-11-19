---
title: osctrl-tls
description: The TLS endpoint that implements the osquery remote API, grouping enrolled nodes into environments.
sidebar:
  order: 1
---

<figure class="component-hero diagram-hero">
  <img class="only-light" src="/img/components-osctrl-tls-light.svg" alt="osctrl-tls between osquery nodes, nginx, the backend and the log sinks" />
  <img class="only-dark" src="/img/components-osctrl-tls-dark.svg" alt="" aria-hidden="true" />
</figure>

The osctrl-tls component is the actual TLS endpoint that implements the [remote API](https://osquery.readthedocs.io/en/stable/deployment/remote/) of [osquery](https://osquery.io).

Inside of the osctrl-tls, you can run a number of TLS endpoints, to group the nodes that will enroll in each of them. Within **osctrl** this concept is called an **environment**.

Each **environment** implements handlers to manage the following actions:

* Enroll nodes,
* Serve osquery configuration to enrolled nodes,
* Log osquery status messages and osquery results from enrolled nodes,
* Run on-demand queries in enrolled nodes and collect results,
* Perform file carving operations in enrolled nodes.

osctrl-tls can send its alert rule node scans and osctrld Prometheus gauges to a [database read replica](/configuration/#read-replica); the osquery and osctrld request paths stay on the primary.

Inside **osctrl** there can be *many* environments, like a typical environment where you have corporate and production systems. Then you would have one environment for corp, and another one for prod. The reasoning is, if you may need to distribute different configurations to the enrolled nodes, then you should use different environments.

## Enrolling osquery nodes

As mentioned before, osquery nodes must be enrolled inside of an environment. In order to achieve that, osquery requires an enrolling secret that will be stored in disk. Also, and given that osquery communicates with **osctrl** using TLS, a certificate may also be needed.

For example, here are the flags to enroll osquery nodes in the *dev* environment, used as example when deploying **osctrl** locally. The important part are the path for the different requests to arrive and the intervals in seconds:

```properties
--host_identifier=uuid
--force=true
--utc=true
--enroll_secret_path=__SECRET_FILE__
--enroll_tls_endpoint=/dev/enroll
--config_plugin=tls
--config_tls_endpoint=/dev/config
--config_tls_refresh=300
--config_tls_max_attempts=5
--logger_plugin=tls
--logger_tls_compress=true
--logger_tls_endpoint=/dev/log
--logger_tls_period=600
--disable_carver=false
--carver_disable_function=false
--carver_start_endpoint=/dev/init
--carver_continue_endpoint=/dev/block
--disable_distributed=false
--distributed_interval=60
--distributed_plugin=tls
--distributed_tls_max_attempts=3
--distributed_tls_read_endpoint=/dev/read
--distributed_tls_write_endpoint=/dev/write
--tls_dump=true
--tls_hostname=osctrl-nginx
--tls_server_certs=__CERT_FILE__
```

The values of `__CERT_FILE__` and `__SECRET_FILE__` will be the local full path for certificate and secret respectively.

## Scaling osctrl-tls

The idea behind making osctrl-tls its own component is all about scalability. In a scenario with many nodes enrolled, the number of requests is going to increase quickly, especially if we lower the intervals.

Running multiple instances of osctrl-tls will help to scale the service and be reliable for a larger number of nodes.

For example, using the intervals of the flags above, where logs, configuration and queries run every 600, 300 and 60 seconds. Presuming that you have 1000 machines and you want to enroll them all in **osctrl**.

## Node keys and request handling

* **Node keys** are 32 random bytes (64 hex characters), with nothing derived from the host or the enrollment time. Nodes enrolled before this change keep their old key until they re-enroll; re-enroll the fleet if those keys must be rotated.
* **Shared node key cache.** Node keys are cached in Redis and shared by osctrl-api and every osctrl-tls replica, so deleting or re-enrolling a node revokes its old key everywhere at once. Changes made with `osctrl-cli --db` don't touch Redis, so an old key can stay valid for up to the 10-minute cache lifetime.
* **Re-enrollment is scoped to the environment.** Enrolling with one environment's secret only finds and updates the node row in that environment, never a node with the same UUID in another one.
* **Unknown environments** get exactly the response a known environment gives to bad credentials (for example `200 {"node_invalid": true}` on config, log and query endpoints, 403 on enroll), so responses don't reveal which environment UUIDs exist. They are logged at debug level and counted in `osctrl_tls_unknown_environment_total`.
* **Ingest budgets.** Log batches are stored before the node gets its answer, and batches in flight share a 256 MiB budget; when it is full the node gets 503 and retries (`osctrl_tls_log_backpressure_total`). Distributed query results have their own 256 MiB budget, so large results can't starve log ingestion.
* **Log types.** The log endpoint accepts only `result` and `status` logs, the two osquery's TLS logger sends. Every other type (`query`, `carve.meta`, `carve.data`, `santa`) is a sink category osctrl emits itself, so a node sending one gets 400 and cannot forge it into the log sinks.
* **Query results** are only accepted from a node that has a pending assignment for that query, and a retried response changes nothing. Carve init and block uploads are scoped to the requesting node and its environment.
* With `santaEnabled`, osctrl-tls also serves the [Santa sync protocol](/usage/santa/) at `POST /{env}/santa/sync/{stage}/{machineid}`.
