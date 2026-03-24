# Environment and configuration

## Required / common environment variables

| Variable | Purpose |
|----------|---------|
| **NODE_ENV** | `development`, `local`, `demo`, or `production`; selects DB config in `config/cliConnection.js` when `DATABASE_URL` is not set. |
| **PORT** | HTTP server port when run as server (default 3000). |
| **DATABASE_URL** | Full PostgreSQL connection string. When set, overrides per-env config and is used by `config/db.js`. |
| **DB_USERNAME** | Database user (when not using DATABASE_URL). |
| **DB_PASSWORD** | Database password. |
| **DB_DATABASE** | Database name (e.g. `configuration`, `ace_configuration`, `ace_configuration_demo` for demo). |
| **DB_HOST** | Database host. |
| **DB_PORT** | Database port (default 5432 for local). |
| **DB_DIALECT** | `postgres` (typical). |

## Optional environment variables

| Variable | Purpose | Default / behaviour |
|----------|---------|---------------------|
| **DB_LOGGING** | Enable Sequelize SQL logging. | `false` in production config. |
| **DB_USE_SSL** | Use SSL for DB connection (e.g. RDS/DocumentDB). | `false` for local. |
| **DB_SSL_STRICT** | Reject unauthorized SSL certificates. | Often `false` for dev/demo. |
| **SLACK_BOT_TOKEN**, **SLACK_SIGNING_SECRET** | Referenced in code for optional Slack integration; not required for schema/seed or health. | Unset. |

## Environment-specific behavior

- **local**: `config/cliConnection.js` uses `DB_HOST=127.0.0.1`, `DB_DATABASE=configuration`, no SSL by default.
- **development / demo / production**: Use SSL when connecting to RDS/DocumentDB; certificate path may be `us-east-1-bundle.pem` in project root (see `config/db.js`).
- **demo**: Often uses database name `ace_configuration_demo` on the same RDS as production (see [demo environment](../../environments/demo.md)).

## Paths and files

- **Certificate**: `us-east-1-bundle.pem` in repo root for TLS (when `DB_USE_SSL=true` and not using default system certs).
- **Sequelize**: `.sequelizerc` points CLI to `config`, `migrations`, `models`, `seeders`.

## Secrets (EKS)

- Secrets are stored in AWS Secrets Manager at **ace/&lt;env&gt;/configuration-secrets** and injected as env (e.g. via workflow-generated Kubernetes Secret). Include `DATABASE_URL` or `DB_*` and any other required vars; never commit secrets.
