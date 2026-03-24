# Dependencies and integrations

## External dependencies

| Dependency | Role |
|------------|------|
| **Slack** | Bot receives events (app_mention, message) and sends messages via Slack API. Requires SLACK_BOT_TOKEN and SLACK_SIGNING_SECRET. ExpressReceiver provides the HTTP endpoint for Slack to deliver events. |
| **Redis** | Session storage (user session keyed by Slack user ID) and cache for Knowledge Base ID per project (key `{projectId}:KnowledgeBase`, TTL 24h). Connection via REDIS_HOST (default not set; code uses host from options). Port 6379 by default. |
| **ace-db-gateway** | Authentication (Slack login), channel/project/user resolution, channel-thread registration, policy docs, Knowledge Base ID. Base URL from ACE_DB_GATEWAY_ENDPOINT. |
| **AI/LLM API** | Chat endpoint at AI_API_URL (e.g. `${AI_API_URL}/api/chat/`). Receives message, token, channel, user, project, client, conversa_id, knowledge_base_id, origin ace_sec; returns message to post in Slack. |

## Main libraries

- **@slack/bolt**: Slack App and ExpressReceiver; event handling and chat.postMessage.
- **express**: JSON body parsing on receiver.router (used by HTTP routes and Bolt).
- **ioredis**: Redis client used by RedisCommon/SecSession.
- **dotenv**: Load env from .env.
- **chalk**: Logger output coloring (Logger.js).

## Who uses ace-sec-bot

- **Users**: Interact via Slack (mention in channel or DM).
- **Other ACE services**: Can send messages or thread replies by calling the bot’s HTTP endpoints (POST /send-message, POST /send-thread-message) if they have network access to the bot’s receiver (same port as Bolt).

## Integration pattern

- **Auth**: User is identified by Slack user ID. First time (or when session missing), bot calls ace-db-gateway `/login/external` with providerType `slack` and providerId (Slack user ID). JWT and user info are stored in Redis with TTL. For channel messages, bot also checks user–project association via ace-db-gateway `/check/user-project/slack-id/{channel}/{user}`.
- **Channel threads**: Before processing a mention, bot registers the message with ace-db-gateway (`POST /api/channel-thread/register` with slackId, ts, origin `SEC`). For follow-up messages in thread, bot checks `GET /api/channel-thread/is-registered-message/{channelSlackId}/{ts}/SEC` and uses existing conversaId when available.
- **AI chat**: Bot builds a payload with projectId, clientId, knowledge_base_id (from DB Gateway or Redis cache), conversa_id (from thread registration or synthetic), and sends it to the AI API; response message is posted back to the channel or DM.
