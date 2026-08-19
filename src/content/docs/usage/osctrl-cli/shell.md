---
title: "shell"
description: "osctrl-cli shell: a persistent, Metasploit-style interactive session for working with osctrl."
sidebar:
  order: 10
---

`osctrl-cli shell` opens a persistent, Metasploit-style interactive session
that keeps the backend connection open and exposes the same surface as the
CLI subcommands through a context-based REPL. Instead of re-invoking the
binary per action, you enter a module (`use nodes`, `use queries`, ...), run
commands against it (`list`, `show <id>`, `run <sql>`), and switch context as
needed. It is available both in `--db` mode (direct database access) and
`--api` mode (remote REST API).

Execute `./osctrl-cli shell -h` to show the help for the shell command:

```properties
$ ./osctrl-cli shell -h
NAME:
   osctrl-cli shell - Interactive shell session (Metasploit-style) — access nodes, queries, carves, ...

USAGE:
   osctrl-cli shell [options]

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
   --help, -h                         show help
```

The shell accepts the aliases `console`, `interactive` and `repl`, so all of
the following are equivalent:

```shell
$ ./osctrl-cli shell --db -D db.yml
$ ./osctrl-cli console --api -A osctrl-api.json
$ ./osctrl-cli interactive --db -D db.yml
$ ./osctrl-cli repl --api -A osctrl-api.json
```

You must enable `--db` or `--api` (with a reachable backend) to use the shell;
without one it exits with an error.

## Sessions, contexts and the prompt

Once the backend connection is established the shell prints a banner and drops
you at the top-level prompt. The prompt reflects the active environment and,
when you are inside a module, the module context:

```text
🛡️  osctrl-cli interactive shell  v0.5.6
📡  db mode
💡  Type 'help' or '?' for help · 'exit' to quit

osctrl>               top level, no environment set
osctrl[dev]>          top level, active env is "dev"
osctrl (nodes:dev)>   inside the "nodes" module, active env "dev"
```

The active environment is shared across every module, so `use queries` after
`use nodes` keeps the same environment you selected. Set it once with
`set env <name>` and every list/show/run acts against that environment until
you change it.

## Global commands

These are available from the top level and from inside any module:

