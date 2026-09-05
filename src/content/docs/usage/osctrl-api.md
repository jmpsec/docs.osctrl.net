---
title: "osctrl-api"
description: "osctrl-api is the API service for osctrl."
sidebar:
  order: 2
---

`osctrl-api` is the API service for **osctrl**.

Execute `./osctrl-api help` to show the main help of the program:

```properties
$ ./osctrl-api help
NAME:
   osctrl-api - API service for osctrl, a fast and efficient osquery management

USAGE:
   osctrl-api [global options] [command [command options]]

DESCRIPTION:
   API service for osctrl, a fast and efficient osquery management

COMMANDS:
   help
   config-validate, config-verify  Validate YAML configuration file
   config-generate                 Generate an example configuration file using the current flag values

GLOBAL OPTIONS:
   --config, -c                      Provide service configuration via YAML file [$SERVICE_CONFIG]
   --config-file FILE, -C FILE       Load service configuration from FILE (default: "./config/api.yml") [$SERVICE_CONFIG_FILE]
   --listener string, -l string      Listener for the service (default: "127.0.0.1") [$SERVICE_LISTENER]
   --port int, -p int                TCP port for the service (default: 9000) [$SERVICE_PORT]
   --host string, -H string          Exposed hostname the service uses (default: "osctrl.net") [$SERVICE_HOST]
   --log-level string                Log level for the service (default: "info") [$SERVICE_LOG_LEVEL]
   --log-format string               Log format for the service (default: "json") [$SERVICE_LOG_FORMAT]
   --audit-log, --audit              Enable audit log for the service. Logs sensitive actions (logins, env mutations, query/carve runs, etc.). Disable only for local dev — production deployments MUST keep this on so SoC tooling has a stream to alert on. [$AUDIT_LOG]
   --trusted-proxies string          Comma-separated CIDR list whose X-Real-IP / X-Forwarded-For headers will be honored. Empty (default) ignores forwarding headers and uses RemoteAddr verbatim — prevents header-spoofed rate-limit bypass and audit-log poisoning. [$SERVICE_TRUSTED_PROXIES]
   --geoip-db string                 Path to a MaxMind GeoLite2-Country .mmdb file. Empty (default) disables GeoIP enrichment. [$SERVICE_GEOIP_DB]
   --posture-enabled                 Enable security and compliance posture ingestion/API/frontend surfaces [$SERVICE_POSTURE_ENABLED]
   --alerts-enabled                  Enable alerting routes, matching and dispatch. Requires restart to change. [$SERVICE_ALERTS_ENABLED]
   --service-config-enabled          Serve the service-config API and show the matching section in the SPA [$SERVICE_CONFIG_ENABLED]
   --log-sinks-enabled               Serve the log-sinks API and show the matching section in the SPA [$LOG_SINKS_ENABLED]
   --auth-providers-enabled          Serve the auth-providers API and show the matching section in the SPA [$AUTH_PROVIDERS_ENABLED]
   --mfa-required                    Require TOTP/passkey/security-key MFA for password logins [$SERVICE_MFA_REQUIRED]
   --mfa-issuer string               Label shown next to the account in authenticator apps [$SERVICE_MFA_ISSUER]
   --mfa-rpid string                 WebAuthn Relying Party ID; empty uses --host [$SERVICE_MFA_RPID]
   --mfa-origins string              Comma-separated browser origins allowed for WebAuthn ceremonies [$SERVICE_MFA_ORIGINS]
   --posture-query-prefix string     Prefix for scheduled query names ingested as posture data (default: "osctrl:posture:") [$SERVICE_POSTURE_QUERY_PREFIX]
   --auth none, -A none              Authentication mechanism for the service (jwt|none — none requires OSCTRL_INSECURE_NO_AUTH=1) (default: "jwt") [$SERVICE_AUTH]
   --logger string, -L string        Logger mechanism to handle status/result logs from nodes (default: "db") [$SERVICE_LOGGER]
   --loggers string                  Logger mechanisms as a comma-separated list [$SERVICE_LOGGERS]
   --logger-db-same                  Use the same DB configuration for the logger [$LOGGER_DB_SAME]
   --always-log, -a, --always        Always log status and on-demand query logs from nodes in database [$ALWAYS_LOG]
   --redis-connection-string string  Redis connection string, must include schema (<redis|rediss|unix>://<user>:<pass>@<host>:<port>/<db>?<options> [$REDIS_CONNECTION_STRING]
   --redis-host string               Redis host to be connected to (default: "127.0.0.1") [$REDIS_HOST]
   --redis-port int                  Redis port to be connected to (default: 6379) [$REDIS_PORT]
   --redis-pass string               Password to be used for redis [$REDIS_PASS]
   --redis-db int                    Redis database to be selected after connecting (default: 0) [$REDIS_DB]
   --redis-conn-retry int            Time in seconds to retry the connection to the cache, if set to 0 the service will stop if the connection fails (default: 10) [$REDIS_CONN_RETRY]
   --db-type string                  Type of backend to be used (default: "postgres") [$DB_TYPE]
   --db-host string                  Backend host to be connected to (default: "127.0.0.1") [$DB_HOST]
   --db-port int                     Backend port to be connected to (default: 5432) [$DB_PORT]
   --db-name string                  Database name to be used in the backend (default: "osctrl") [$DB_NAME]
   --db-user string                  Username to be used for the backend (default: "postgres") [$DB_USER]
   --db-pass string                  Password to be used for the backend (default: "postgres") [$DB_PASS]
   --db-sslmode string               SSL native support to encrypt the connection to the backend (default: "disable") [$DB_SSLMODE]
   --db-max-idle-conns int           Maximum number of connections in the idle connection pool (default: 20) [$DB_MAX_IDLE_CONNS]
   --db-max-open-conns int           Maximum number of open connections to the database (default: 100) [$DB_MAX_OPEN_CONNS]
   --db-conn-max-lifetime int        Maximum amount of time a connection may be reused (default: 30) [$DB_CONN_MAX_LIFETIME]
   --db-conn-retry int               Time in seconds to retry the connection to the database, if set to 0 the service will stop if the connection fails (default: 10) [$DB_CONN_RETRY]
   --db-filepath string              File path to the SQLite database, only used when type is sqlite (default: "./osctrl.db") [$DB_SQLITE_FILEPATH]
   --tls, -t                         Enable TLS termination. It requires certificate and key [$TLS_SERVER]
   --cert FILE, -T FILE              TLS termination certificate from FILE (default: "./config/tls.crt") [$TLS_CERTIFICATE]
   --key FILE, -K FILE               TLS termination private key from FILE (default: "./config/tls.key") [$TLS_KEY]
   --jwt-secret string               Password to be used for the backend [$JWT_SECRET]
   --jwt-expire int                  Maximum amount of hours for the tokens to expire (default: 3) [$JWT_EXPIRE]
   --oidc-enabled                    Enable the federated-login surface on osctrl-api [$OIDC_ENABLED]
   --oidc-issuer-url string          OIDC issuer URL (the realm root, /.well-known/openid-configuration is appended automatically) [$OIDC_ISSUER_URL]
   --oidc-client-id string           OIDC client ID registered with the IdP [$OIDC_CLIENT_ID]
   --oidc-client-secret string       OIDC client secret [$OIDC_CLIENT_SECRET]
   --oidc-redirect-url string        OIDC redirect URL — must match the IdP client config and end with /oidc/callback [$OIDC_REDIRECT_URL]
   --oidc-scopes string              OIDC scopes as a comma-separated list (defaults to "openid,profile,email") [$OIDC_SCOPES]
   --oidc-username-claim string      ID-token claim used as the osctrl username (preferred_username, email, sub) [$OIDC_USERNAME_CLAIM]
   --oidc-groups-claim string        ID-token claim that contains group memberships (default: groups) [$OIDC_GROUPS_CLAIM]
   --oidc-required-groups string     Comma-separated list of groups — user must belong to at least one to log in [$OIDC_REQUIRED_GROUPS]
   --oidc-jit-provision              Auto-create osctrl users on first OIDC login (as non-admin) [$OIDC_JIT_PROVISION]
   --oidc-use-pkce                   Enable PKCE (S256) for the OIDC Authorization Code flow [$OIDC_USE_PKCE]
   --oidc-link-local-accounts        Let OIDC claim an existing local password account with the same username [$OIDC_LINK_LOCAL_ACCOUNTS]
   --saml-enabled                    Enable the SAML 2.0 federated-login surface on osctrl-api [$SAML_ENABLED]
   --saml-idp-metadata-url string    URL to the IdP's SAML metadata document — fetched once at startup, signing certs + SSO endpoint discovered from it [$SAML_IDP_METADATA_URL]
   --saml-entity-id string           SP entity ID — what the IdP knows us by (conventionally the metadata URL) [$SAML_ENTITY_ID]
   --saml-acs-url string             Assertion Consumer Service URL — where the IdP POSTs SAMLResponse (must end with /api/v1/auth/saml/acs) [$SAML_ACS_URL]
   --saml-jit-provision              Auto-create osctrl users on first SAML login (as non-admin) [$SAML_JIT_PROVISION]
   --saml-link-local-accounts        Let SAML claim an existing local password account with the same username [$SAML_LINK_LOCAL_ACCOUNTS]
   --saml-username-attribute string  SAML attribute (Name or FriendlyName) whose value becomes the osctrl username; empty = use NameID verbatim [$SAML_USERNAME_ATTRIBUTE]
   --saml-signing-cert string        Path to PEM cert used to sign outbound AuthnRequests (paired with --saml-signing-key); empty disables AuthnRequest signing [$SAML_SIGNING_CERT]
   --saml-signing-key string         Path to PEM RSA private key used to sign outbound AuthnRequests (paired with --saml-signing-cert) [$SAML_SIGNING_KEY]
   --saml-force-authn                Force re-authentication at the IdP on every SAML login (defaults true; SLO substitute since v1 has no SAML SLO) [$SAML_FORCE_AUTHN]
   --osquery-version string          Version of osquery to be used (default: "5.23.0") [$OSQUERY_VERSION]
   --osquery-tables-file string      File with the osquery tables to be used (default: "./data/5.23.0.json") [$OSQUERY_TABLES]
   --osquery-logger                  Enable remote tls logger for osquery [$OSQUERY_LOGGER]
   --osquery-config                  Enable remote tls config for osquery [$OSQUERY_CONFIG]
   --osquery-query                   Enable remote tls queries for osquery [$OSQUERY_QUERY]
   --osquery-carve                   Enable remote tls carver for osquery [$OSQUERY_CARVE]
   --read-only-configuration         Disable configuration changes via operator UI or API services [$OSQUERY_READ_ONLY]
   --carver-type string              Carver to be used to receive files extracted from nodes (default: "db") [$CARVER_TYPE]
   --carver-s3-bucket string         S3 bucket to be used as configuration for carves [$CARVER_S3_BUCKET]
   --carver-s3-region string         S3 region to be used as configuration for carves [$CARVER_S3_REGION]
   --carve-s3-key-id string          S3 access key id to be used as configuration for carves [$CARVER_S3_KEY_ID]
   --carve-s3-secret string          S3 access key secret to be used as configuration for carves [$CARVER_S3_SECRET]
   --carver-local-dir string         Local directory to store carved files (default: "./carved_files/") [$CARVER_LOCAL_DIR]
   --enable-http-debug               Enable HTTP Debug mode to dump full HTTP incoming request [$HTTP_DEBUG]
   --http-debug-file string          File to dump the HTTP requests when HTTP Debug mode is enabled (default: "./debug-http-api.log") [$HTTP_DEBUG_FILE]
   --http-debug-show-body            Show body of the HTTP requests when HTTP Debug mode is enabled [$HTTP_DEBUG_SHOW_BODY]
   --version, -v                     Print version information
   --help, -h                        show help
```
