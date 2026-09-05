---
title: nginx
description: TLS termination and load balancing in front of the osctrl services.
sidebar:
  order: 6
---

<figure class="component-hero diagram-hero">
  <img class="only-light" src="/img/components-nginx-light.svg" alt="nginx terminating TLS in front of osctrl-tls and the frontend" />
  <img class="only-dark" src="/img/components-nginx-dark.svg" alt="" aria-hidden="true" />
</figure>

The nginx component performs the task of TLS termination for the **osctrl** services. If your network has many nodes enrolled in **osctrl**, most likely your TLS service ([osctrl-tls](/components/osctrl-tls/)) will receive a large number of requests per second, and on top of TLS termination, this component can then also act as load balancer.