| Command | Description |
|---|---|
| `help` (or `?`) | Show help for the current context. At the top level it lists every module; inside a module it lists that module's commands. |
| `use <module>` | Enter a module context (`nodes`, `queries`, `carves`, `environments`, `tags`, `users`, `audit`, `settings`). |
| `back` | Leave the current module context and return to the top level. |
| `set env <name>` | Set the active environment all modules operate against. |
| `set <option> <value>` | Set a run option used by `run` (see [Run options](#run-options)). |
| `show` / `show environments` / `show env` | List all TLS environments. |
| `stats` | Print dashboard statistics across environments (totals, active/inactive nodes, platforms, per-environment queries and carves). |
| `exit` (or `quit`) | Exit the shell. Ctrl-D also exits; Ctrl-C aborts the current line and stays in the shell. |

The shell provides tab completion for commands, module names, environment
names, and the primary resource key of the active module (node UUIDs/hostnames,
query names), refreshed as you run `list`/`show`.

## Modules

Type `help` at the top level to see the available modules:

```text
📚 osctrl-cli interactive shell

Modules (use <module>):
  🖥️  nodes         enrolled osquery nodes
  🔍  queries       on-demand distributed queries
  📦  carves        file carves
  🌐  environments  TLS environments
  🏷️  tags          node tags
  👤  users         users and permissions
  📜  audit         audit logs
  🛠️  settings      configuration values (db mode)

Global commands:
  help                       show help for the current context
  use <module>               enter a module context
  back                       leave the current module context
  set env <name>  |  <option> <value>  set active env or a run option
  show environments | env    list environments
  stats                      dashboard statistics across environments
  exit                       exit the shell
```

### nodes

Commands for enrolled osquery nodes in the active environment.

| Command | Arguments | Description |
|---|---|---|
| `list` (or `ls`) | `[active\|inactive\|all]` | List nodes (default: `all`). |
| `search` | `<query>` | Search nodes by hostname/uuid/ip/localname/username. |
| `show` | `<uuid\|hostname>` | Show full node detail. |
| `delete` (or `rm`) | `<uuid>` | Delete (archive) a node, with confirmation. |
| `tag` | `<uuid> <tag>` | Apply a tag to a node. |

### queries

Commands for on-demand distributed queries in the active environment.

| Command | Arguments | Description |
|---|---|---|
| `list` (or `ls`) | `[active\|completed\|expired\|deleted\|hidden\|all]` | List queries (default: `active`). |
| `show` | `<name>` | Show query detail. |
| `run` | `<sql>` | Dispatch a new query using the active [run options](#run-options). |
| `results` | `<name>` | Show the results for a query. |
| `complete` | `<name>` | Mark a query as completed. |
| `expire` | `<name>` | Expire a query. |
| `delete` (or `rm`) | `<name>` | Delete a query, with confirmation. |
| `options` | | Show the current run options. |

### carves

Commands for file carves in the active environment.

| Command | Arguments | Description |
|---|---|---|
| `list` (or `ls`) | `[active\|completed\|expired\|deleted\|all]` | List carve queries (default: `active`). |
| `files` | | List the carved files in the active environment. |
| `show` | `<name>` | Show carve query detail. |
| `run` | `<path>` | Start a carve for a file or directory using the active run options. |
| `complete` | `<name>` | Mark a carve as completed. |
| `expire` | `<name>` | Expire a carve. |
| `delete` (or `rm`) | `<name>` | Delete a carve, with confirmation. |
| `options` | | Show the current run options. |

### environments

Commands for TLS environments. `delete` is only available in `--db` mode; the
enroll/remove URL actions work in both modes.

| Command | Arguments | Description |
|---|---|---|
| `list` (or `ls`) | | List environments. |
| `show` | `<name>` | Show environment detail (hostname, type, enroll/remove expiry, ...). |
| `delete` (or `rm`) | `<name>` | Delete an environment (db mode only), with confirmation. |
| `extend-enroll` | `<name>` | Extend the enroll URL expiry. |
| `rotate-enroll` | `<name>` | Rotate the enroll URL. |
| `expire-enroll` | `<name>` | Expire the enroll URL. |
| `notexpire-enroll` | `<name>` | Mark the enroll URL as non-expiring. |
| `extend-remove` | `<name>` | Extend the remove URL expiry. |
| `rotate-remove` | `<name>` | Rotate the remove URL. |
| `expire-remove` | `<name>` | Expire the remove URL. |
| `notexpire-remove` | `<name>` | Mark the remove URL as non-expiring. |

### tags

Commands for node tags. `add`/`edit`/`delete` operate in the active
environment.

| Command | Arguments | Description |
|---|---|---|
| `list` (or `ls`) | `[env]` | List tags (active env if omitted). |
| `show` | `<name>` | Show tag detail. |
| `add` | `<name> [color=.. icon=.. desc=.. type=.. custom=..]` | Add a tag to the active env. |
| `edit` | `<name> [color=.. icon=.. desc=.. type=.. custom=..]` | Edit a tag in the active env. |
| `delete` (or `rm`) | `<name>` | Delete a tag, with confirmation. |

### users

Commands for users and permissions.

| Command | Arguments | Description |
|---|---|---|
| `list` (or `ls`) | | List users. |
| `show` | `<username>` | Show user detail. |
| `add` | `<username> password=<..> [email=.. fullname=.. admin=true service=false]` | Create a user. |
| `edit` | `<username> <password\|email\|fullname\|admin\|service>=<value>` | Edit a single user field. |
| `delete` (or `rm`) | `<username>` | Delete a user, with confirmation. |
| `permissions` (or `perms`) | `<username>` | Show a user's per-environment permissions. |

### audit

Commands for audit logs.

| Command | Arguments | Description |
|---|---|---|
| `list` (or `ls`) | | List audit log entries. |

### settings

Commands for configuration values. These operate directly against the
configuration store and are available in `--db` mode.

| Command | Arguments | Description |
|---|---|---|
| `list` (or `ls`) | | List configuration values. |
| `add` | `<service> <name> <type> <value>` | Add a setting. |
| `update` | `<service> <name> <type> <value>` | Update a setting. |
| `delete` (or `rm`) | `<service> <name>` | Delete a setting, with confirmation. |

## Run options

`set <option> <value>` configures the options that `run` (in the `queries`
and `carves` modules) uses when dispatching a new query or carve. List the
current values with `options`. All option values are comma-separated lists
except `hidden` (a boolean) and `expiration` (an integer in hours).

| Option | Default | Meaning |
|---|---|---|
| `env` | first detected env | The active environment the query/carve targets. |
| `uuids` | _empty_ | Restrict the query/carve to these node UUIDs. |
| `hosts` | _empty_ | Restrict the query/carve to these hostnames. |
| `platforms` | _empty_ | Restrict the query/carve to these platforms (`linux`, `darwin`, `windows`). |
| `tags` | _empty_ | Restrict the query/carve to nodes with these tags. |
| `hidden` | `no` | Whether the query is hidden from regular users. |
| `expiration` | `6` | Hours before the query/carve expires. |

## Example session

```text
$ ./osctrl-cli shell --db -D db.yml
🛡️  osctrl-cli interactive shell  v0.5.6
📡  db mode
💡  Type 'help' or '?' for help · 'exit' to quit

osctrl> show env
Name    UUID   Hostname     Type    Debug  Accept
────────────────────────────────────────────────────
dev     ...    osctrl.net   osquery no     yes
prod    ...    osctrl.net   osquery no     yes

osctrl[dev]> use nodes
-> context: nodes

osctrl (nodes:dev)> list active
3 active nodes in dev
Hostname   UUID   Platform   Version   Osquery   IP          LastSeen          Status
─────────────────────────────────────────────────────────────────────────────────────
host-01    ...    ubuntu     22.04    5.23.1    10.0.0.21   2026-07-08 09:00  active
...

osctrl (nodes:dev)> back
<- leaving nodes

osctrl[dev]> use queries
-> context: queries

osctrl (queries:dev)> set expiration 12
expiration = 12

osctrl (queries:dev)> run SELECT name, path FROM processes;
✅ query dispatched in dev

osctrl (queries:dev)> exit
```
