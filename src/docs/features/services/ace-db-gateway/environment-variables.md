# Environment variables

The following environment variables are used by ace-db-gateway (derived from the application code). Production and staging should set all required and security-sensitive variables explicitly; avoid relying on in-code defaults for secrets.

## Server

| Variable | Required | Description |
|----------|----------|-------------|
| `PORT` | No | HTTP server port. Default: `3000`. |
| `NODE_ENV` | No | When `production`, error responses hide detailed messages. |

## Database

| Variable | Required | Description |
|----------|----------|-------------|
| `CONFIGURATION_DB_URL` | Yes | PostgreSQL connection URL for the configuration database. Used by Sequelize in `config/database.js`. |
| `DB_USE_SSL` | No | If `true` (case-insensitive), enables SSL for the DB connection (e.g. RDS with `us-east-1-bundle.pem`). |

## Authentication and session

| Variable | Required | Description |
|----------|----------|-------------|
| `JWT_SECRET` | Yes (prod) | Secret used to sign and verify JWT. Code fallback exists for dev only; must be set in production. |
| `SESSION_EXPIRATION_SECONDS` | No | JWT expiration; can be a number of seconds or a string like `12h`. Default: `12h`. |
| `SESSION_SECRET` | No | Used for session signing in Redis. If unset, a random value is generated (not suitable for multi-instance). |
| `REDIS_HOST` | No | Redis host for session storage. Default: `redis-service`. |
| `REDIS_PORT` | No | Redis port. Default: `6379` (parsed as integer). |

## Internal service tokens

Used by ace-stack-backend, ace-commands-api, and ace-ops-scheduler to call the DB gateway without user JWT. Production should set these to unique secrets.

| Variable | Required | Description |
|----------|----------|-------------|
| `BACKEND_DB_GATEWAY_TOKEN` | No | Token for backend; code default exists for dev. |
| `COMMANDS_API_DB_GATEWAY_TOKEN` | No | Token for commands-api. |
| `SCHEDULLER_DB_GATEWAY_TOKEN` | No | Token for ops-scheduler (note: typo “scheduller” in code). |

## AWS and secrets

| Variable | Required | Description |
|----------|----------|-------------|
| `SECRETS_REGION` | No | AWS region for Secrets Manager. Default: `us-east-1`. |
| `SECRET_PATH_PREFIX` | No | Prefix for secret names in Secrets Manager. Default: `/ace/dev/secrets/`. |
| `ENCRYPTION_KEY` | When using encryption | Key used in `utils/encryption.js` for encrypting/decrypting data. |

## Summary

- **Required for running**: `CONFIGURATION_DB_URL`. For production, also set `JWT_SECRET` and avoid relying on default internal tokens.
- **Recommended**: Set `REDIS_HOST` and `REDIS_PORT` (and `SESSION_SECRET` if multiple instances) for session storage; set `SECRETS_REGION` and `SECRET_PATH_PREFIX` when using AWS Secrets Manager; set `ENCRYPTION_KEY` if encryption is used.
