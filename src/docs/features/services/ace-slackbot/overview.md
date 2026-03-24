# ace-slackbot – Overview

## What it is

**ace-slackbot** is the ACE Slack bot service. It runs a Node.js application built on **@slack/bolt** (ExpressReceiver). It receives Slack events (app mentions, channel messages in threads, DMs), authenticates users via **ace-db-gateway**, keeps session state in **Redis**, and sends user messages to **ace-stack-backend** for LLM processing. It also exposes HTTP endpoints so other services (e.g. ace-commands-api) can post messages or thread replies into Slack channels.

## Purpose

- **Slack event handling**: React to `app_mention`, `message` (in threads), and DMs.
- **User auth and project association**: Login via DB Gateway (`/login/external` with Slack ID), validate that the user is associated with the channel’s project.
- **Session management**: Store user session (user + JWT) in Redis with configurable TTL; support reset and auto-login when session expires.
- **LLM flow**: Build request (project, channel, user, optional model and knowledge base), call backend `POST /api/omni/ingest`; backend uses slackbot’s `/send-thread-message` as callback to post replies in the thread.
- **HTTP API**: Health check, send message to channel, send message to thread (used by backend/commands-api to deliver responses).

## High-level architecture

- **Entry**: `index.js` – loads env, validates required vars (`SLACK_BOT_TOKEN`, `SLACK_SIGNING_SECRET`), instantiates `SlackAPI`, registers handlers for mention/message/DM, mounts `routes`, starts the Bolt app on `PORT` (default 3000).
- **SlackAPI** (`controller/SlackAPI.js`): Bolt `App` + `ExpressReceiver`. Registers `app_mention`, `message` (thread only), and DM handlers. For each event: auth user (Redis session or DB Gateway login), check project association, register or verify thread with DB Gateway, then delegate to the same message handler (`mentionMessage` / `dmMessage` from `customerMessages.js`).
- **customerMessages.js**: Implements mention/DM logic: strip bot mention, handle built-in commands (`reset session`, `get docs`, `set model`, `get model`), otherwise call `sendToLLM` which builds payload and POSTs to stack-backend `/api/omni/ingest` with callback URL `{SELF_URL}/send-thread-message`.
- **DBGateway** (`controller/DBGateway.js`): HTTP client to ace-db-gateway for login, project-by-channel, project-users check, thread registration, discovery docs, model get/save, knowledge-base ID.
- **DevSession** (`controller/DevSession.js`): Extends Redis common layer; keyed by Slack user ID, prefix `SESSION_PREFIX` (default `session:bots:slack:`). Stores `{ user, token }` and TTL; used for auth and for channel-level model preference cache (`model:channel:{channelID}`).
- **RedisCommon** (`controller/RedisCommon.js`): Generic Redis (ioredis) wrapper: get/set/delete session, TTL, override, restart session.
- **routes/index.js**: Mounted on Bolt’s Express receiver. GET `/health`, POST `/send-message` (channel + text), POST `/send-thread-message` (channelID, ts, message); thread route checks debug flag and filters debug-only messages unless project has debug enabled.

## Main flows

1. **Mention in channel (new thread)**  
   User @mentions bot → `app_mention` → auth (session or login) → project check → register thread (DB Gateway) → `mentionMessage` → optional commands or `sendToLLM` → backend ingests, later calls `/send-thread-message` to post reply.

2. **Reply in existing thread**  
   User sends message in thread → `message` (with `thread_ts`) → auth → `isRegisteredMessage` (only process if thread was registered by ACE) → same handler as mention (commands or `sendToLLM`).

3. **DM**  
   User DMs bot → `message` with `channel_type === 'im'` → `dmMessage` → commands or `sendToLLM` (project resolved by user’s first project).

4. **Backend/commands-api posting to Slack**  
   External service POSTs to `/send-message` or `/send-thread-message`; slackbot uses Bolt client to post to channel or thread. For thread, debug mode is checked via DB Gateway and some debug-only message patterns are suppressed if project debug is off.

## Bot authorization

Messages from **bots** (e.g. other Slack apps posting as a user) are allowed only if the assumed `user` (Slack user ID) is authorized for the channel’s project. `DBGateway.checkProjectUsersSlackIds(channel, user)` is used for this check; if not authorized, the event is ignored.

## Repository

- **Code**: `ace-slackbot/` (sibling to ace-manual). Entry point `index.js`, `controller/`, `routes/`.
- **Docs in repo**: `ace-slackbot/docs/` (may be outdated; this manual is the source of truth derived from code).
