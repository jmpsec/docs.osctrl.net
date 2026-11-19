---
title: "Santa sync server"
description: "Using osctrl as the sync server for Santa on macOS: rules, client modes, events, approval requests, voting, risk checks, reports and rollout."
sidebar:
  order: 5
---

osctrl can act as the sync server for [Santa](https://github.com/northpolesec/santa), the macOS binary-authorization agent, so it can push allow/block rules and collect execution events without a separate sync server. It reuses osctrl's environments and osquery enrollment: a Mac syncs as the osquery node it already is.

Off by default. Enable it with `santaEnabled: true` (flag `--santa-enabled`, env `SERVICE_SANTA_ENABLED`) on **both** osctrl-tls and osctrl-api, then restart them. When off, no Santa tables are created, no routes are registered and `/api/v1/features` reports `santa: false`.

## How it works

* **osctrl-tls** serves the stable Santa sync protocol (v1, `santa.sync.v1`) at `POST /{env}/santa/sync/{stage}/{machineid}`, with `stage` one of `preflight`, `eventupload`, `ruledownload` and `postflight`. `{env}` is the environment UUID and `{machineid}` the node UUID. Requests may be JSON or protobuf, optionally gzip or zlib compressed; responses use the request's encoding.
* **osctrl-api** manages it under `/api/v1/santa/{env}/` (`config`, `token`, `rules`, `modes`, `machines`, `events`, `binaries`, `reports`, `requests`, `votes`, `file-access-events`). Management routes need env admin and write an audit line (`santa: <action>`). The environment's **Santa** page in the frontend has Rules, Events, Machines, Requests, Reports, File access, Changes and Settings tabs.
* **Rules** are scoped to the environment, a tag, or a node; for each rule type and identifier the most specific scope wins (node over tag over environment). Rule types are BINARY, CERTIFICATE, TEAMID, SIGNINGID and CDHASH; policies are ALLOWLIST, ALLOWLIST_COMPILER, BLOCKLIST, SILENT_BLOCKLIST and CEL. A duplicate rule returns 409. Rules can be edited in place (`PUT rules/{id}`), with the same validation as create; the audit line records the rule before and after.
* **Rule sync** uses a digest. At preflight osctrl computes a SHA-256 of the machine's effective rules; if it differs from the last applied one, the client gets `CLEAN_RULES` and downloads the full set (500 rules per page), otherwise `NORMAL`. If rules change mid-sync, `ruledownload` returns 409 and the client picks the change up on its next sync, so it never applies a mixed rule set.
* **Events** uploaded by clients are kept for 30 days; osctrl-api prunes them daily. `GET events` takes optional `machine_id`, `decision` (a name or a prefix, so `BLOCK` matches every block) and `sha256` filters. `GET machines/{machineid}/rules` returns the rules and client mode one node gets after scope resolution.

## Client mode

The environment sets the default mode, MONITOR or LOCKDOWN. A tag or a single node can be given a different one under **Settings > Mode overrides** (`PUT /api/v1/santa/{env}/modes` with `scope`, `scope_value` and `client_mode`). A node override wins over a tag override, which wins over the environment; if a node's tags disagree, LOCKDOWN wins. A change applies at the client's next sync. The **Machines** tab shows the mode each machine got at its last preflight and where it came from (`env`, `tag` or `node`).

**Temporary MONITOR mode.** A machine can leave LOCKDOWN for 1 to 24 hours and go back on its own. An env admin grants it as a node override with `hours`, or a user asks for it from the request page and it goes through the approval queue. Expiry is server-side: osctrl-api removes expired overrides every minute and audits each one. The machine switches at its next preflight and switches back at its first preflight after the expiry.

**Sync interval.** Rule and setting changes reach a machine at its next full sync. **Settings > Sync interval** (`full_sync_interval_seconds`) is `0` (Santa's own 600 seconds) or 60 to 86400 seconds. Each full sync is three or four requests per machine, so 1,000 Macs at 60 seconds is 50 to 67 requests a second. There is no push: Santa's push and NATS are tied to North Pole Security, so a shorter interval is the only lever.

## Settings

All of these live in the Santa **Settings** tab (and the config API) and are sent at every preflight.

* **Removable media** (`removable_media_policy`): `ALLOW` (default), `BLOCK`, or `REMOUNT` with options (`rdonly`, `noexec`, `nosuid`, `nodev`, `nobrowse`, `noowners`, `async`, `-j`). An optional **Encrypted removable media** policy takes the same values. Older clients get the deprecated `block_usb_mount`/`remount_usb_mode`, derived from the main policy; configs with `block_usb_mount: true` are migrated to `BLOCK`.
* **Bundles** (`enable_bundles`): when Santa blocks a binary inside an app bundle, osctrl asks for every binary of the bundle once, and they appear in **Events** as `BUNDLE_BINARY` events.
* **Upload all execution events** (`enable_all_event_upload`) and **Skip would-be blocks** (`disable_unknown_event_upload`) widen or narrow what clients upload.
* **Allowed and blocked path regexes** (`allowed_path_regex`, `blocked_path_regex`): decide executions that match no rule, by path. Blocked is checked first. Use the subset of ICU that Go's RE2 accepts (no lookaround or backreferences), at most 1024 characters. A writable path in the allowed regex lets anyone run anything from it.
* **Block dialog URL and button label** (`event_detail_url`, `event_detail_text`): an `https` URL, with Santa placeholders such as `%file_sha%` and `%machine_id%`. It overrides the approval request link.
* **File access rules override** (`override_file_access_action`): unset (default, nothing sent), `NONE`, `AUDIT_ONLY` or `DISABLE`. Choosing unset again does not undo an override a client already received; choose `NONE`.

## Events in log sinks and alert rules

After an upload is stored, osctrl-tls forwards its events to the [log sinks](/components/osctrl-frontend/#log-sinks) and [alert rules](/components/osctrl-frontend/#alerts), with no Santa-specific setting:

* **Log sinks** get each upload as one JSON array under the category `santa`. A sink with no category filter receives it; a filtered sink needs `santa` in its categories. The DB sink ignores it, since the events are already stored. A failing sink is logged and does not fail the upload.
* **Alert rules** with the source `santa_event` match on `decision`, `file_sha256`, `file_path`, `file_name`, `executing_user`, `team_id`, `signing_id`, `cd_hash` or `bundle_id`. For example, field `decision` with pattern `BLOCK` alerts on every blocked execution. A rule alerts once per node and file hash within its cooldown. These rules need the same permission as log-content rules (superuser or full query tier), and osctrl-tls needs `alertsEnabled`.
* The osquery log endpoint accepts only `result` and `status` logs, so an osquery node cannot forge Santa events (or query and carve logs) into the sinks.

## Binary details and retrieval

The file name in **Events** or the approval queue opens a detail view for that hash (`GET binaries/{sha256}`): name, path, team ID, signing ID, CDHash, bundle and signing chain from its newest event; first and last seen; event, block and machine counts; and every rule in the environment that names the file, its team ID, signing ID, CDHash or leaf certificate. Counts cover the 30-day event window. Everything there is written by Santa clients, so it is context for a decision, not proof of who signed the file.

**Retrieve file** on an event (`POST events/{id}/carve`) starts an osquery carve of the event's path on that machine. Carve access, sensitive paths, node scope and carve auditing apply as for any carve, and the event row links to the carve. The machine must be an osquery node of the same environment (409 otherwise).

## Reports

The **Reports** tab (`GET reports?days=N`, 1 to 30, default 14) answers when machines can move to LOCKDOWN:

* **Ready for lockdown:** every MONITOR machine with its count of `ALLOW_UNKNOWN` executions, the ones LOCKDOWN would have blocked. A machine with none and a sync in the window is ready.
* **Top blocked:** binaries and team IDs ranked by `BLOCK_*` events, with machine counts.
* **Rule coverage gaps:** the most executed binaries no rule decided, the first candidates for a rule.

## Bulk import and export

Rule sets can be moved between environments or kept in git. **Export** on the Rules tab writes the portable fields, sorted so the file diffs cleanly. **Import** (`POST rules/import`, at most 4 MiB) classifies each rule as new, duplicate (skipped), conflict (same type, identifier and scope with a different policy, message, URL or CEL) or invalid. With `?dry_run=true` only the report is returned. Otherwise new rules are stored in one transaction, but only if nothing conflicts or is invalid; existing rules are never changed. The UI always runs the dry run first. From the CLI, see [`osctrl-cli santa`](/usage/osctrl-cli/santa/).

## Approval requests

A user whose software was blocked can ask for it to be allowed from the Santa block dialog. Off by default.

**Turn it on** by setting **Approval request URL** to the osctrl UI address, for example `https://osctrl.example.com` (`approval_url`). The block dialog then shows a **Request approval** button that opens `https://osctrl.example.com/santa/request/ENVIRONMENT-UUID?sha256=%file_sha%&machine_id=%machine_id%`. Emptying the setting removes the link and turns the request routes off.

**The request.** The page asks the user to sign in and for a justification. A request is stored only when osctrl holds a `BLOCK` event for that file on that machine; until Santa uploads it the answer is 404. A pending request for the same file and machine is reused, one account can hold 20 pending requests per environment, and pending requests expire after 30 days. Any signed-in account can request, with no environment permission: a requester is usually an account with no permissions created at first SSO login by JIT provisioning. Requesters see only their own requests.

:::caution
osctrl does not check that the person signing in is the user of the Mac in the link. Anyone with an account who knows a machine UUID and a file hash blocked on it can file the request. The request records who asked.
:::

**Deciding.** Env admins decide in the **Requests** tab, one at a time or in bulk. Approving creates one `BINARY` `ALLOWLIST` rule for the hash in the scope the approver picks (`node`, `tag` or `env`), or points at an existing allow rule. A block rule for the same hash and scope returns 409. Denying needs a reason. The requester sees the status and reason on the request page.

**Notifications.** With `alertsEnabled` on osctrl-api, **Notify about new requests** sends each new request to alert channels (email or webhook) with **Approve** and **Deny** links. A link only opens the decision page; the approver still signs in as an env admin and confirms. Each link works once, for its action, for 72 hours. Choosing channels needs a superuser.

**Voting.** For software many people use, votes can allow a file without an approver. Set a **local** and a **global** threshold (`vote_local_threshold`, `vote_global_threshold`; 0 is off). Any signed-in account can vote once per file from the request page. At the local threshold every voter's machine gets a node-scoped allow rule; at the global threshold the file gets an environment-scoped one. Rules made this way have `votes` as their creator. Only files Santa blocked as unknown can be voted for, and a block rule on the file, its team ID, signing ID or CDHash stops a vote. Accounts are the unit, so choose thresholds with the number of accounts in mind.

## Risk checks

Before a vote or an approval allows a file, osctrl-api can ask reputation services about its hash. Off until a check is configured on **osctrl-api**:

| Setting | Flag / environment | Default |
| --- | --- | --- |
| VirusTotal API key | `--santa-virustotal-key` / `SANTA_VIRUSTOTAL_KEY` | off |
| Engines that must flag a file | `--santa-virustotal-threshold` / `SANTA_VIRUSTOTAL_THRESHOLD` | 3 |
| Custom webhook URL | `--santa-risk-webhook-url` / `SANTA_RISK_WEBHOOK_URL` | off |
| Webhook HMAC secret | `--santa-risk-webhook-secret` / `SANTA_RISK_WEBHOOK_SECRET` | none |
| Verdict cache | `--santa-risk-ttl-hours` / `SANTA_RISK_TTL_HOURS` | 24 |
| Timeout per check | `--santa-risk-timeout-seconds` / `SANTA_RISK_TIMEOUT_SECONDS` | 10 |

* **Secrets** (key, webhook URL and secret) come from the environment, flags or YAML only, and are never stored in `service_config`, returned or logged. Prefer environment variables.
* **Only hashes are sent.** The webhook gets `POST {"sha256": "..."}`, signed with `X-Osctrl-Signature` (HMAC-SHA256, hex) when a secret is set, and answers `{"verdict": "clean" | "flagged" | "unknown", "detail": "..."}`.
* **Votes fail closed:** a vote for a flagged file is refused (409), and so is one while a check cannot answer (503). **Approvals** of a flagged file need `override` and a reason, which is recorded and audited.
* Verdicts show in the approval queue, the decision link page and binary details, which has a **Check now** button. Failed checks are cached for 5 minutes. VirusTotal's free tier allows 4 lookups a minute, so keep the TTL long.

## IdP group tags

Tag-scoped rules and mode overrides can follow department membership. **Settings > IdP group tags** maps IdP groups to existing tags of the environment; at each preflight the machine gets the tags of its primary user's groups and loses the ones it no longer qualifies for.

* **Groups come from SSO login or [SCIM](/usage/auth-providers/#scim-provisioning).** Each OIDC or SAML login stores the user's groups from the provider's groups claim. With SCIM the IdP pushes changes without a login, and SSO logins stop changing the groups of SCIM users. **Users > user > Groups…** shows what is stored.
* **Matching** compares Santa's `primary_user` with osctrl users, ignoring case. **Match the primary user to** (`group_match`) picks the whole username (default), the username before `@` (`local_part`), or the account email (`email`). A primary user matching more than one osctrl user gets no group tags.
* Only tags the mapping added are removed; a tag set by hand stays.
* **Trust.** `primary_user` is reported by the client, so a host holding the environment token can claim any user and get that user's group tags. Prefer mapping groups to tags that tighten rules.

## Second admin approval

With **Settings > Require a second admin** (`require_second_admin`), one compromised admin cannot switch enforcement off alone. These actions are held instead of applied:

* switching the environment to MONITOR, or a new `allowed_path_regex`;
* turning `require_second_admin` off (turning it on is immediate);
* any MONITOR mode override, permanent or temporary;
* creating or changing an environment-wide rule that can allow (`ALLOWLIST`, `ALLOWLIST_COMPILER`, `CEL`), including through an import;
* rotating the sync token.

A held action answers 202 and appears in **Santa > Changes** (`GET /api/v1/changes/{env}`), where another admin of the environment approves or denies it. The requester then applies it with **Apply**, or by sending the exact same request again. A change expires 24 hours after it was requested. The request, decision and apply are each audited. With the setting on, an admin also cannot approve their own MONITOR or environment-wide approval request. Deleting rules, LOCKDOWN overrides, other settings and vote-created rules are not held.

## File access rules

Santa can restrict which processes read or write sensitive files such as browser cookies, keychains or SSH keys. The v1 sync protocol cannot deliver these rules, so deploy them with MDM (`FileAccessPolicy` or `FileAccessPolicyPlist` in the `com.northpolesec.santa` payload). osctrl stores the violations clients upload and shows them in the **File access** tab (`GET file-access-events`), pruned with the execution events after 30 days, and sends the override action set in [Settings](#settings). Start a new policy with `AuditOnly`, check the tab for legitimate access, then remove it. They are not forwarded to log sinks or alert rules.

## MCP tools

[osctrl-mcp](/usage/osctrl-mcp/) has four read tools (`list_santa_rules`, `list_santa_machines`, `list_santa_events`, `get_santa_node_rules`) and, behind the write opt-in, `create_santa_rule` and `delete_santa_rule`. They need env admin like the Santa page.

## Authentication

* Each environment has one token, sent by Santa in the `X-Osctrl-Santa-Token` header. Only its SHA-256 is stored. Generating a new one returns the plaintext once and the old token stops working immediately.
* The machine ID must be the UUID of a node already enrolled in osquery in the same environment. A malformed machine ID gets 400; unknown machines, unknown environments and wrong tokens all get 403.
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
3. Pilot enforcement on a few Macs: under **Settings > Mode overrides**, set **LOCKDOWN** for a tag or node while the environment stays in MONITOR. **Reports > Ready for lockdown** shows which machines would block nothing.
4. When no new legitimate binaries show up, switch the environment to **LOCKDOWN** and remove the pilot override.

## Not implemented

The `commands`, `metrics` and `signalupload` stages; push notifications; delivering file access rules through sync (deploy them with MDM); network-flow, telemetry signal and package rules; Santa-side mode transitions; telemetry export; one-step rules for a whole bundle.
