# Dependencies and integrations

## External dependencies

- **Slack**: @slack/bolt (App, ExpressReceiver). Requires `SLACK_BOT_TOKEN` and `SLACK_SIGNING_SECRET`. Events are received via Bolt’s HTTP endpoints (signature verification). Outbound: `chat.postMessage`, `files.uploadV2` for channel/thread messages and file uploads.
- **Redis**: ioredis. Used for user sessions (key: `SESSION_PREFIX` + Slack user ID) and channel-level model preference (`model:channel:{channelID}`). Configured via `REDIS_HOST`; optional `SESSION_PREFIX` (default `session:bots:slack:`). Port defaults to 6379 in RedisCommon when not provided.
- **ace-db-gateway**: HTTP. Used for login, project/channel resolution, thread registration, user–project checks, discovery docs, model get/save, knowledge-base ID, and debug-project flag. Base URL: `ACE_DB_GATEWAY_ENDPOINT` (default `http://ace-db-gateway-service`). No JWT for some check endpoints; JWT from session for authenticated calls.
- **ace-stack-backend**: HTTP. Single integration point: `POST /api/omni/ingest` with message, channel, user, project, optional model and knowledge base; callback URL is slackbot’s `SELF_URL/send-thread-message` for async replies in thread. Base URL: `STACK_BACKEND_URL` (default `http://web-backend-service`).
- **DOCS_API_URL** (optional): Used only by the “get docs” / save-doc flow for `customerMessages.saveDoc` (POST to `/api/documentation`). If not used in your deployment, this integration may be deprecated or unused.

## Main libraries (from code)

- **@slack/bolt** – Slack app server and client.
- **express** – Used by Bolt’s ExpressReceiver for HTTP routes.
- **ioredis** – Redis client for session and cache.
- **dotenv** – Env loading.
- **chalk** – Console log formatting (Logger).
- **@aws-sdk/client-sqs** – Present in package.json but not used in the current codebase for producing or consuming SQS messages; slackbot does not implement queue producers or consumers.

## Who calls ace-slackbot

- **Slack**: Events (app_mention, message) and user interactions.
- **ace-stack-backend**: Calls slackbot `POST /send-thread-message` (and possibly `/send-message`) to post LLM/output into Slack channels or threads. Callback URL is built from `SELF_URL`.
- **ace-commands-api** (or other services): Can call `POST /send-message` and `POST /send-thread-message` to deliver command output or notifications; typically with internal auth/network only.

## Integration pattern

- **Inbound**: Slack → Bolt (events) and HTTP → Express routes (`/health`, `/send-message`, `/send-thread-message`).
- **Outbound**: Slackbot → DB Gateway (REST), Slackbot → Stack Backend (ingest), Slackbot → Redis (sessions/cache). No outbound SQS in the current code.
