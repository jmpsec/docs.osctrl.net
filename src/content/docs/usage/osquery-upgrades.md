---
title: "osquery upgrades"
description: "Upgrading osquery across a fleet from osctrl: version library, environment targets with staged rollout, and per-node pins."
sidebar:
  order: 7
---

osctrl can upgrade osquery on enrolled hosts. It decides the version; [osctrld](/usage/osctrld-daemon/) on each host downloads, verifies, installs and confirms it, rolling back automatically when it can. Requires `osctrld.enabled` on `osctrl-tls`, and hosts running osctrld.

## Version library

Super admins add the official package SHA-256 digests per version (`GET /api/v1/osquery/versions`, `PUT` / `DELETE /api/v1/osquery/versions/{version}`): Linux `deb`/`rpm` for amd64 and arm64, macOS `pkg` (universal) and Windows `msi` (amd64). A version used by an environment target or an unexpired pin can't be removed or have its digests changed (409). Packages are always downloaded from the official osquery server, never from a URL osctrl supplies.

## Environment target and staged rollout

Environment admins set a target with `PUT /api/v1/environments/osquery/{env}` (`GET` shows it with host counts per version and the rollout; `DELETE` clears it).

A target starts a **staged rollout**: **1%, 10%, 50%, 100%** of the environment's hosts, with each stage open two hours. Each host has a stable position, so it joins at the same stage every time. Hosts already at or above the version are left alone, and osctrld upgrades the others only while `osqueryd` is running. There is no background job: the stage is computed from the clock.

- A host that **rolls back pauses the rollout**. Resume it with `POST .../osquery/{env}/rollout/resume` (continues at the stage it paused in) or abort it with `POST .../osquery/{env}/rollout/abort` (also clears the target; hosts keep their version and pins are kept).
- Setting the same version again starts a new rollout and retries hosts that rejected it. Environment targets never expire; use a pin for a temporary override.
- Hosts waiting for their stage report `rollout_pending`; while paused, `rollout_paused`.

## Per-node pin

`PUT /api/v1/nodes/{env}/node/{uuid}/osquery` pins one node to a library version, overriding the environment target, optionally until `expires_at`. `GET` shows the pin, the effective version, the installed version, the node's rollout stage and osctrld's last upgrade report; `DELETE` removes the pin. Extending only the expiry doesn't retry a version the host rejected; re-saving the same version with the same expiry does. Pin one host first to validate a version.

## Requirements and limits

- The environment's osquery configuration must carry the **osquery version decorator**, so osctrl sees the new version at check-in; otherwise every upgrade rolls back with `version_not_reported`.
- Hosts poll `osctrld-osquery` about every 30 seconds for up to 5 minutes while confirming (about 10 requests each). These count against `rateLimits.osctrld` (300/min per client IP by default). Behind a NAT or a wrong `service.trustedProxies` many hosts share one budget, so stage large rollouts.
- **Windows** upgrades need a current osctrld, support amd64 only, and cannot be rolled back automatically. The environment page shows the number of Windows hosts. Platforms without a package (for example FreeBSD) are `unsupported_platform`.
- Rollbacks, paused rollouts and hosts behind target are audited and exported as [metrics](/components/metrics/#osctrld-metrics).
