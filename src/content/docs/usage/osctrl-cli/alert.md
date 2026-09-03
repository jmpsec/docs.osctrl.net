---
title: "alert"
description: "osctrl-cli alert: commands for the alerting system."
sidebar:
  order: 13
---

`osctrl-cli alert` manages the [alerting system](/components/osctrl-frontend/#alerts): rules that watch osquery traffic and node liveness, and channels that deliver matches. It only works in `--api` mode — there is no `--db` equivalent, and it requires `service.alertsEnabled: true` on `osctrl-api` (see [Configuration](/configuration/)).

```properties
$ ./osctrl-cli alert -h
NAME:
   osctrl-cli alert - Commands for the alerting system

USAGE:
   osctrl-cli alert [command [command options]]

COMMANDS:
   rules, r            List alert rules
   rule-create, rc     Create an alert rule
   rule-delete, rd     Delete an alert rule by ID
   channels, c         List alert channels
   channel-create, cc  Create an alert channel
   channel-delete, cd  Delete an alert channel by ID
   apply               Apply alert changes (hot-reload rules and channels in osctrl-tls)

OPTIONS:
   --help, -h  show help

GLOBAL OPTIONS:
   --db, -d                           Connect to local osctrl DB using YAML config file [$DB_CONFIG]
   --api, -a                          Connect to remote osctrl using JSON config file [$API_CONFIG]
   --api-file FILE, -A FILE           Load API JSON configuration from FILE (default: "osctrl-api.json") [$API_CONFIG_FILE]
   --api-url string, -U string        The URL for osctrl API to be used [$API_URL]
   --api-token string, -T string      Token to authenticate with the osctrl API [$API_TOKEN]
   --db-file FILE, -D FILE            Load DB YAML configuration from FILE [$DB_CONFIG_FILE]
   --db-host string                   Backend host to be connected to (default: "127.0.0.1") [$DB_HOST]
   --db-port int                      Backend port to be connected to (default: 5432) [$DB_PORT]
   --db-name string                   Database name to be used in the backend (default: "osctrl") [$DB_NAME]
   --db-user string                   Username to be used for the backend (default: "postgres") [$DB_USER]
   --db-pass string                   Password to be used for the backend (default: "postgres") [$DB_PASS]
   --db-max-idle-conns int            Maximum number of connections in the idle connection pool (default: 20) [$DB_MAX_IDLE_CONNS]
   --db-max-open-conns int            Maximum number of open connections to the database (default: 100) [$DB_MAX_OPEN_CONNS]
   --db-conn-max-lifetime int         Maximum amount of time a connection may be reused (default: 30) [$DB_CONN_MAX_LIFETIME]
   --insecure, -i                     Allow insecure server connections when using SSL
   --verbose, -V                      Increase output verbosity for debugging
   --output-format string, -o string  Format to be used for data output (default: "pretty") [$OUTPUT_FORMAT]
   --silent, -s                       Silent mode
   --version, -v                      Print version information
```

## List rules

```properties
$ ./osctrl-cli alert rules -h
NAME:
   osctrl-cli alert rules - List alert rules

USAGE:
   osctrl-cli alert rules [options]

OPTIONS:
   --help, -h  show help
```

## Create rule

```properties
$ ./osctrl-cli alert rule-create -h
NAME:
   osctrl-cli alert rule-create - Create an alert rule

USAGE:
   osctrl-cli alert rule-create [options]

OPTIONS:
   --name string, -n string         Rule name
   --source string, -s string       Source: result_log | status_log | query_log | node_inactive | node_recovered
   --match-type string, -m string   Match type: substring | regex (default: "substring")
   --match-field string             Field to match (empty = any field)
   --match-value string, -v string  Pattern to match
   --cooldown int                   Cooldown minutes between repeat alerts (default: 0)
   --channels string                Comma-separated channel IDs
   --enabled                        Create the rule enabled
   --help, -h                       show help
```

`--match-field` empty tries every column of the row (or every status/query-result field). `--cooldown 0` uses the system default of 15 minutes. `node_inactive` and `node_recovered` rules ignore `--match-type`/`--match-field`/`--match-value` — they fire on the node's active/inactive state transition itself, not on log content.

## Delete rule

```properties
$ ./osctrl-cli alert rule-delete -h
NAME:
   osctrl-cli alert rule-delete - Delete an alert rule by ID

USAGE:
   osctrl-cli alert rule-delete [options]

OPTIONS:
   --id uint   Rule ID (default: 0)
   --help, -h  show help
```

## List channels

```properties
$ ./osctrl-cli alert channels -h
NAME:
   osctrl-cli alert channels - List alert channels

USAGE:
   osctrl-cli alert channels [options]

OPTIONS:
   --help, -h  show help
```

## Create channel

```properties
$ ./osctrl-cli alert channel-create -h
NAME:
   osctrl-cli alert channel-create - Create an alert channel

USAGE:
   osctrl-cli alert channel-create [options]

OPTIONS:
   --name string, -n string  Channel name
   --type string, -t string  Channel type: webhook | email
   --config string           Channel config as JSON
   --enabled                 Create the channel enabled
   --help, -h                show help
```

`--config` takes the channel's field values as a JSON object. For `webhook`: `{"url": "...", "secret": "...", "timeoutSeconds": 10, "insecureSkipVerify": false, "allowPrivateTargets": false}` (only `url` is required). For `email`: `{"host": "...", "port": 587, "username": "...", "password": "...", "from": "...", "to": "a@example.com,b@example.com", "starttls": true}` (`host`, `from` and `to` are required).

```bash
./osctrl-cli alert channel-create --api -n oncall-webhook -t webhook \
  --config '{"url":"https://hooks.example.com/osctrl","secret":"replace-me"}' --enabled
```

## Delete channel

```properties
$ ./osctrl-cli alert channel-delete -h
NAME:
   osctrl-cli alert channel-delete - Delete an alert channel by ID

USAGE:
   osctrl-cli alert channel-delete [options]

OPTIONS:
   --id uint   Channel ID (default: 0)
   --help, -h  show help
```

## Apply

```properties
$ ./osctrl-cli alert apply -h
NAME:
   osctrl-cli alert apply - Apply alert changes (hot-reload rules and channels in osctrl-tls)

USAGE:
   osctrl-cli alert apply [options]

OPTIONS:
   --help, -h  show help
```

Queues a hot-reload of the current rules and channels into `osctrl-tls`, without restarting it — run this after creating or deleting rules/channels so the running service picks them up.
