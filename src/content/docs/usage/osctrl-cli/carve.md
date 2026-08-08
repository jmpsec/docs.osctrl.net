---
title: "carve"
description: "osctrl-cli carve: commands for file carves."
sidebar:
  order: 2
---

```properties
$ ./osctrl-cli carve -h
NAME:
   osctrl-cli carve - Commands for file carves

USAGE:
   osctrl-cli carve [command [command options]]

COMMANDS:
   complete, c      Mark an file carve query as completed
   delete, d        Mark a file carve query as deleted
   expire, e        Mark a file carve query as expired
   run, r           Start a new carve for a file or a directory
   list, l          List file carves
   list-queries, l  List file carves queries

OPTIONS:
   --help, -h  show help
```

## Complete carve

```properties
$ ./osctrl-cli carve complete -h
NAME:
   osctrl-cli carve complete - Mark an file carve query as completed

USAGE:
   osctrl-cli carve complete [options]

OPTIONS:
   --name string, -n string  Carve name to be completed
   --env string, -e string   Environment to be used
   --help, -h                show help

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

## Delete carve

```properties
$ ./osctrl-cli carve delete -h
NAME:
   osctrl-cli carve delete - Mark a file carve query as deleted

USAGE:
   osctrl-cli carve delete [options]

OPTIONS:
   --name string, -n string  Carve name to be deleted
   --env string, -e string   Environment to be used
   --help, -h                show help

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

## Expire carve

```properties
$ ./osctrl-cli carve expire -h
NAME:
   osctrl-cli carve expire - Mark a file carve query as expired

USAGE:
   osctrl-cli carve expire [options]

OPTIONS:
   --name string, -n string  Carve name to be expired
   --env string, -e string   Environment to be used
   --help, -h                show help

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

## Run carve

```properties
$ ./osctrl-cli carve run -h
NAME:
   osctrl-cli carve run - Start a new carve for a file or a directory

USAGE:
   osctrl-cli carve run [options]

OPTIONS:
   --path string, -p string                     File or directory path to be carved
   --env string, -e string                      Environment to be used
   --uuid string, -u string                     Node UUID(s) to be used. Comma separated for multiple values
   --host string, --hostname string, -H string  Node hostname(s) to be used. Comma separated for multiple values
   --platform string, -p string                 Node platform(s) to be used. Comma separated for multiple values
   --tag string, -t string                      Tag(s) to be used. Comma separated for multiple values
   --expiration int, -E int                     Expiration in hours (0 for no expiration) (default: 6)
   --help, -h                                   show help

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

## List carves

```properties
$ ./osctrl-cli carve list -h
NAME:
   osctrl-cli carve list - List file carves

USAGE:
   osctrl-cli carve list [options]

OPTIONS:
   --env string, -e string  Environment to be used
   --help, -h               show help

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

## List carve queries

```properties
$ ./osctrl-cli carve list-queries -h
NAME:
   osctrl-cli carve list-queries - List file carves queries

USAGE:
   osctrl-cli carve list-queries [options]

OPTIONS:
   --all, -A                Show all file carves queries
   --active, -a             Show active file carves queries
   --completed, -c          Show completed file carves queries
   --expired, -E            Show expired file carves queries
   --deleted, -d            Show deleted file carves queries
   --env string, -e string  Environment to be used
   --help, -h               show help

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
