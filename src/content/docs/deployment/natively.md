---
title: "Using provision.sh"
description: "Deploy osctrl in your own environment with the provision.sh script, which installs dependencies and configures all components."
sidebar:
  order: 3
---

If you want to deploy **osctrl** in your own environment, you can use the [provision.sh](https://github.com/jmpsec/osctrl/blob/master/deploy/provision.sh) script to install all necessary dependencies and to configure all components.

It is a good idea to take a look at the [usage](/usage/provision.sh/) of the provisioning script to understand all the options and all the parameters.

## Examples

A very common use case for provisioning **osctrl** , would run in dev mode and using self-signed certificates. This is would be the preferred choice for development purposes:

```properties
./deploy/provision.sh -m dev -s /path/to/code --nginx --postgres --redis -p all --all-hostname "dev.osctrl.net" -E
```
