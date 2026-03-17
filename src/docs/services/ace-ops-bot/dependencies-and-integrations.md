# Dependencies and integrations

## Runtime and libraries

- **Node.js** – application runtime.
- **@slack/bolt** (^4.2.1) – Slack app (events, auth, `chat.postMessage`); uses Express receiver for HTTP and custom routes.
- **express** (^4.21.2) – JSON body parsing and route mounting on Bolt’s receiver router.
- **ioredis** (^5.6.0) – Redis client for session storage.
- **dotenv** (^16.4.7) – Loads `.env` into `process.env`.
- **chalk** (4) – Console log formatting in Logger.

## External services (integrations)

| Service | Purpose | How |
|---------|---------|-----|
| **Slack** | Events (mention, message, DM) and posting messages | Bolt with `SLACK_BOT_TOKEN`, `SLACK_SIGNING_SECRET`; `chat.postMessage` for channel and thread. |
| **Redis** | User session storage (Slack user ID → `{ user, token }`) | ioredis; host from `REDIS_HOST`, key prefix from `SESSION_PREFIX`. |
| **ace-db-gateway** | Auth, project/channel/thread and KB data | HTTP with JWT from session. Base URL: `ACE_DB_GATEWAY_ENDPOINT`. Endpoints: login/external, channels, channel-thread/register, is-registered-message, documentations, users/projects, projects/knowledge-base, check user-project, check debug-project. |
| **ace-stack-backend** | LLM/omni processing and async reply | POST to `STACK_BACKEND_URL/api/omni/ingest` with message and callback URL; stack-backend calls back `SELF_URL/send-thread-message` to post the reply in the thread. |
| **ace-ops-scheduler** | Referenced for “add checker” (period/check) | `OPS_SCHEDULER_API` used in `addChecker` in customerMessages; this function is not currently invoked from the command flow. |

## Who uses ace-ops-bot

- **Users** – Interact via Slack (mention in channel, reply in thread, DM).
- **ace-stack-backend** – Calls ops-bot’s `POST /send-thread-message` to post LLM replies into the correct Slack thread.
- **Other internal callers** – Can post to a channel or thread via `POST /send-message` or `POST /send-thread-message` if they have network access to the service.

## Authentication and session

- Slack identity is established via Bolt (signing secret, bot token).
- ACE identity: ops-bot calls ace-db-gateway `POST /login/external` with `providerType: 'slack', providerId: slackUserId` to get a JWT and user data; these are stored in Redis keyed by Slack user ID with TTL.
- For channel/thread interactions, ops-bot also checks user–project association via ace-db-gateway (`check/user-project/slack-id/:channel/:user`). Threads are registered with origin `OPS` when the bot is mentioned in channel; only threads with origin OPS, PAYLOADS, or CRON are accepted for follow-up messages.
