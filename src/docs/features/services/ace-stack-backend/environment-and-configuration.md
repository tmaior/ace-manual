# Environment and configuration

## Bootstrap behaviour

- **NODE_ENV**: `development`, `test`, or other (e.g. `staging`, `production`). In `development` or `test`, dotenv loads `.env.development` or `.env.test`. Otherwise, secrets are loaded from AWS Secrets Manager (see below).
- **AWS (non-dev/non-test)**: If `AWS_SECRET_NAME` and `AWS_REGION` are set, `main.ts` calls `fetchSecrets(secretName, region)` and merges the returned key-value map into `process.env`. All other env vars (JWT, DB, REDIS, OAuth, etc.) can be provided via that secret or by the runtime environment.
- **ConfigModule**: `ConfigModule.forRoot({ isGlobal: true, ignoreEnvFile: true })` so Nest uses `process.env` only (env files are loaded in `main.ts` before the app is created).

## Required / common environment variables

| Variable | Purpose |
|----------|---------|
| **PORT** | HTTP server port (default 3000). |
| **JWT_SECRET** | Secret used to sign and verify JWT; required for auth. |
| **ACE_GATEWAY_URL** | Base URL for ace-db-gateway (e.g. `http://ace-db-gateway-service`). |
| **REDIS_URL** | Redis connection URL (e.g. `redis://redis-service:6379`, `redis://localhost:6379`) for WebSocket adapter and OAuth state. |
| **FRONTEND_HOST** | Allowed CORS origin and base for OAuth redirects (e.g. login success, provider callback). |
| **API_HOST** | Backend base URL used in OAuth redirect URIs (e.g. Google redirect_uri, GitHub callback). |

## Database (TypeORM)

| Variable | Purpose |
|----------|---------|
| **DB_HOST** | PostgreSQL host. |
| **DB_PORT** | PostgreSQL port (default 5432 in code). |
| **DB_USER** | Database user. |
| **DB_PASSWORD** | Database password. |
| **DB_NAME** | Database name. |
| **DB_SSL_ENABLED** | Set to `true` to enable SSL. |
| **DB_SSL_REJECT_UNAUTHORIZED** | Set to `false` to allow self-signed or non-strict SSL (e.g. dev). |
| **DB_SSL_CA** | Optional CA certificate for SSL. |

## Auth (OAuth and JWT)

| Variable | Purpose |
|----------|---------|
| **JWT_EXPIRES_IN** | JWT expiry (default `24h`). |
| **GOOGLE_CLIENT_ID** | Google OAuth client ID. |
| **GOOGLE_CLIENT_SECRET** | Google OAuth client secret. |
| **GITHUB_CLIENT_ID** | GitHub OAuth app client ID. |
| **GITHUB_CLIENT_SECRET** | GitHub OAuth app client secret (used by github-integrations service for token exchange). |

## AWS (when not using dotenv)

| Variable | Purpose |
|----------|---------|
| **AWS_SECRET_NAME** | Name of the secret in AWS Secrets Manager (non-dev/non-test). |
| **AWS_REGION** | AWS region for Secrets Manager (and SES, etc.). |
| **SES_FROM_EMAIL** | Sender email for SES (notification module). |
| **AWS_ACCOUNT_ID** | Optional; used by knowledge-base provisioning when needed. |

## Optional / fallbacks

| Variable | Purpose | Default / behaviour |
|----------|---------|---------------------|
| **FRONTEND_URL** | Alternative base for redirects when different from FRONTEND_HOST. | Fallback to FRONTEND_HOST or `http://web-frontend-service`. |
| **LLM_URL** | LLM service base URL for chat. | `http://llm-service`. |
| **SLACK_BOT_TOKEN** | Slack API token for slack service. | Required only when using Slack. |
| **NODE_ENV** | Environment mode. | `development`; controls dotenv vs AWS secrets. |

## Environment-specific notes

- **development / test**: Use `.env.development` or `.env.test`; no AWS Secrets Manager call. Local Redis and DB typical.
- **staging / production**: Typically use AWS Secrets Manager; ensure ACE_GATEWAY_URL, REDIS_URL, DB_*, JWT_SECRET, OAuth credentials, FRONTEND_HOST, API_HOST, and SES vars are set (in secret or env).
- **CORS**: Allowed origins are FRONTEND_HOST and a fixed set of localhost URLs; other origins are blocked.

## Paths and files

- **Migrations**: `src/migrations/`; datasource `src/shared/database/datasource.ts` (uses dotenv for the env file by NODE_ENV when running CLI).
- **Entities**: TypeORM entities under `src/**/*.entity.ts` (e.g. user, user-provider, password-reset-token, email-verification-token).

## Secrets (EKS / production)

- Store sensitive values in AWS Secrets Manager and inject via `AWS_SECRET_NAME` at runtime, or via Kubernetes Secrets / deployment env. Do not commit secrets; use env or secret injection for JWT_SECRET, DB_*, OAuth client secrets, REDIS_URL, and SES_FROM_EMAIL.
