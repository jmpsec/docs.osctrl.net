---
title: "Configuration"
description: "Configure the osctrl components with a single YAML file per service, covering all required fields and defaults."
sidebar:
  order: 3
---

Each [component](/components/) of **osctrl** requires configuration in order to operate properly. **As of [PR #754](https://github.com/jmpsec/osctrl/pull/754), osctrl uses a single YAML configuration file per service**, consolidating all settings into one place.

:::caution
**BREAKING CHANGE**: The previous JSON-based configuration system with multiple separate files (`service.json`, `db.json`, `redis.json`, `jwt.json`, `saml.json`, etc.) has been replaced with a single YAML file per service. If you are upgrading from an older version, you will need to migrate your configuration to the new YAML format.
:::

## Single YAML Configuration

All osctrl services ([osctrl-tls](/components/osctrl-tls/) and [osctrl-api](/components/osctrl-api/)) now use a unified YAML configuration file that includes all necessary settings in one place. The default filenames are:

* `tls.yaml` for [osctrl-tls](/components/osctrl-tls/)
* `api.yaml` for [osctrl-api](/components/osctrl-api/)

You can specify a different configuration file using the `--config-file` or `-C` flag.

Only the `service`, `db` and `redis` sections are required — every other section (`rateLimits`, `osquery`, `saml`, `oidc`, `jwt`, `tls`, `logger`, `carver`, `debug`, and the osctrl-tls-only `batchWriter`, `configEndpoints`, `osctrld`, `metrics`) can be omitted entirely and the service falls back to its built-in defaults, or to values already stored in the database.

### Generating and Validating Configuration

Each osctrl binary includes built-in commands to help you create and validate configuration files:

#### Generate Configuration

To generate a valid YAML configuration file with default values:

```bash
./osctrl-tls config-generate -f tls.yaml
./osctrl-api config-generate -f api.yaml
```

This will create configuration files (`tls.yaml`, `api.yaml`) populated with all required fields and default settings. If not using `-f`, the file will be created in `config/<service>.yml` by default.

The generated configuration includes:

* All required configuration sections
* Default values for each setting
* Service-specific options relevant to each binary

#### Verify Configuration

To validate an existing configuration file before starting the service:

```bash
./osctrl-tls config-verify --file tls.yaml
./osctrl-api config-verify --file api.yaml
```

The verification process checks:

* YAML syntax validity
* Required fields are present
* Configuration values are within valid ranges
* Authentication method compatibility
* Logger and carver type validity
* Database and Redis connection parameters

If the configuration is valid, the command will exit with status code 0. If there are errors, detailed error messages will be displayed indicating what needs to be fixed.

:::tip
It's recommended to run `config-verify` on your configuration files before deploying to production to catch configuration errors early.
:::

### Configuration Structure

The YAML configuration file contains the following main sections:

#### Service Configuration

Basic service settings including network listener, ports, authentication, and logging:

```yaml
service:
  listener: "0.0.0.0"               # Network interface to bind to
  port: "9000"                      # TCP port for the service
  host: "tls.example.com"           # Public hostname
  auth: "none"                      # Authentication method
  logLevel: "info"                  # Logging level: debug, info, warn, error
  logFormat: "json"                 # Log format: json, console
  serviceConfigEnabled: false        # osctrl-api only: expose /service-config API + frontend UI
  mfaRequired: false                 # osctrl-api only: require MFA for password logins
  mfaIssuer: ""                      # Authenticator app label; empty uses "osctrl (<host>)"
  mfaRPID: ""                        # WebAuthn Relying Party ID; empty uses `host`
  mfaOrigins: ""                     # Comma-separated origins allowed for WebAuthn ceremonies
  auditLog: true                    # Enable audit logging for sensitive actions
  trustedProxies: ""                 # CIDR list trusted for X-Forwarded-For/X-Real-IP
  geoipDBPath: ""                    # Path to a GeoLite2-Country .mmdb for node IP geolocation
  postureEnabled: false              # Enable security & compliance posture collection
  postureQueryPrefix: "osctrl:posture:" # Scheduled-query name prefix ingested as posture data
  dbHealthCheck: false               # Enable DB health monitor / stale-serve mode on outages
  dbHealthInterval: 5                # Seconds between DB health pings
  dbHealthThreshold: 3               # Consecutive failures before entering stale-serve mode
```

**Authentication types** (`auth`):

* `none` - No authentication. For [osctrl-tls](/components/osctrl-tls/) this is the only valid value — osquery nodes authenticate with their enroll secret and `node_key` instead. For [osctrl-api](/components/osctrl-api/), `none` requires `OSCTRL_INSECURE_NO_AUTH=1` in the environment and is intended for local development only, since it impersonates a super-admin on every request.
* `jwt` - JWT token-based authentication (osctrl-api). The only supported value for production deployments.

SAML and OIDC federated login are configured independently of `auth`, via the `saml.enabled` and `oidc.enabled` switches — the frontend discovers which methods are available through `GET /api/v1/auth/methods` and can offer JWT, SAML and OIDC login side by side.

**MFA** (`mfaRequired` and friends) adds a second factor — an authenticator app (TOTP) or a WebAuthn passkey/security key — to password logins on osctrl-api. When `mfaRequired: true`, users without an enrolled factor are sent to enrollment on their next login instead of being locked out; service accounts (token auth) and federated SAML/OIDC logins are unaffected. Users can always enroll a factor voluntarily from their profile page regardless of this setting.

`serviceConfigEnabled`, MFA and posture fields also appear under `service:` in `tls.yaml`, but only `osctrl-api` acts on them — they're kept in both files so the shared `service` section round-trips through the [Service Configuration API](#service-configuration-api-and-restart).

#### Database Configuration

Backend database connection settings:

```yaml
db:
  type: "postgres"                  # Database type: postgres, mysql, sqlite
  host: "127.0.0.1"
  port: "5432"
  name: "osctrl"
  username: "postgres"
  password: "postgres"
  sslmode: "disable"                # SSL mode for database connection
  maxIdleConns: 20                  # Maximum idle connections
  maxOpenConns: 100                 # Maximum open connections
  connMaxLifetime: 30               # Connection max lifetime (minutes)
  connRetry: 5                      # Connection retry timeout (seconds, 0=no retry)
  filePath: "./osctrl.db"           # For SQLite only
```

#### Redis Configuration

Cache configuration for session management and performance:

```yaml
redis:
  host: "127.0.0.1"
  port: "6379"
  password: ""
  db: 0
  connectionString: ""              # Optional: full connection string
  connRetry: 5                      # Connection retry timeout (seconds)
```

#### Rate Limits Configuration

HTTP rate limiting, shared by both services but enforced per endpoint:

```yaml
rateLimits:
  login:                            # Password/SSO login attempts (osctrl-api)
    burst: 10                       # Tokens available immediately
    period: 1m                      # Refill window
    evictAfter: 10m                 # Idle client buckets are evicted after this
    retryAfter: 60                  # Retry-After response header, in seconds
    maxBuckets: 0                   # Max tracked client buckets; 0 = internal default
  preAuth:                          # Pre-auth discovery routes (osctrl-api)
    burst: 60
    period: 1m
    evictAfter: 10m
    retryAfter: 60
    maxBuckets: 0
  serviceConfigApply:               # POST /service-config/apply restart requests (osctrl-api)
    burst: 3
    period: 10m
    evictAfter: 30m
    retryAfter: 60
    maxBuckets: 0
  enroll:                           # osquery enroll attempts (osctrl-tls)
    burst: 20
    period: 1m
    evictAfter: 10m
    retryAfter: 60
    maxBuckets: 0
```

`osctrl-api` only enforces `login`, `preAuth` and `serviceConfigApply`; `osctrl-tls` only enforces `enroll`. Every limiter still has to appear in both `tls.yaml` and `api.yaml` so the section has one shared shape and can round-trip through the [Service Configuration API](#service-configuration-api-and-restart) — the values the other service doesn't use are simply ignored.

#### Logger Configuration

Logging destination and settings:

```yaml
logger:
  type: "db"                        # Logger type: db, splunk, graylog, s3, file, kinesis, kafka, logstash, elastic
  loggerDBSame: true                # Use same DB config as main DB for logging
  alwaysLog: false                  # Always log to DB regardless of other loggers

  # Database logger settings (if type: db and loggerDBSame: false)
  db:
    type: "postgres"
    host: "127.0.0.1"
    # ... (same structure as main db config)

  # S3 logger settings (if type: s3)
  s3:
    bucket: "osctrl-logs"
    region: "us-east-1"
    accessKey: "your-access-key"
    secretAccessKey: "your-secret-key"

  # Splunk logger settings (if type: splunk)
  splunk:
    url: "https://splunk.example.com:8088/services/collector"
    token: "your-hec-token"
    host: "osctrl"
    index: "osquery"

  # Graylog logger settings (if type: graylog)
  graylog:
    url: "http://graylog.example.com:12201/gelf"
    host: "osctrl"
    queries: "osquery_queries"
    status: "osquery_status"
    results: "osquery_results"

  # File logger settings (if type: file)
  local:
    filePath: "/var/log/osctrl/osctrl.log"
    maxSize: 25                     # Max file size in MB before rotation
    maxBackups: 5                   # Number of old log files to retain
    maxAge: 10                      # Max days to retain old log files
    compress: true                  # Compress rotated files with gzip
```

**Logger types**:

* `db` - Store logs in the backend database
* `splunk` - Send logs to Splunk HEC
* `graylog` - Send logs to Graylog GELF endpoint
* `s3` - Upload logs to AWS S3
* `file` - Write logs to local file with rotation
* `kinesis` - Send logs to AWS Kinesis
* `kafka` - Send logs to Kafka topics
* `logstash` - Send logs to Logstash
* `elastic` - Send logs directly to Elasticsearch

#### Carver Configuration

File carving storage settings:

```yaml
carver:
  type: "db"                        # Carver type: db, s3, local

  # S3 carver settings (if type: s3)
  s3:
    bucket: "osctrl-carves"
    region: "us-east-1"
    accessKey: "your-access-key"
    secretAccessKey: "your-secret-key"

  # Local carver settings (if type: local)
  local:
    carvesDir: "/var/osctrl/carves" # Directory to store carved files
```

**Carver types**:

* `db` - Store carved files in the database
* `s3` - Upload carved files to AWS S3
* `local` - Store carved files in local directory

#### TLS Configuration

TLS termination settings:

```yaml
tls:
  termination: true                 # Enable TLS termination
  certificateFile: "/path/to/cert.pem"
  keyFile: "/path/to/key.pem"
```

#### Osquery Configuration

Osquery-specific settings for TLS service:

```yaml
osquery:
  version: "5.12.1"                 # Osquery version
  tablesFile: "data/5.12.1.json"   # Path to osquery tables JSON
  logger: true                      # Enable remote TLS logger endpoint
  config: true                      # Enable remote TLS config endpoint
  query: true                       # Enable remote TLS query endpoints
  carve: true                       # Enable remote TLS carver endpoints
  accelerated: false                # Enable accelerated query polling
  console: false                    # Enable the per-node interactive console
  fileExplorer: false                # Enable the per-node file explorer (requires query: true)
  readOnly: false                   # Prevent operator-driven osquery configuration changes
```

* `console` gates the per-node interactive console (reachable from the node detail page in [osctrl-frontend](/components/osctrl-frontend/)): on osctrl-api it controls whether the feature is advertised and its routes registered (also requires `query: true`); on osctrl-tls it controls whether active console sessions can request accelerated query polling.
* `fileExplorer` behaves the same way for the per-node file explorer, and no longer requires `accelerated: true` — only `query: true`.

#### Metrics Configuration

Prometheus metrics endpoint settings (TLS service only):

```yaml
metrics:
  enabled: false                    # Enable Prometheus metrics
  listener: "0.0.0.0"
  port: "9090"
```

#### JWT Configuration

JWT authentication settings:

```yaml
jwt:
  jwtSecret: "your-jwt-secret"      # Secret for signing JWT tokens
  hoursToExpire: 3                  # Token expiration time in hours
```

#### SAML Configuration

SAML 2.0 federated login for `osctrl-api`, disabled by default. When `enabled: true`, the API fetches the IdP metadata at startup and refuses to start if that fails; the frontend discovers the method via `GET /api/v1/auth/methods` and shows a "Continue with SAML" button automatically:

```yaml
saml:
  enabled: false                    # Enable SAML routes
  entityId: ""                      # SP entity ID, conventionally the metadata URL
  acsUrl: ""                        # Where the IdP POSTs the SAMLResponse (must end with /api/v1/auth/saml/acs)
  metadataUrl: "https://idp.example.com/metadata"
  logoutUrl: "https://idp.example.com/logout" # IdP session-termination URL, used on logout
  jitProvision: false                # Just-in-time user provisioning, as non-admin
  usernameAttribute: ""              # SAML attribute mapped to the osctrl username
  signingCertPath: ""                # PEM cert + key for signing outbound AuthnRequests
  signingKeyPath: ""
  forceAuthn: true                   # Force re-authentication at the IdP on every login
```

Register osctrl with the IdP by pointing it at the SP metadata URL: `https://<host>/api/v1/auth/saml/metadata`. The `certPath`, `keyPath`, `rootUrl`, `loginUrl` and `spInitiated` fields are legacy, consumed only by the retired `osctrl-admin` service, and ignored by `osctrl-api`.

#### Osctrld Configuration

Settings for the osctrld endpoints exposed by `osctrl-tls`:

```yaml
osctrld:
  enabled: false                    # Enable osctrld endpoints
```

#### Batch Writer Configuration

Database batch writer settings for TLS service:

```yaml
batchWriter:
  writerBatchSize: 50               # Events before flushing
  writerTimeout: 60s                # Max wait time before flush
  writerBufferSize: 2000            # Event channel buffer size
```

#### Debug Configuration

HTTP request debugging:

```yaml
debug:
  enableHTTP: false                 # Enable HTTP request debugging
  httpFile: "debug-http.log"        # File to dump HTTP requests
  showBody: false                   # Include request body in dumps
```

### Service Configuration API and Restart

Set `service.serviceConfigEnabled: true` (`osctrl-api` only) to expose `/api/v1/service-config` and show the **Service Config** section in [osctrl-frontend](/components/osctrl-frontend/). Once enabled, operators can review and edit any of the sections above from the UI, persisted to the `service_config` table in the backend database.

* This switch only gates the read/edit/apply UI and API surface. Configuration is always resolved the same way at every boot: the YAML file's sections are seeded into `service_config`, and the stored rows are then resolved back over them — so the service always runs on the stored values, whether or not the API is enabled.
* Edited values can be written back to the on-disk YAML file, and a restart of `osctrl-tls` can be requested from `osctrl-api` (rate limited by `rateLimits.serviceConfigApply`) so it picks up the new values. The frontend shows a confirmation warning before requesting a restart, since it can disrupt in-flight osquery traffic.

### Example Configuration Files

#### osctrl-tls (tls.yaml)

```yaml
service:
  listener: "0.0.0.0"
  port: "9000"
  host: "tls.example.com"
  auth: "none"
  logLevel: "info"
  logFormat: "json"

db:
  type: "postgres"
  host: "127.0.0.1"
  port: "5432"
  name: "osctrl"
  username: "postgres"
  password: "postgres"
  sslmode: "disable"
  maxIdleConns: 20
  maxOpenConns: 100
  connMaxLifetime: 30
  connRetry: 5

redis:
  host: "127.0.0.1"
  port: "6379"
  password: ""
  db: 0
  connRetry: 5

rateLimits:
  enroll:
    burst: 20
    period: 1m
    evictAfter: 10m
    retryAfter: 60
    maxBuckets: 0

logger:
  type: "db"
  loggerDBSame: true
  alwaysLog: false

carver:
  type: "db"

tls:
  termination: false

osquery:
  version: "5.12.1"
  tablesFile: "data/5.12.1.json"
  logger: true
  config: true
  query: true
  carve: true
  accelerated: false
  console: false
  fileExplorer: false
  readOnly: false

metrics:
  enabled: true
  listener: "0.0.0.0"
  port: "9090"

osctrld:
  enabled: false

batchWriter:
  writerBatchSize: 50
  writerTimeout: 60s
  writerBufferSize: 2000

debug:
  enableHTTP: false
  httpFile: "debug-http-tls.log"
  showBody: false
```

### Command-Line Flags

All configuration values can be overridden using command-line flags. Use `--help` or `-h` to see all available flags for each service:

```bash
$ ./osctrl-tls --help
$ ./osctrl-api --help
```

Common flags:

* `--config` or `-c`: Enable configuration from YAML file
* `--config-file` or `-C`: Path to YAML configuration file
* `--db-host`: Database host
* `--db-port`: Database port
* `--redis-host`: Redis host
* `--listener` or `-l`: Service listener
* `--port` or `-p`: Service port
* `--service-config-enabled`: Expose the service configuration API/frontend (`osctrl-api` only)
* `--mfa-required`, `--mfa-issuer`, `--mfa-rpid`, `--mfa-origins`: MFA settings (`osctrl-api` only)
* `--osquery-console`: Enable the per-node interactive console
* `--rate-limit-<name>-burst`, `--rate-limit-<name>-period`, `--rate-limit-<name>-evict-after`, `--rate-limit-<name>-retry-after`, `--rate-limit-<name>-max-buckets`: Rate limit tuning, where `<name>` is `login`, `pre-auth`, `service-config-apply` (`osctrl-api`) or `enroll` (`osctrl-tls`)

### Environment Variables

Configuration values can also be set using environment variables. Each flag has a corresponding environment variable (see `--help` output for details).

Example:

```bash
export SERVICE_PORT="9000"
export DB_HOST="postgres.example.com"
export REDIS_HOST="redis.example.com"
```

### Migration from JSON to YAML

If you're upgrading from the old JSON-based configuration, you'll need to consolidate your separate JSON files into a single YAML file:

**Old (JSON)**:

- `tls.json` - Service settings
- `db.json` - Database settings
- `redis.json` - Redis settings
- `jwt.json` - JWT settings
- `saml.json` - SAML settings
- `logger_tls.json` - Logger settings
- `carver_tls.json` - Carver settings

**New (YAML)**:

- `tls.yaml` - All settings in one file

All configuration is now organized into sections within a single YAML file, making it easier to manage and version control your osctrl deployment configuration.
