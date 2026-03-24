# ace-sec-bot – Overview

## What it is

**ace-sec-bot** is the ACE security bot service. It is a Node.js application that runs as a Slack bot using the Slack Bolt framework. It handles app mentions and direct messages (DMs), authenticates users via **ace-db-gateway**, stores sessions in **Redis**, and sends conversation payloads to an external **AI/LLM API** for chat responses. It also exposes HTTP endpoints so other services can post messages or thread replies to Slack channels.

## Purpose

- **Slack security bot**: Respond to @mentions in channels and to DMs with AI-powered replies, after validating that the user is registered and associated with the project (channel).
- **Session management**: Keep user sessions (JWT and user info) in Redis with configurable TTL; support "reset session" and auto-login when session expires.
- **Channel and thread registration**: Register channel threads with ace-db-gateway (origin `SEC`) so conversations are tracked and linked to projects.
- **HTTP API for other services**: Allow posting messages to a channel or to a thread via `POST /send-message` and `POST /send-thread-message` on the same Express router used by Bolt.

## High-level architecture

- **Slack**: @slack/bolt (App + ExpressReceiver). Events: `app_mention`, `message` (for threads and DMs). Bot token and signing secret from environment.
- **Redis**: ioredis. Session storage with prefix (e.g. `session:bots:slack:`). Also used for caching Knowledge Base ID per project (24h TTL).
- **ace-db-gateway**: Login (`/login/external` with Slack provider), channel/project resolution, user–project check, channel-thread register/is-registered, documentations (policy docs), projects by user, Knowledge Base ID per project. All requests use JWT from session or login.
- **AI API**: External URL from `AI_API_URL`. POST to `/api/chat/` with message, token, channel, user, project, client, `conversa_id`, `knowledge_base_id`, `origin: "ace_sec"`. Response message is sent back to Slack.

## Main components

| Component | Role |
|-----------|------|
| **index.js** | Entry point: loads dotenv, checks required env (SLACK_BOT_TOKEN, SLACK_SIGNING_SECRET), creates SlackAPI, wires mention/DM handlers and routes, starts app. |
| **controller/SlackAPI.js** | Bolt App + ExpressReceiver, auth via session or DB Gateway login, project association check, registerMessage/isRegisteredMessage, messageToChannel/messageToThread, mention and message (thread) and DM handlers. |
| **controller/customerMessages.js** | Handlers: removeBotMention, resetSession, getDocs, checkCommands (reset session, get docs), sendToLLM (build request with conversa_id, projectId, knowledge_base_id; call AI API; return message). mentionMessage and dmMessage. |
| **controller/DBGateway.js** | HTTP client for ace-db-gateway: get/post/delete, slackLogin, getProjectByChannel, getUserIdBySlackId, getPolicyDocs, checkProjectUsersSlackIds, registerMessage, isRegisteredMessage, getProjectsByUser, getChannelsByProject, getKnowledgeBaseIdForProject. |
| **controller/SecSession.js** | Session wrapper over Redis: startSession, getSession, deleteUserSession, updateSession, restartSession, getOrStartSession, sessionExists. Uses RedisCommon. |
| **controller/RedisCommon.js** | ioredis wrapper: getSession, saveSession, deleteSession, getExp, overrideSession, restartSession, sessionExists. Prefix and TTL configurable. |
| **controller/Logger.js** | Structured console logger (level, context, message) with chalk. Exported as single function. |
| **routes/index.js** | Mounts on SlackAPI.receiver.router: POST /send-message (channel, message), POST /send-thread-message (channelID, ts, message). |

## What you can do with this app

- Run the security bot locally or in EKS with Slack credentials, Redis, and env pointing to ace-db-gateway and AI API.
- Let users mention the bot in a channel or send DMs; get AI replies after auth and project association check.
- Use "reset session" and "get docs" (policy docs for the channel’s project) as built-in commands.
- Call the HTTP endpoints from other ACE services to send messages or thread replies into Slack.

## Related documentation

- [Dependencies and integrations](./dependencies-and-integrations.md)
- [Features and capabilities](./features-and-capabilities.md)
- [Environment and configuration](./environment-and-configuration.md)
- [Architecture and message flow](./architecture-and-message-flow.md)
- Repository: `ace-sec-bot/` (sibling to ace-manual). Prefer reading the code; repo `docs/` may be outdated.
- Architecture: [service-catalog](../../architecture/service-catalog.md).
