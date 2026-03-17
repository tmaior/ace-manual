# ace-slackbot

Slack bot service for ACE. Node.js, @slack/bolt, Redis for sessions. Handles Slack events and commands, integrates with ace-db-gateway and ace-stack-backend, and exposes HTTP endpoints for posting messages and thread replies to Slack.

## Purpose

- Receive Slack events: app mentions, messages in threads, and DMs.
- Authenticate users via ace-db-gateway (login, project association); store session in Redis.
- Send user messages to ace-stack-backend (`/api/omni/ingest`); backend calls back to slackbot to post replies in threads.
- Expose HTTP routes for health, send-message, and send-thread-message so other services (e.g. ace-commands-api) can deliver content to Slack.

## Documentation in this folder

| Document | Description |
|----------|-------------|
| [overview.md](./overview.md) | What the app is, architecture, main components, and flows. |
| [dependencies-and-integrations.md](./dependencies-and-integrations.md) | Slack, Redis, ace-db-gateway, ace-stack-backend; who calls slackbot. |
| [features-and-capabilities.md](./features-and-capabilities.md) | Events, commands, HTTP endpoints, auth, debug and mock mode. |
| [environment-and-configuration.md](./environment-and-configuration.md) | Required/optional env vars, defaults, secrets. |

## How to use

- **New to this service**: Start with [START_HERE.md](./START_HERE.md), then read [overview.md](./overview.md).
- **Setting up or deploying**: Use [environment-and-configuration.md](./environment-and-configuration.md) and the repository’s setup instructions (e.g. Docker, K8s in ace-infra).
- **Integrating with slackbot**: See [dependencies-and-integrations.md](./dependencies-and-integrations.md) and [features-and-capabilities.md](./features-and-capabilities.md) for HTTP API and callback contract.

## Related

- [Architecture service catalog](../../architecture/service-catalog.md)
- [Security rules](../../rules/security-rules.md) (rate limiting, auth)
- [Environments](../../environments/) for local, demo, production
