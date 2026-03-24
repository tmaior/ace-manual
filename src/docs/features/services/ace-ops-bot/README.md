# ace-ops-bot

Operations bot service for ACE. Node.js, Slack API (Bolt), Redis for sessions. Integrates with ace-db-gateway (auth, project/channel/thread, knowledge-base) and ace-stack-backend (LLM/omni ingest). Handles channel mentions, threaded replies, and DMs; exposes HTTP endpoints so other services can post messages or receive callbacks (e.g. LLM replies into a thread).

## Purpose

- Provide an operations-focused Slack bot: answer in channels (mention or registered threads) and in DMs.
- Authenticate users via ace-db-gateway and keep sessions in Redis.
- Register threads with origin OPS and accept replies in threads from OPS, PAYLOADS, or CRON.
- Send user messages to stack-backend omni ingest and receive LLM responses via callback to post in the same Slack thread.

## Documentation in this folder

| Document | Description |
|----------|-------------|
| [overview.md](./overview.md) | What the app is, architecture, main components, flow. |
| [dependencies-and-integrations.md](./dependencies-and-integrations.md) | Runtime and npm deps; Slack, Redis, ace-db-gateway, ace-stack-backend. |
| [features-and-capabilities.md](./features-and-capabilities.md) | Event handling, commands, LLM integration, thread filtering, mock mode, endpoints. |
| [environment-and-configuration.md](./environment-and-configuration.md) | Required/optional env vars, defaults, secrets. |
| [express-routes-and-callbacks.md](./express-routes-and-callbacks.md) | POST /send-message, POST /send-thread-message, callback flow. |

## How to use

- **New to the service**: Start with [START_HERE.md](./START_HERE.md), then [overview.md](./overview.md).
- **Integrating with ops-bot**: See [dependencies-and-integrations.md](./dependencies-and-integrations.md) and [express-routes-and-callbacks.md](./express-routes-and-callbacks.md).
- **Config / deploy**: See [environment-and-configuration.md](./environment-and-configuration.md).

## Related

- [Architecture service catalog](../../architecture/service-catalog.md)
- [Security rules](../../rules/security-rules.md)
- **Repository**: `ace-ops-bot/` – application code; use the code as the source of truth; repo `docs/` may be outdated.
