---
title: osctrl-tls
description: The TLS endpoint that implements the osquery remote API, grouping enrolled nodes into environments.
sidebar:
  order: 1
---

<figure class="component-hero">
  <img src="/img/osctrl-tls.png" alt="osctrl-tls" />
</figure>

The osctrl-tls component is the actual TLS endpoint that implements the [remote API](https://osquery.readthedocs.io/en/stable/deployment/remote/) of [osquery](https://osquery.io).

Inside of the osctrl-tls, you can run a number of TLS endpoints, to group the nodes that will enroll in each of them. Within **osctrl** this concept is called an **environment**.

Each **environment** implements handlers to manage the following actions:

* Enroll nodes,
* Serve osquery configuration to enrolled nodes,
* Log osquery status messages and osquery results from enrolled nodes,
* Run on-demand queries in enrolled nodes and collect results,
* Perform file carving operations in enrolled nodes.

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
