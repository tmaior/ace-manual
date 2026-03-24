# Environment and configuration

All configuration is via environment variables. No config files are required beyond `.env` (or env injected at runtime).

## Required

| Variable | Purpose |
|----------|---------|
| **SLACK_BOT_TOKEN** | Slack Bot OAuth token. Must be set at startup; process exits if missing. For local/mock use, `xoxb-mock-token` enables mock mode (no real Slack calls). |
| **SLACK_SIGNING_SECRET** | Slack signing secret for request verification. Required at startup. |

## Optional (with defaults)

| Variable | Default | Purpose |
|----------|---------|---------|
| **PORT** | `3000` | HTTP port for Bolt app and Express routes. |
| **ACE_DB_GATEWAY_ENDPOINT** | `http://ace-db-gateway-service` | Base URL for ace-db-gateway (login, channels, threads, projects, model, knowledge base, debug flag). |
| **REDIS_HOST** | (none; RedisCommon default host is `localhost`) | Redis host for DevSession/RedisCommon. Passed into `DevSession({ host: process.env.REDIS_HOST, ... })`. |
| **SESSION_PREFIX** | `session:bots:slack:` | Redis key prefix for user sessions. |
| **STACK_BACKEND_URL** | `http://web-backend-service` | Base URL for ace-stack-backend; used for `POST /api/omni/ingest`. |
| **SELF_URL** | `http://ace-slackbot-service` | Base URL of this service; used to build callback URL for ingest (e.g. `{SELF_URL}/send-thread-message`). |
| **DEBUG_TRUE_FIXED** | (unset) | If set (non-null/undefined), forces debug mode on for all channels (debug-only messages are not filtered in `/send-thread-message`). |

## Optional (no default in code)

| Variable | Purpose |
|----------|---------|
| **DOCS_API_URL** | Used by `saveDoc` in customerMessages (POST to `/api/documentation`). Omit if this flow is not used. |
| **REDIS_PORT** | Not passed from current code into DevSession; RedisCommon defaults to 6379 if not provided in options. |

## Startup behavior

- **index.js** checks for `SLACK_BOT_TOKEN` and `SLACK_SIGNING_SECRET`; if any is missing, it logs and exits with `process.exit(1)`.
- **SlackAPI** constructor also validates these two and exits if missing. It then creates ExpressReceiver and Bolt App; in mock mode it uses a custom `authorize` that returns fixed team/bot IDs.

## Secrets

- **SLACK_BOT_TOKEN** and **SLACK_SIGNING_SECRET** must be kept secret and provided via a secure mechanism (e.g. Kubernetes secrets, GitHub Environments). No secrets are read from config files in the repo.
