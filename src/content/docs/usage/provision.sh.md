---
title: "provision.sh"
description: "provision.sh is the provisioning script for osctrl in Ubuntu 20.04."
# Astro strips the dot when deriving a slug from the filename, which would
# give /usage/provisionsh/. Pinned to keep URL parity with the Hugo site.
slug: usage/provision.sh
sidebar:
  order: 4
---

[provision.sh](https://github.com/jmpsec/osctrl/blob/master/deploy/provision.sh) is the provisioning script for **osctrl** in Ubuntu 20.04. It uses several functions from [lib.sh](https://github.com/jmpsec/osctrl/blob/master/deploy/lib.sh).

Its purpose is to install all necessary components to deploy **osctrl** in an Ubuntu 20.04 system, although it could be easily adapted to work in CentOS 7.

:::note
`provision.sh` also detects and provisions Ubuntu 22.04, 24.04 and 26.04, installing the matching PostgreSQL release for each: 22.04 → PostgreSQL 14, 24.04 → PostgreSQL 16, 26.04 → PostgreSQL 18. No parameters changed for this — it's automatic based on the running OS version.
:::

Execute `./deploy/provision.sh [-h|--help]` to show the usage of the script:

```properties
$ ./deploy/provision.sh -h

Usage: ./deploy/provision.sh [-h|--help] [PARAMETER [ARGUMENT]] [PARAMETER [ARGUMENT]] ...

Parameters:
  -h, --help 		Shows this help message and exit.
  -m MODE, --mode MODE 	Mode of operation. Default value is dev
  -t TYPE, --type TYPE 	Type of certificate to use. Default value is self
  -p PART, --part PART 	Part of the service. Default is all

Arguments for MODE:
  dev 		Provision will run in development mode. Certificate will be self-signed.
  prod 		Provision will run in production mode.

Arguments for TYPE:
  self 		Provision will use a self-signed TLS certificate that will be generated.
  own 		Provision will use the TLS certificate provided by the user.
  certbot 	Provision will generate a TLS certificate using letsencrypt/certbot. More info here: https://certbot.eff.org/

Arguments for PART:
  admin 	Provision will deploy only the admin interface.
  tls 		Provision will deploy only the TLS endpoint.
  api 		Provision will deploy only the API endpoint.
  all 		Provision will deploy both the admin and the TLS endpoint.

Optional Parameters:
  --public-tls-port PORT 	Port for the TLS endpoint service. Default is 443
  --public-admin-port PORT 	Port for the admin service. Default is 8443
  --public-api-port PORT 	Port for the API service. Default is 8444
  --private-tls-port PORT 	Port for the TLS endpoint service. Default is 9000
  --private-admin-port PORT 	Port for the admin service. Default is 9001
  --private-api-port PORT 	Port for the API service. Default is 9002
  --all-hostname HOSTNAME 	Hostname for all the services. Default is 127.0.0.1
  --tls-hostname HOSTNAME 	Hostname for the TLS endpoint service. Default is 127.0.0.1
  --admin-hostname HOSTNAME 	Hostname for the admin service. Default is 127.0.0.1
  --api-hostname HOSTNAME 	Hostname for the API service. Default is 127.0.0.1
  -X PASS     --password 	Force the admin password for the admin interface. Default is random
  -c PATH     --certfile PATH 	Path to supplied TLS server PEM certificate(s) bundle
  -d DOMAIN   --domain DOMAIN 	Domain for the TLS certificate to be generated using letsencrypt
  -e EMAIL    --email EMAIL 	Domain for the TLS certificate to be generated using letsencrypt
  -s PATH     --source PATH 	Path to code. Default is ~/osctrl
  -S PATH     --dest PATH 	Path to binaries. Default is /opt/osctrl
  -n          --nginx 		Install and configure nginx as TLS termination
  -P          --postgres 	Install and configure PostgreSQL as backend
  -R          --redis 		Install and configure Redis as cache
  -E          --enroll  	Enroll the serve into itself using osquery. Default is disabled
  -N NAME     --env NAME 	Initial environment name to be created. Default is the mode (dev or prod)
  -U          --upgrade 	Keep osctrl upgraded with the latest code from Github

Examples:
  Provision service in development mode, code is in /code/osctrl and all components (admin, tls, api):
	./deploy/provision.sh -m dev -s /code/osctrl -p all
  Provision service in production mode using my own certificate and only with TLS endpoint:
	./deploy/provision.sh -m prod -t own -k /etc/certs/my.key -c /etc/certs/cert.crt -p tls
  Upgrade service with the latest code from Github. Does not create services nor certificates:
	./deploy/provision.sh -U -s /code/osctrl -S /srv/osctrl
```
