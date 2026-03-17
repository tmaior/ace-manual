# Environment and configuration

## Required environment variables

| Variable | Purpose |
|----------|---------|
| **SLACK_BOT_TOKEN** | Slack Bot OAuth token. Required at startup. If it starts with `xoxb-mock-token`, the app runs in mock mode (no real Slack API calls, stub auth). |
| **SLACK_SIGNING_SECRET** | Slack signing secret for request verification (ExpressReceiver). Required at startup. |

The application exits on startup if either of these is missing (index.js and SlackAPI.js).

## Optional / deployment configuration

| Variable | Default | Purpose |
|----------|---------|---------|
| **PORT** | 3000 | HTTP server port. Bolt (ExpressReceiver) listens on this port for Slack events and for the custom routes (/send-message, /send-thread-message). |
| **REDIS_HOST** | (from options; not set in index) | Redis host. Passed to SecSession/RedisCommon. If omitted, RedisCommon uses `localhost`. |
| **SESSION_PREFIX** | `session:bots:slack:` | Redis key prefix for session keys. |
| **ACE_DB_GATEWAY_ENDPOINT** | `http://ace-db-gateway-service` | Base URL for ace-db-gateway (login, channels, channel-thread, documentations, projects, knowledge-base). |
| **AI_API_URL** | (none) | Base URL for the AI/LLM chat API. Full URL used is `${AI_API_URL}/api/chat/`. Must be set for sendToLLM to work; otherwise the request will fail. |

## Secrets and security

- **SLACK_BOT_TOKEN** and **SLACK_SIGNING_SECRET** are sensitive; they must be provided via a secure mechanism (e.g. Kubernetes secrets, GitHub Environments) and must not be committed.
- JWT tokens from ace-db-gateway are stored in Redis; ensure Redis is not exposed publicly and access is restricted.
- The HTTP routes (/send-message, /send-thread-message) do not implement authentication in the scanned code; only the Slack event path verifies requests via the signing secret. If the bot is exposed to the network, consider protecting these endpoints (e.g. internal only or API key).

## Per-environment notes

- **Local**: Set REDIS_HOST if Redis is not on localhost. Use mock token for offline testing. Point ACE_DB_GATEWAY_ENDPOINT and AI_API_URL to local or dev instances.
- **EKS/Production**: Typically PORT is set by the platform. REDIS_HOST and ACE_DB_GATEWAY_ENDPOINT should point to cluster services. AI_API_URL must point to the deployed AI chat service.
