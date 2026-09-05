---
title: osctrld
description: Bootstrap and maintenance of osquery installations, served by osctrl-tls endpoints.
sidebar:
  order: 9
---

<figure class="component-hero">
  <img src="/img/osctrld.png" alt="osctrld" />
</figure>

The `osctrld` functionality in **osctrl** is the bootstrap and maintenance surface used to prepare osquery installations with environment-specific flags, certificates, scripts and packages.

:::note[No longer a standalone binary]
In current upstream **osctrl**, `osctrld` is no longer documented as a standalone binary. Instead, the osctrld functionality lives behind dedicated endpoints exposed by `osctrl-tls` when `osctrld.enabled: true` or `--enable-osctrld` is set.
:::

Those endpoints are intended to help bootstrap and manage osquery installations without having to use the operator frontend or call the API manually. They currently cover:

* Retrieving generated osquery flags,
* Retrieving the environment certificate,
* Verifying the full osquery enrollment payload,
* Generating install and remove scripts for Linux, macOS and Windows,
* Serving expiring quick-enroll / quick-remove links,
* Serving install packages for `deb`, `rpm`, `pkg` and `msi`.

The endpoints are mounted inside `osctrl-tls`, use the environment UUID in the path, and validate either the environment secret or the time-bounded secret path generated for the environment.

For the current endpoint layout and request examples, take a look at the [usage of osctrld](/usage/osctrld/).
