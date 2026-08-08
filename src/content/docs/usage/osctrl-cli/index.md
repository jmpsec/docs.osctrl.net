---
title: "osctrl-cli"
description: "osctrl-cli is the CLI for osctrl."
sidebar:
  order: 3
---

`osctrl-cli` is the CLI for **osctrl**. Its purpose is to execute actions in **osctrl** without having to access directly the backend or use the admin interface. It can be very handy to automate actions in scripts or similar.

Execute `./osctrl-cli help` to show the main help of the program:

```properties
$ ./osctrl-cli help
NAME:
   osctrl-cli - CLI for osctrl

USAGE:
   osctrl-cli [global options] [command [command options]]

DESCRIPTION:
   CLI for osctrl, a fast and efficient osquery management

COMMANDS:
   shell, console, interactive, repl  Interactive shell session (Metasploit-style) — access nodes, queries, carves, ...
   audit-logs                         Get all audit logs for actions performed in osctrl
   carve                              Commands for file carves
   check-api                          Checks API token
   check-db                           Checks DB connection
   environment, env                   Commands for TLS environment
   login                              Login into API and generate JSON config file with token
   node                               Commands for nodes
   query                              Commands for queries
   settings                           Commands for settings
   tag                                Commands for tags
   user                               Commands for users
   help, h                            Shows a list of commands or help for one command

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
   --help, -h                         show help
```

Each command has its own help and it is displayed appending `-h` to them. Each subcommand has its own help as well.
