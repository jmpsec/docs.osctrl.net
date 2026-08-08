---
title: "login"
description: "osctrl-cli login: login into API and generate JSON config file with token."
sidebar:
  order: 6
---

```properties
$ ./osctrl-cli login -h
NAME:
   osctrl-cli login - Login into API and generate JSON config file with token

USAGE:
   osctrl-cli login [options]

OPTIONS:
   --username string, -u string     User to be used in login
   --url string, -U string          URL to be used in login
   --environment string, -e string  Environment to be used in login
   --expiration int, -E int         Expiration in hours (0 for server default) (default: 6)
   --write-api-file, -w             Write API configuration to JSON file
   --help, -h                       show help

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
