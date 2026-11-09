---
title: "Santa sync server"
description: "Using osctrl as the sync server for Santa on macOS: allow/block rules, execution events and client mode."
sidebar:
  order: 5
---

osctrl can act as the sync server for [Santa](https://github.com/northpolesec/santa), the macOS binary-authorization agent, so it can push allow/block rules and collect execution events without a separate sync server. It reuses osctrl's environments and osquery enrollment: a Mac syncs as the osquery node it already is.

Off by default. Enable it with `santaEnabled: true` (flag `--santa-enabled`, env `SERVICE_SANTA_ENABLED`) on **both** osctrl-tls and osctrl-api, then restart them. When off, no Santa tables are created, no routes are registered and `/api/v1/features` reports `santa: false`.

## How it works

* **osctrl-tls** serves the Santa sync protocol (v2) at `POST /{env}/santa/sync/{stage}/{machineid}`, with `stage` one of `preflight`, `eventupload`, `ruledownload` and `postflight`. `{env}` is the environment UUID and `{machineid}` the node UUID. Requests may be JSON or protobuf, optionally gzip or zlib compressed; responses use the request's encoding.
* **osctrl-api** manages it under `/api/v1/santa/{env}/`: `config` (GET/PUT), `token` (POST), `rules` (GET/POST, DELETE `rules/{id}`), `machines` and `events` (GET). All routes need env admin and write an audit line (`santa: <action>`).
* **Rules** are scoped to the environment, a tag, or a node; for each rule type and identifier the most specific scope wins (node over tag over environment). Rule types are BINARY, CERTIFICATE, TEAMID, SIGNINGID and CDHASH; policies are ALLOWLIST, ALLOWLIST_COMPILER, BLOCKLIST, SILENT_BLOCKLIST and CEL. A duplicate rule returns 409.
* **Rule sync** uses a digest. At preflight osctrl computes a SHA-256 of the machine's effective rules; if it differs from the last applied one, the client gets `CLEAN_RULES` and downloads the full set (500 rules per page), otherwise `NORMAL`. If rules change mid-sync, `ruledownload` returns 409 and the client picks the change up on its next sync, so it never applies a mixed rule set.
* **Machine config** (MONITOR or LOCKDOWN mode, batch size, transitive rules, USB blocking) is sent at preflight. The mode applies to the whole environment.
* **Events** uploaded by clients are kept for 30 days; osctrl-api prunes them daily.

## Authentication

* Each environment has one token, sent by Santa in the `X-Osctrl-Santa-Token` header. Only its SHA-256 is stored. Generating a new one returns the plaintext once and the old token stops working immediately.
* The machine ID must be the UUID of a node already enrolled in osquery in the same environment. Unknown machines, unknown environments and wrong tokens all get 403.
* Requests are not IP rate limited. Failed authentications are audited, capped at 60 per hour per environment (drops are counted in `santa_auth_audit_dropped_total`).
* Bodies are capped at 10 MiB raw and 20 MiB decompressed (413).

:::caution
The token is shared by every Santa host in the environment, and hardware UUIDs are not secret, so any host holding the token can sync as any enrolled node in that environment. Where that matters, require client mTLS at the reverse proxy in front of osctrl-tls (Santa's `ClientAuthCertificate*` keys); osctrl does not verify client certificates itself.
:::

## Rolling out to a Mac

### osctrl side

1. Set `santaEnabled: true` on osctrl-tls and osctrl-api and restart both.
2. Enroll the Mac in osquery in the target environment.
3. Open the environment's **Santa** page, then **Settings**: set the mode to **MONITOR** to start, click **Rotate token** and copy it (it is shown only once), and copy the `SyncBaseURL` shown under client setup.

### Mac side, through MDM

1. Install Santa from [North Pole Security's releases](https://github.com/northpolesec/santa/releases).
2. Deploy the profiles that pre-approve Santa: a system extension policy for its Endpoint Security extension, a PPPC profile granting Full Disk Access to the Santa daemon and, optionally, notification settings. Take the team ID and bundle identifiers from Santa's documentation.
3. Deploy a `com.northpolesec.santa` configuration profile:

```xml
<key>SyncBaseURL</key>
<string>https://osctrl.example.com/ENVIRONMENT-UUID/santa/sync/</string>
<key>SyncExtraHeaders</key>
<dict>
    <key>X-Osctrl-Santa-Token</key>
    <string>TOKEN-FROM-THE-UI</string>
</dict>
<!-- Optional: protobuf transfers instead of JSON -->
<key>SyncEnableProtoTransfer</key>
<true/>
```

The trailing slash on `SyncBaseURL` is required. Leave `MachineIDKey` unset so the machine ID is the hardware UUID, which matches the osquery node UUID, and leave `ClientMode` unset so the mode chosen in osctrl applies. The Mac must trust the certificate served by osctrl-tls; deploy a private CA through MDM if needed.

### Verify

* On the Mac, run `santactl sync` and then `santactl status`.
* In osctrl, the Mac shows under **Santa > Machines** with its last sync time.
* Optionally enable the `santa_agent` posture check (seeded disabled), which reports whether the Santa system extension is active on each macOS node.

### Move to enforcement

1. Run in MONITOR mode, review **Events**, and use **Allow** for legitimate binaries. Allowing by TEAMID covers a whole vendor with far fewer rules than per-binary hashes.
2. Use tag- or node-scoped rules for exceptions.
3. When no new legitimate binaries show up, switch the environment to **LOCKDOWN**. Because the mode is per environment, a LOCKDOWN pilot group needs its own environment.

## Not implemented

The `commands`, `metrics` and `signalupload` stages; push notifications; file-access, network-flow and telemetry rules; bundle uploads; forwarding Santa events to alert rules or log sinks; MCP tools.
