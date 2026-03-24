# Architecture and message flow

## Process and startup

1. **index.js**: Load dotenv, validate SLACK_BOT_TOKEN and SLACK_SIGNING_SECRET, instantiate SlackAPI (which creates Bolt App and ExpressReceiver, initializes bot info via auth.test).
2. Register handlers: `slackAPI.mention(mentionMessage)`, `slackAPI.message(mentionMessage)` (for thread messages), `slackAPI.dm(dmMessage)`.
3. Apply routes: `routing(slackAPI)` mounts POST /send-message and POST /send-thread-message on `slackAPI.receiver.router`.
4. Start: `slackAPI.start()` starts the app on PORT (default 3000). Slack sends events to this server; the same server serves the custom HTTP routes.

## Mention flow (channel, no thread)

1. User mentions the bot in a channel (no thread_ts). Slack sends `app_mention`.
2. **SlackAPI**: authUser(event.user, event.channel) — get or create session (Redis or DB Gateway login), then check project association via DB Gateway. If not associated or not registered, send error message and return.
3. **SlackAPI**: Create DBGateway with auth token; registerMessage(channel, ts) with origin SEC.
4. **customerMessages.mentionMessage**: Build obj (channelID, userID, message without bot mention, say, client, token, thread_ts). checkCommands(obj): if "reset session" or "get docs", run and return; otherwise continue.
5. **sendToLLM**: Resolve session (or auto-login), get projectId (from channel or DM from user’s projects), resolve conversa_id (from isRegisteredMessage or synthetic), get knowledge_base_id (Redis cache or DB Gateway), build request with origin "ace_sec", POST to AI_API_URL/api/chat/, then say(response message).

## Thread message flow

1. User posts in a thread where the bot was previously mentioned. Slack sends `message` with thread_ts.
2. **SlackAPI**: authUser; then DBGateway.isRegisteredMessage(channel, thread_ts). If !register.ok or !register.registered, return (no reply). Otherwise call customerMessages.mentionMessage (same handler as mention) with the event; sendToLLM uses thread_ts for conversa_id when applicable.

## DM flow

1. User sends a DM. Slack sends `message` with channel_type === 'im'.
2. **SlackAPI.dm**: Calls dmMessage with (args, sendErrorMessage). No project association check; channelID is set to 'DM'.
3. **customerMessages.dmMessage**: checkCommands (reset session, get docs); if not handled, sendToLLM. In sendToLLM, projectId is taken from getProjectsByUser (first project); if none, user is told they are not associated with any project.

## HTTP route flow

- **POST /send-message**: Validate body (channel, message). slackAPI.messageToChannel(channel, message). Return JSON with ok, ts, channel or error.
- **POST /send-thread-message**: Validate body (channelID, ts, message). slackAPI.messageToThread(channelID, ts, message). Return JSON with ok, ts, channel or error.

## Session and Redis

- **SecSession** wraps **RedisCommon**. Keys are `{SESSION_PREFIX}{slackUserId}`. Values are JSON: { user, token } (and possibly userInfo). TTL from login (tokenEXP - now - 100 seconds) or default in RedisCommon (12 hours).
- **Knowledge Base cache**: Key `{projectId}:KnowledgeBase`, value knowledgeBaseId or "null"; TTL 86400 seconds. Set after getKnowledgeBaseIdForProject(projectId).

## DB Gateway endpoints used

| Method | Endpoint | Use |
|--------|----------|-----|
| POST | /login/external | Slack login (providerType: slack, providerId: slackUserId). |
| GET | /check/user-project/slack-id/:channel/:user | User allowed in channel’s project. |
| GET | /api/channels?slackId=... (or project) | Project by channel; channel list by project. |
| POST | /api/channel-thread/register | Register thread (slackId, ts, origin SEC). |
| GET | /api/channel-thread/is-registered-message/:channelSlackId/:ts/SEC | Thread registered and conversaId. |
| GET | /api/documentations?projectId=...&identifier=policy | Policy docs for get docs. |
| GET | /api/users/projects?slackId=... | User’s projects (DM flow). |
| GET | /api/projects/:projectId/knowledge-base | Knowledge Base ID for project. |

## Project structure (code-based)

```
ace-sec-bot/
├── index.js                 # Entry: env check, SlackAPI, handlers, routes, start
├── routes/
│   └── index.js             # POST /send-message, POST /send-thread-message
└── controller/
    ├── SlackAPI.js          # Bolt App, ExpressReceiver, auth, mention/message/dm, messageToChannel/Thread
    ├── customerMessages.js  # mentionMessage, dmMessage, checkCommands, sendToLLM, getDocs, resetSession
    ├── DBGateway.js         # HTTP client for ace-db-gateway
    ├── SecSession.js        # Session API over Redis
    ├── RedisCommon.js       # ioredis session and key operations
    └── Logger.js            # Structured logger (level, context, message)
```
