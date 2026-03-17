# ace-sec-bot

Node.js Slack bot service for ACE security-related interactions. Uses Slack Bolt, Redis for sessions, **ace-db-gateway** for authentication and project/channel data, and an external **AI/LLM API** for chat. Other ACE services can post messages or thread replies via HTTP endpoints on the same server.

## Purpose

- **Slack security bot**: Handle @mentions and DMs; authenticate users and check project association; register channel threads; send messages to the AI API and post replies.
- **Session management**: Store user sessions (JWT and user info) in Redis; support "reset session" and auto-login.
- **HTTP API**: POST /send-message and POST /send-thread-message for other services to send messages or thread replies to Slack.

## Documentation in this folder

| Document | Description |
|----------|-------------|
| [overview.md](./overview.md) | What the app is, architecture, main components, capabilities. |
| [dependencies-and-integrations.md](./dependencies-and-integrations.md) | Slack, Redis, ace-db-gateway, AI API; libraries; integration pattern. |
| [features-and-capabilities.md](./features-and-capabilities.md) | Event handling, built-in commands, HTTP routes, session, limitations. |
| [environment-and-configuration.md](./environment-and-configuration.md) | Required/optional env vars, defaults, secrets, per-environment. |
| [architecture-and-message-flow.md](./architecture-and-message-flow.md) | Startup, mention/thread/DM flows, HTTP routes, Redis, DB Gateway usage, project structure. |

## Repository

- **ace-sec-bot/** – Application code (index.js, routes/, controller/). Use the code as the source of truth; repo `docs/` may be outdated.

## Related

- [Architecture service catalog](../../architecture/service-catalog.md)
- [Security rules](../../rules/security-rules.md)
