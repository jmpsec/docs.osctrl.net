---
title: "santa"
description: "osctrl-cli santa: export and import Santa rules."
sidebar:
  order: 15
---

`osctrl-cli santa` exports and imports the rules of the [Santa sync server](/usage/santa/), so a rule set can be copied between environments or kept in git. It works in `--api` and `--db` mode and needs `santaEnabled: true` (see [Configuration](/configuration/)). In `--db` mode the first `santa` command creates the Santa tables if they do not exist yet.

```properties
$ ./osctrl-cli santa -h
NAME:
   osctrl-cli santa - Commands for the Santa sync server

USAGE:
   osctrl-cli santa [command [command options]]

COMMANDS:
   rules-export  Export the Santa rules of an environment as JSON
   rules-import  Import Santa rules from a JSON export; nothing is imported if a rule is conflicting or invalid

OPTIONS:
   --help, -h  show help

GLOBAL OPTIONS:
   --db, -d                           Connect to local osctrl DB using YAML config file (required by db-only commands: settings add, settings delete). DB-mode environment edits do not invalidate the shared Redis environment cache: osctrl-api/osctrl-tls may serve the old environment for up to 5 minutes [$DB_CONFIG]
   --api, -a                          Connect to remote osctrl using JSON config file (the default; every command except settings add/delete also works with --db; ignored when --db is set) [$API_CONFIG]
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
   --silent, -s                       Silent mode: suppress output and skip confirmation prompts
   --version, -v                      Print version information
```

## Export rules

Writes the portable fields of every rule (`identifier`, `rule_type`, `policy`, `cel_expr`, `custom_msg`, `custom_url`, `scope`, `scope_value`), sorted by type, identifier and scope so the file diffs cleanly.

```properties
$ ./osctrl-cli santa rules-export -h
NAME:
   osctrl-cli santa rules-export - Export the Santa rules of an environment as JSON

USAGE:
   osctrl-cli santa rules-export [options]

OPTIONS:
   --environment string, -e string  Environment name or UUID
   --file string, -f string         File to write (default: standard output)
   --help, -h                       show help

GLOBAL OPTIONS:
   --db, -d                           Connect to local osctrl DB using YAML config file (required by db-only commands: settings add, settings delete). DB-mode environment edits do not invalidate the shared Redis environment cache: osctrl-api/osctrl-tls may serve the old environment for up to 5 minutes [$DB_CONFIG]
   --api, -a                          Connect to remote osctrl using JSON config file (the default; every command except settings add/delete also works with --db; ignored when --db is set) [$API_CONFIG]
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
   --silent, -s                       Silent mode: suppress output and skip confirmation prompts
   --version, -v                      Print version information
```

## Import rules

Each rule is classified as new, duplicate (skipped), conflict (same type, identifier and scope with a different policy, message, URL or CEL expression) or invalid. New rules are stored in one transaction only if no rule is conflicting or invalid; existing rules are never changed, and re-importing an export is a no-op. The command exits non-zero when a rule is conflicting or invalid, dry run included, so `--dry-run` can gate a review pipeline.

```properties
$ ./osctrl-cli santa rules-import -h
NAME:
   osctrl-cli santa rules-import - Import Santa rules from a JSON export; nothing is imported if a rule is conflicting or invalid

USAGE:
   osctrl-cli santa rules-import [options]

OPTIONS:
   --environment string, -e string  Environment name or UUID
   --file string, -f string         JSON file with the rules
   --dry-run                        Report new, duplicate, conflicting and invalid rules without importing
   --help, -h                       show help

GLOBAL OPTIONS:
   --db, -d                           Connect to local osctrl DB using YAML config file (required by db-only commands: settings add, settings delete). DB-mode environment edits do not invalidate the shared Redis environment cache: osctrl-api/osctrl-tls may serve the old environment for up to 5 minutes [$DB_CONFIG]
   --api, -a                          Connect to remote osctrl using JSON config file (the default; every command except settings add/delete also works with --db; ignored when --db is set) [$API_CONFIG]
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
   --silent, -s                       Silent mode: suppress output and skip confirmation prompts
   --version, -v                      Print version information
```

Example:

```bash
osctrl-cli santa rules-export -e prod -f rules.json
osctrl-cli santa rules-import -e staging -f rules.json --dry-run
osctrl-cli santa rules-import -e staging -f rules.json
```

When `require_second_admin` is on, an import that adds an environment-wide allow rule is held for a second admin (see [Second admin approval](/usage/santa/#second-admin-approval)).
