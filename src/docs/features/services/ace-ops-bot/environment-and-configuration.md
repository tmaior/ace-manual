# Environment and configuration

All configuration is via environment variables (e.g. `.env` locally; in Kubernetes or Docker, set in deployment or compose). No config file is loaded besides `.env` (dotenv).

## Required

| Variable | Description |
|----------|-------------|
| **SLACK_BOT_TOKEN** | Slack Bot OAuth token (starts with `xoxb-`). If it starts with `xoxb-mock-token`, the app runs in mock mode (no real Slack API calls). |
| **SLACK_SIGNING_SECRET** | Slack signing secret for request verification (Bolt ExpressReceiver). |

The application exits at startup if either of these is missing.

## Optional (with defaults)

| Variable | Default | Description |
|----------|---------|-------------|
| **PORT** | `3000` | HTTP port for the Bolt/Express server. |
| **REDIS_HOST** | (none; RedisCommon uses `localhost` if host not set) | Redis host for session storage. OpsSession is constructed with `host: process.env.REDIS_HOST` (no default in code; RedisCommon defaults to `localhost`). |
| **SESSION_PREFIX** | `session:bots:slack:` | Redis key prefix for session keys (Slack user ID). |
| **ACE_DB_GATEWAY_ENDPOINT** | `http://ace-db-gateway-service` | Base URL for ace-db-gateway (login, channels, channel-thread, documentations, projects, knowledge-base, checks). |
| **STACK_BACKEND_URL** | `http://web-backend-service` | Base URL for ace-stack-backend (used for `POST /api/omni/ingest`). |
| **SELF_URL** | `http://ace-ops-bot-service` | Public/base URL of this service; used as callback base for `callback_url` in omni ingest (e.g. `{SELF_URL}/send-thread-message`). |
| **OPS_SCHEDULER_API** | `http://ops-scheduler-service` | Base URL for ace-ops-scheduler; referenced in `addChecker` (customerMessages), which is not currently wired into the command flow. |

## Redis port

Redis port is not read from env in the code; `RedisCommon` uses default `options?.port || 6379`. To change it, the app would need to be extended to pass `process.env.REDIS_PORT` (or similar) into the Redis client options.

## Secrets

- **SLACK_BOT_TOKEN** and **SLACK_SIGNING_SECRET** must be kept secret; they grant access to the Slack app and allow forging requests if leaked.
- JWT from ace-db-gateway is stored in Redis (session); ensure Redis is not exposed and access is restricted.
- In production, use a secrets manager or cluster secrets; do not commit `.env` or real tokens.

## Per-environment notes

- **Local**: Set `ACE_DB_GATEWAY_ENDPOINT`, `STACK_BACKEND_URL`, `SELF_URL` to local or tunnel URLs if other services run on host or in Docker. Use `xoxb-mock-token` for Slack token to run without a real Slack app.
- **Kubernetes/Docker**: Typically `ACE_DB_GATEWAY_ENDPOINT`, `STACK_BACKEND_URL`, and `SELF_URL` point to in-cluster service names or ingress; `REDIS_HOST` to the Redis service.
