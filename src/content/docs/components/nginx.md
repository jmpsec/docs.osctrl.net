---
title: nginx
description: TLS termination and load balancing in front of the osctrl services.
sidebar:
  order: 5
---

<figure class="component-hero">
  <img src="/img/nginx.png" alt="nginx" />
</figure>

The nginx component performs the task of TLS termination for the **osctrl** services. If your network has many nodes enrolled in **osctrl**, most likely your TLS service ([osctrl-tls](/components/osctrl-tls/)) will receive a large number of requests per second, and on top of TLS termination, this component can then also act as load balancer.
