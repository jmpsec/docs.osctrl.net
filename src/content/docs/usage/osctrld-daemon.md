---
title: "osctrld daemon"
description: "The osctrld daemon: syncing flags, certificates and extensions, rollbacks, health reports and native install."
sidebar:
  order: 10
---

`osctrld` is the host-side daemon of **osctrl** ([jmpsec/osctrld](https://github.com/jmpsec/osctrld)). It keeps a host's osquery aligned with its osctrl environment by talking to the [osctrld endpoints](/usage/osctrld/) of `osctrl-tls`: it manages flags, the enrollment secret, the server certificate, [extensions](/usage/osquery-extensions/) and [osquery upgrades](/usage/osquery-upgrades/), and restarts osquery through the OS service manager when a managed artifact changes. It runs on Linux, macOS and Windows and must run as root (Administrator on Windows).

## Commands

| Command | Purpose |
| --- | --- |
| `enroll`, `remove` | Print the osctrl enroll/remove script for review |
| `flags`, `cert` | Fetch the flagfile or certificate and write it locally |
| `verify` | Check local flags, certificate and secret against osctrl |
| `service` | Run as a daemon, syncing every `interval` minutes with jitter |
| `install`, `uninstall` | Native enrollment: install osquery if needed, write secret, flags and certificate, start the service (or reverse it, leaving osquery installed) |
| `check-config`, `default-config` | Validate a configuration file / print a template |

## Configuration

```yaml
osctrld:
  osctrlSecret: "thisisthesecret"
  osquerySecretFile: "/path/to/osquery.secret"
  osqueryFlagFile: "/path/to/osquery.flags"
  osqueryCertFile: "/path/to/osquery.crt"
  environment: "osctrl_environment_UUID"   # the UUID, not the name
  baseurl: "https://osctrl.url"
  insecure: false
  interval: 60                             # minutes between syncs
  extensionsDir: "/path/to/extensions/"    # configuration file only
```

Paths default to the osquery directory of the OS; set them only with absolute paths (a relative path resolves against `/` under systemd and launchd and logs a warning). Other fields: `osquerySHA256`, `osqueryPackage` and `allowUnverified` for `install`. JSON files work too.

**Precedence:** when a configuration file is used (`--config` or `OSCTRL_CONFIG`) it replaces the entire effective configuration: every other flag and environment variable is ignored, even for fields the file leaves unset, and osctrld logs one warning naming the ignored ones (never their values). Without a file, flags win over environment variables. Set `interval` in the file, since `--interval` is ignored with `--config`. Unknown keys are ignored with a warning naming them. Keep the file `0600`: it holds the enrollment secret. The packages (deb, rpm, apk) install the binary, the systemd unit and the configuration; on macOS the launchd plist runs `/usr/local/bin/osctrld`.

## Native install

`osctrld install` does what the quick-add script does without a shell. The osquery package is verified before installation: the SHA-256 comes from osctrl when it has one, else from `--osquery-sha256`; with neither, install is refused unless `--allow-unverified` is passed. `--osquery-package` points to a mirror for air-gapped installs, and only then does `--insecure` apply to the package download.

```bash
sudo --preserve-env=OSCTRL_SECRET osctrld --environment <environment-uuid> --osctrl-url https://osctrl.example.com \
  --osquery-sha256 <sha256-of-the-package> install
```

## Hardening

- **HTTPS:** use an `https` `baseurl`, since requests carry the enrollment secret; `http://` still works but logs a warning (also shown by `check-config`). `baseurl` can't carry a query or fragment. Credentials in it are sent as Basic auth (for a reverse proxy) but are discouraged.
- **Redirects:** requests to osctrl follow redirects only within the same scheme, host and port, or from http to https on the same host; package downloads may follow https redirects to another host (a CDN). Nothing may go from https to http. `HTTPS_PROXY`, `HTTP_PROXY` and `NO_PROXY` are honored.
- **Secret handling:** don't pass `--secret`, which is visible in the process list and shell history. Use a root-only configuration file or the `OSCTRL_SECRET` environment variable (with `sudo --preserve-env=OSCTRL_SECRET`).
- **Files:** secret `0600`, flags and certificate `0700`; on Windows the secret gets a protected ACL for SYSTEM and Administrators only. Every file is written to a temp file and renamed, and a symlink in its place is replaced, never written through. osctrld warns when it runs as root with a config file others can write.
- **Windows:** the extensions directory must be owned by SYSTEM, Administrators or TrustedInstaller with no other account able to write, create or delete in it; the default `C:\Program Files\osquery` qualifies, a folder under `C:\ProgramData` does not.
- **Uninstall:** `uninstall` also deletes osctrld's own state files next to the flagfile and the upgrade directory (its snapshot holds the secret). Installed extensions and their `extensions.load` lines stay and must be removed by hand.

## Flags and rollback

Flag requests carry the host UUID, so osctrl can serve a [per-node flagfile](/usage/node-overrides/), and the outcome of the last flags change. osquery exits on an unknown flag, so after a restart caused by new flags osctrld waits up to 60 seconds for `osqueryd` to stay running (three consecutive checks). If it doesn't, it rolls back in steps (writing, restarting once and checking each): to the environment flags, then to the last known good flags, then it stops and leaves `osqueryd` down until osctrl serves something new. Failed content is remembered by SHA-256 and not applied again while osctrl serves it, and is reported as `rejected`. State lives next to the flagfile in `<osqueryFlagFile>.osctrld-state.json` and `<osqueryFlagFile>.osctrld-lastgood`.

## Extensions

Synced after flags and certificates, with their own restart and check, so a bad extension never rolls back good flags.

- **Verified:** each binary is downloaded from the configured osctrl server (never from a URL in the list), checked for size and SHA-256 and renamed into `extensionsDir` as `<name>.ext` (`<name>.exe` on Windows).
- **Root-owned:** files are `0755` root-owned (Administrators/SYSTEM-only on Windows), and symlinks are never written or removed through.
- **Autoload file:** osctrld manages only its own lines of osquery's `extensions.load` (or the file named by `--extensions_autoload`) and keeps lines and files an operator added. It only writes a `.load` file directly in osquery's base directory.
- **Removal:** extensions no longer assigned are removed and reported `removed`; if removal fails they stay and are reported `kept`, so osctrl can show a revoked extension is still on the host.
- **Rollback:** after the restart osctrld waits for `osqueryd` and for a process running each new extension. If `osqueryd` doesn't stay up the previous extensions are restored; if only some don't run, just those are removed. Failed content is `rejected` until osctrl serves a different SHA-256.
- **Windows:** a running executable can't be replaced, so it is renamed to `<file>.prev` and deleted after osquery restarts.

## Watcher and status report

Between syncs osctrld checks for `osqueryd` every 60 seconds: down after 2 consecutive misses, up after 3 consecutive hits. It never restarts a dead `osqueryd` itself, but a sync still runs, so a fix served by osctrl is applied. It reports to `POST /{env}/osctrld-status` (whether `osqueryd` runs, since when it is down, the last flags/extensions rollback and the last upgrade result):

- at the end of every sync;
- immediately when the watcher's state changes;
- about every 5 minutes while `osqueryd` is down, until it has been up for 10 minutes.

Each report has a 15-second timeout and a failure never stops the service. A 404/405 pauses reports until the next sync; a 429 honors `Retry-After` (up to an hour). osctrl turns these reports into the node page banner, [metrics](/components/metrics/#osctrld-metrics) and audit entries.

## osquery upgrades

osctrld upgrades osquery when [osctrl serves a target version](/usage/osquery-upgrades/). At most one attempt per sync, at its end, only when `osqueryd` is running and nothing else changed in that sync. Packages always come from the official osquery package server, are verified against the SHA-256 in osctrl's library, and the flagfile, secret, certificate and autoload file are snapshotted and restored over the package. osctrl must see the new version within 5 minutes of the restart, otherwise osctrld reinstalls the previous package, restores the snapshot and reports the target `rejected`.

**Requirement:** the environment's osquery configuration must carry the osquery version decorator, otherwise confirmation never succeeds and every upgrade rolls back with `version_not_reported`. Pin one host first.

The result is reported as `ok`, `rejected` or `skipped` with a reason (`no_target`, `ahead_of_target`, `no_digest_for_target`, `no_digest_for_installed`, `unsupported_platform`, `rollout_pending`, `rollout_paused`, `previous_unavailable`, `target_unavailable`, `digest_mismatch`, `version_not_reported`, `osqueryd_not_running`, `install_failed`, `config_changed`, `rolled_back`, `rollback_unavailable`). A rejected version isn't retried until osctrl serves a new request id, version or digest; a failed install is skipped for 6 hours.

**Windows:** the MSI is checked against its SHA-256 and Authenticode signature and installed with `msiexec` (killed after 10 minutes; exit codes 0, 3010 and 1641 count as success and the installed version decides). Only amd64 is supported; ARM64 hosts skip with `target_unavailable`. There is no MSI downgrade, so a failed upgrade is **not rolled back**: osctrld restores the configuration, starts `osqueryd` and reports `rejected` with `rollback_unavailable`; the host needs manual repair. After every install and on every sync osctrld repairs the `osqueryd` service's automatic start and `--flagfile=` argument if the MSI reset them.

On macOS the upgrade copies the packaged launchd plist into `/Library/LaunchDaemons`, overwriting a customised one.
