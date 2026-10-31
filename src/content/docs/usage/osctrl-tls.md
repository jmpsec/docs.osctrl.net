---
title: "osctrl-tls"
description: "osctrl-tls is the TLS service for osctrl."
sidebar:
  order: 1
---

`osctrl-tls` is the TLS service for **osctrl**.

Execute `./osctrl-tls help` to show the main help of the program:

```properties
$ ./osctrl-tls help
NAME:
   osctrl-tls - TLS service for osctrl, a fast and efficient osquery management

USAGE:
   osctrl-tls [global options] [command [command options]]

DESCRIPTION:
   TLS service for osctrl, a fast and efficient osquery management

COMMANDS:
   help
   config-validate, config-verify  Validate YAML configuration file
   config-generate                 Generate an example configuration file using the current flag values

GLOBAL OPTIONS:
   --query-dispatch-ttl duration                       Empty distributed-query cache TTL; non-positive values use the 2m default (default: 0s) [$QUERY_DISPATCH_TTL]
   --config, -c                                        Provide service configuration via YAML file [$SERVICE_CONFIG]
   --config-file FILE, -C FILE                         Load service configuration from FILE (default: "./config/tls.yml") [$SERVICE_CONFIG_FILE]
   --listener string, -l string                        Listener for the service (default: "127.0.0.1") [$SERVICE_LISTENER]
   --port int, -p int                                  TCP port for the service (default: 9000) [$SERVICE_PORT]
   --host string, -H string                            Exposed hostname the service uses (default: "osctrl.net") [$SERVICE_HOST]
   --log-level string                                  Log level for the service (default: "info") [$SERVICE_LOG_LEVEL]
   --log-format string                                 Log format for the service (default: "json") [$SERVICE_LOG_FORMAT]
   --audit-log, --audit                                Enable audit log for the service. Logs sensitive actions (logins, env mutations, query/carve runs, etc.). Disable only for local dev — production deployments MUST keep this on so SoC tooling has a stream to alert on. [$AUDIT_LOG]
   --trusted-proxies string                            Comma-separated CIDR list whose X-Real-IP / X-Forwarded-For headers will be honored. Empty (default) ignores forwarding headers and uses RemoteAddr verbatim — prevents header-spoofed rate-limit bypass and audit-log poisoning. [$SERVICE_TRUSTED_PROXIES]
   --base-path string                                  Public URL prefix the SPA and API are served under (e.g. /osctrl). The reverse proxy must strip it before forwarding; it is used for cookie paths, redirects and login URLs. Empty (default) serves at the root. [$SERVICE_BASE_PATH]
   --geoip-db string                                   Path to a MaxMind GeoLite2-Country .mmdb file. When set, node IP addresses are resolved to country codes (shown as flag emojis in the SPA). Empty (default) disables GeoIP entirely. [$SERVICE_GEOIP_DB]
   --posture-enabled                                   Enable the security and compliance posture system. Disabled by default; when enabled osctrl-tls ingests posture query results and osctrl-api exposes posture endpoints to the SPA. [$SERVICE_POSTURE_ENABLED]
   --alerts-enabled                                    Enable the alerting subsystem: rule matching on ingested logs and dispatch through configured channels. Disabled by default; when enabled osctrl-tls evaluates alert rules on the log ingest path. Requires a service restart to change. [$SERVICE_ALERTS_ENABLED]
   --events-enabled                                    Enable best-effort query/carve change notifications (API and TLS). [$SERVICE_EVENTS_ENABLED]
   --events-namespace string                           Shared API/TLS event namespace, unique per deployment within Redis (required when enabled). [$SERVICE_EVENTS_NAMESPACE]
   --health-enabled                                    Enable the health/system-status subsystem: component health, service heartbeats and runtime stats in the SPA. Disabled by default; when enabled osctrl-tls writes a heartbeat row every 60s and osctrl-api serves /api/v1/health/status. Requires a service restart to change. [$SERVICE_HEALTH_ENABLED]
   --vuln-enabled                                      Enable vulnerability monitoring: osctrl-tls stores software inventory from osctrl:vuln: scheduled queries and osctrl-api matches it against OSV and CISA KEV data. Disabled by default; when off no tables are created and nothing is downloaded. Requires a service restart to change. [$SERVICE_VULN_ENABLED]
   --service-config-enabled                            Serve the service-config API and show the matching section in the SPA. Disabled by default: the YAML sections are still seeded into the database at every boot and resolved back at startup, but none of the /api/v1/service-config routes are registered — change the rows or the YAML file directly instead. [$SERVICE_CONFIG_ENABLED]
   --log-sinks-enabled                                 Serve the log-sinks API and show the matching section in the SPA. Independent of --service-config-enabled. Set false to hide the log-sinks management UI. [$LOG_SINKS_ENABLED]
   --auth-providers-enabled                            Serve the auth-providers API and show the matching section in the SPA. Independent of --service-config-enabled. Set false to hide the auth-providers management UI. [$AUTH_PROVIDERS_ENABLED]
   --mfa-required                                      Require a second authentication factor (TOTP, passkey or security key) for password logins. Users without one enroll at their next login. Service accounts and federated logins are unaffected. [$SERVICE_MFA_REQUIRED]
   --mfa-issuer string                                 Label shown next to the account in authenticator apps. Defaults to "osctrl". [$SERVICE_MFA_ISSUER]
   --mfa-rpid string                                   WebAuthn Relying Party ID — the registrable domain passkeys and security keys are bound to, no scheme or port. Empty uses --host. Changing it invalidates every registered credential. [$SERVICE_MFA_RPID]
   --mfa-origins string                                Comma-separated origins the SPA is served from, with scheme and any non-default port. Empty uses https://<host>. A browser refuses a WebAuthn ceremony whose origin is not listed. [$SERVICE_MFA_ORIGINS]
   --posture-query-prefix string                       Prefix for scheduled query names whose results are ingested as node posture data when --posture-enabled is true. (default: "osctrl:posture:") [$SERVICE_POSTURE_QUERY_PREFIX]
   --auth string, -A string                            Authentication mechanism for the service (default: "none") [$SERVICE_AUTH]
   --logger string, -L string                          Logger mechanism to handle status/result logs from nodes (default: "db") [$SERVICE_LOGGER]
   --loggers string                                    Logger mechanisms to handle status/result logs from nodes as a comma-separated list [$SERVICE_LOGGERS]
   --logger-db-same                                    Use the same DB configuration for the logger [$LOGGER_DB_SAME]
   --always-log, -a, --always                          Always log status and on-demand query logs from nodes in database [$ALWAYS_LOG]
   --metrics-listener string                           Listener for prometheus metrics (default: "127.0.0.1") [$METRICS_LISTENER]
   --metrics-port int                                  Port for exposing prometheus metrics (default: 9090) [$METRICS_PORT]
   --metrics-enabled                                   Enable prometheus metrics [$METRICS_ENABLED]
   --writer-batch-size int                             Maximum number of events before flushing (default: 50) [$WRITER_BATCH_SIZE]
   --writer-timeout duration                           Maximum wait time before flushing (default: 1m0s) [$WRITER_TIMEOUT]
   --writer-buffer-size int                            Size of the event channel buffer (default: 2000) [$WRITER_BUFFER_SIZE]
   --redis-connection-string string                    Redis connection string, must include schema (<redis|rediss|unix>://<user>:<pass>@<host>:<port>/<db>?<options> [$REDIS_CONNECTION_STRING]
   --redis-host string                                 Redis host to be connected to (default: "127.0.0.1") [$REDIS_HOST]
   --redis-port int                                    Redis port to be connected to (default: 6379) [$REDIS_PORT]
   --redis-pass string                                 Password to be used for redis [$REDIS_PASS]
   --redis-db int                                      Redis database to be selected after connecting (default: 0) [$REDIS_DB]
   --redis-conn-retry int                              Time in seconds to retry the connection to the cache, if set to 0 the service will stop if the connection fails (default: 10) [$REDIS_CONN_RETRY]
   --db-type string                                    Type of backend to be used (default: "postgres") [$DB_TYPE]
   --db-host string                                    Backend host to be connected to (default: "127.0.0.1") [$DB_HOST]
   --db-port int                                       Backend port to be connected to (default: 5432) [$DB_PORT]
   --db-name string                                    Database name to be used in the backend (default: "osctrl") [$DB_NAME]
   --db-user string                                    Username to be used for the backend (default: "postgres") [$DB_USER]
   --db-pass string                                    Password to be used for the backend (default: "postgres") [$DB_PASS]
   --db-sslmode string                                 SSL native support to encrypt the connection to the backend (default: "disable") [$DB_SSLMODE]
   --db-max-idle-conns int                             Maximum number of connections in the idle connection pool (default: 20) [$DB_MAX_IDLE_CONNS]
   --db-max-open-conns int                             Maximum number of open connections to the database (default: 100) [$DB_MAX_OPEN_CONNS]
   --db-conn-max-lifetime int                          Maximum amount of time a connection may be reused (default: 30) [$DB_CONN_MAX_LIFETIME]
   --db-conn-retry int                                 Time in seconds to retry the connection to the database, if set to 0 the service will stop if the connection fails (default: 10) [$DB_CONN_RETRY]
   --db-filepath string                                File path to the SQLite database, only used when type is sqlite (default: "./osctrl.db") [$DB_SQLITE_FILEPATH]
   --db-read-enabled                                   Open a second connection to a read replica for read-heavy, staleness-tolerant queries (osctrl-api: dashboard, node lists, audit, posture, vulnerabilities; osctrl-tls: alert node scans, osctrld metrics). Not supported with sqlite [$DB_READ_ENABLED]
   --db-read-host string                               Read replica host (defaults to the primary host) [$DB_READ_HOST]
   --db-read-port int                                  Read replica port (defaults to the primary port) (default: 0) [$DB_READ_PORT]
   --db-read-user string                               Read replica username (defaults to the primary username) [$DB_READ_USER]
   --db-read-pass string                               Read replica password (defaults to the primary password) [$DB_READ_PASS]
   --db-read-sslmode string                            Read replica SSL mode (defaults to the primary SSL mode) [$DB_READ_SSLMODE]
   --db-read-max-idle-conns int                        Maximum idle connections in the read replica pool (defaults to the primary value) (default: 0) [$DB_READ_MAX_IDLE_CONNS]
   --db-read-max-open-conns int                        Maximum open connections to the read replica (defaults to the primary value) (default: 0) [$DB_READ_MAX_OPEN_CONNS]
   --tls, -t                                           Enable TLS termination. It requires certificate and key [$TLS_SERVER]
   --cert FILE, -T FILE                                TLS termination certificate from FILE (default: "./config/tls.crt") [$TLS_CERTIFICATE]
   --key FILE, -K FILE                                 TLS termination private key from FILE (default: "./config/tls.key") [$TLS_KEY]
   --enable-osctrld                                    Enable osctrld endpoints and functionality. [$OSCTRLD]
   --osquery-version string                            Version of osquery to be used (default: "5.24.0") [$OSQUERY_VERSION]
   --osquery-tables-file string                        File with the osquery tables to be used (default: "./data/5.24.0.json") [$OSQUERY_TABLES]
   --osquery-logger                                    Enable remote tls logger for osquery [$OSQUERY_LOGGER]
   --osquery-config                                    Enable remote tls config for osquery [$OSQUERY_CONFIG]
   --osquery-query                                     Enable remote tls queries for osquery [$OSQUERY_QUERY]
   --osquery-carve                                     Enable remote tls carver for osquery [$OSQUERY_CARVE]
   --osquery-accelerated                               Enable accelerated mode when returning distributed queries to osquery [$OSQUERY_ACCELERATED]
   --osquery-file-explorer                             Enable on-demand node file explorer queries [$OSQUERY_FILE_EXPLORER]
   --osquery-console                                   Enable on-demand per-node console queries [$OSQUERY_CONSOLE]
   --read-only-configuration                           Disable configuration changes via admin or api services [$OSQUERY_READ_ONLY]
   --carver-type string                                Carver to be used to receive files extracted from nodes (db or s3) (default: "db") [$CARVER_TYPE]
   --carver-s3-bucket string                           S3 bucket to be used as configuration for carves [$CARVER_S3_BUCKET]
   --carver-s3-region string                           S3 region to be used as configuration for carves [$CARVER_S3_REGION]
   --carve-s3-key-id string                            S3 access key id to be used as configuration for carves [$CARVER_S3_KEY_ID]
   --carve-s3-secret string                            S3 access key secret to be used as configuration for carves [$CARVER_S3_SECRET]
   --carver-local-dir string                           Unused: the local carver is not supported (default: "./carved_files/") [$CARVER_LOCAL_DIR]
   --rate-limit-enroll-burst int                       Maximum burst for the enroll rate limiter (default: 20) [$RATE_LIMIT_ENROLL_BURST]
   --rate-limit-enroll-period duration                 Refill period for the enroll rate limiter (default: 1m0s) [$RATE_LIMIT_ENROLL_PERIOD]
   --rate-limit-enroll-evict-after duration            Idle bucket eviction window for the enroll rate limiter (default: 10m0s) [$RATE_LIMIT_ENROLL_EVICT_AFTER]
   --rate-limit-enroll-retry-after int                 Retry-After seconds returned by the enroll rate limiter (default: 60) [$RATE_LIMIT_ENROLL_RETRY_AFTER]
   --rate-limit-enroll-max-buckets int                 Maximum per-key buckets for the enroll rate limiter; 0 uses the default (default: 0) [$RATE_LIMIT_ENROLL_MAX_BUCKETS]
   --rate-limit-osctrld-auth-burst int                 Maximum burst for the osctrld-auth rate limiter (default: 10) [$RATE_LIMIT_OSCTRLD_AUTH_BURST]
   --rate-limit-osctrld-auth-period duration           Refill period for the osctrld-auth rate limiter (default: 1h0m0s) [$RATE_LIMIT_OSCTRLD_AUTH_PERIOD]
   --rate-limit-osctrld-auth-evict-after duration      Idle bucket eviction window for the osctrld-auth rate limiter (default: 2h0m0s) [$RATE_LIMIT_OSCTRLD_AUTH_EVICT_AFTER]
   --rate-limit-osctrld-auth-retry-after int           Retry-After seconds returned by the osctrld-auth rate limiter (default: 360) [$RATE_LIMIT_OSCTRLD_AUTH_RETRY_AFTER]
   --rate-limit-osctrld-auth-max-buckets int           Maximum per-key buckets for the osctrld-auth rate limiter; 0 uses the default (default: 0) [$RATE_LIMIT_OSCTRLD_AUTH_MAX_BUCKETS]
   --rate-limit-osctrld-burst int                      Maximum burst for the osctrld rate limiter (default: 300) [$RATE_LIMIT_OSCTRLD_BURST]
   --rate-limit-osctrld-period duration                Refill period for the osctrld rate limiter (default: 1m0s) [$RATE_LIMIT_OSCTRLD_PERIOD]
   --rate-limit-osctrld-evict-after duration           Idle bucket eviction window for the osctrld rate limiter (default: 10m0s) [$RATE_LIMIT_OSCTRLD_EVICT_AFTER]
   --rate-limit-osctrld-retry-after int                Retry-After seconds returned by the osctrld rate limiter (default: 60) [$RATE_LIMIT_OSCTRLD_RETRY_AFTER]
   --rate-limit-osctrld-max-buckets int                Maximum per-key buckets for the osctrld rate limiter; 0 uses the default (default: 0) [$RATE_LIMIT_OSCTRLD_MAX_BUCKETS]
   --rate-limit-osctrld-download-burst int             Maximum burst for the osctrld-download rate limiter (default: 30) [$RATE_LIMIT_OSCTRLD_DOWNLOAD_BURST]
   --rate-limit-osctrld-download-period duration       Refill period for the osctrld-download rate limiter (default: 1m0s) [$RATE_LIMIT_OSCTRLD_DOWNLOAD_PERIOD]
   --rate-limit-osctrld-download-evict-after duration  Idle bucket eviction window for the osctrld-download rate limiter (default: 10m0s) [$RATE_LIMIT_OSCTRLD_DOWNLOAD_EVICT_AFTER]
   --rate-limit-osctrld-download-retry-after int       Retry-After seconds returned by the osctrld-download rate limiter (default: 60) [$RATE_LIMIT_OSCTRLD_DOWNLOAD_RETRY_AFTER]
   --rate-limit-osctrld-download-max-buckets int       Maximum per-key buckets for the osctrld-download rate limiter; 0 uses the default (default: 0) [$RATE_LIMIT_OSCTRLD_DOWNLOAD_MAX_BUCKETS]
   --log-s3-bucket string                              S3 bucket to be used as configuration for logging [$LOG_S3_BUCKET]
   --log-s3-region string                              S3 region to be used as configuration for logging [$LOG_S3_REGION]
   --log-s3-key-id string                              S3 access key id to be used as configuration for logging [$LOG_S3_KEY_ID]
   --log-s3-secret string                              S3 access key secret to be used as configuration for logging [$LOG_S3_SECRET]
   --log-kafka-boostrap-servers string                 Kafka bootstrap servers to be used as configuration for logging [$LOG_KAFKA_BOOTSTRAP_SERVERS]
   --log-kafka-sslca-location string                   Kafka sslca location to be used as configuration for logging [$LOG_KAFKA_SSLCA_LOCATION]
   --log-kafka-connection-timeout duration             Kafka connection timeout to be used as configuration for logging (default: 5s) [$LOG_KAFKA_CONNECTION_TIMEOUT]
   --log-kafka-topic string                            Kafka topic to be used as configuration for logging [$LOG_KAFKA_TOPIC]
   --log-kafka-sasl-mechanism string                   Kafka sasl mechanism' to be used as configuration for logging [$LOG_KAFKA_SASL_MECHANISM]
   --log-kafka-sasl-username string                    Kafka sasl username' to be used as configuration for logging [$LOG_KAFKA_SASL_USERNAME]
   --log-kafka-sasl-password string                    Kafka sasl password' to be used as configuration for logging [$LOG_KAFKA_SASL_PASSWORD]
   --enable-http-debug                                 Enable HTTP Debug mode to dump full HTTP incoming request [$HTTP_DEBUG]
   --http-debug-file string                            File to dump the HTTP requests when HTTP Debug mode is enabled (default: "./debug-http-tls.log") [$HTTP_DEBUG_FILE]
   --http-debug-show-body                              Show body of the HTTP requests when HTTP Debug mode is enabled [$HTTP_DEBUG_SHOW_BODY]
   --http-debug-host string                            Only dump HTTP requests from the osquery node with this UUID / host_identifier (case-insensitive). Empty dumps everything when --enable-http-debug is set. [$HTTP_DEBUG_HOST]
   --db-health-check                                   Enable a background DB liveness monitor. After --db-health-threshold consecutive ping failures, EnvCache and SettingsCache switch to stale-serve mode (serve cached entries on DB miss, extend TTLs) so osquery nodes keep getting config/logs during a DB outage. Disabled by default. [$DB_HEALTH_CHECK]
   --db-health-interval int                            Seconds between DB health pings when --db-health-check is enabled. (default: 5) [$DB_HEALTH_INTERVAL]
   --db-health-threshold int                           Consecutive DB ping failures required before EnvCache and SettingsCache enter stale-serve mode. (default: 3) [$DB_HEALTH_THRESHOLD]
   --version, -v                                       Print version information
   --help, -h                                          show help
```
