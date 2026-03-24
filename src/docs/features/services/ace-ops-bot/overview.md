# ace-ops-bot – Overview

**ace-ops-bot** is the ACE operations bot service. It runs a Slack app (using Bolt with Express receiver), stores user sessions in Redis, and integrates with **ace-db-gateway** (auth and project/channel data) and **ace-stack-backend** (LLM/omni ingest). It handles channel mentions, threaded replies, and DMs, and exposes HTTP endpoints so other services can post messages or receive callbacks (e.g. LLM replies into a thread).

## Purpose

- Provide an operations-focused Slack bot for ACE: answer in channels (when mentioned or in registered threads) and in DMs.
- Authenticate users via ace-db-gateway (Slack login, project association) and keep sessions in Redis.
- Register channel threads with origin `OPS` (and accept replies in threads created by `PAYLOADS` or `CRON`).
- Send user messages to the stack-backend omni ingest API and receive LLM responses via a callback to post in the same Slack thread.

## High-level architecture

- **Entry point**: `index.js` – loads env, checks required Slack env vars, instantiates `SlackAPI`, registers handlers for mentions, generic messages (threads), and DMs via `customerMessages`, applies Express routes (e.g. `/send-message`, `/send-thread-message`), then starts the Bolt app (HTTP server).
- **Slack**: `@slack/bolt` with `ExpressReceiver`. Events: `app_mention` (channel, non-thread only), `message` (thread messages only; ignores if not a registered ACE thread or if origin is not OPS/PAYLOADS/CRON).
- **Session**: `OpsSession` (extends `RedisCommon`) – keyed by Slack user ID, stores `{ user, token }` with TTL; used for auth and JWT for ace-db-gateway.
- **DB Gateway client**: `DBGateway` – all calls use JWT from session. Used for: Slack login, project-by-channel, user–project check, channel-thread register/is-registered, monitoring docs, user projects, knowledge-base ID (with Redis cache in customerMessages).
- **Message handling**: `customerMessages.js` – `mentionMessage`, `dmMessage`; built-in commands (`reset session`, `get docs`); otherwise sends to stack-backend `/api/omni/ingest` with `callback_url` pointing to ops-bot’s `/send-thread-message`.

## Main components (from code)

| Component | Role |
|-----------|------|
| `index.js` | Bootstrap: env check, SlackAPI, handlers, routes, `slackAPI.start()` (port from `PORT` or 3000). |
| `controller/SlackAPI.js` | Bolt App + ExpressReceiver; auth via session + DB Gateway; mention/message/DM handlers; `messageToChannel` / `messageToThread`. |
| `controller/customerMessages.js` | `mentionMessage`, `dmMessage`; `checkCommands` (reset session, get docs); `sendToLLM` (omni ingest + callback). |
| `controller/DBGateway.js` | HTTP client for ace-db-gateway (JWT). Login, channels, channel-thread, documentations, projects, knowledge-base, user–project check. |
| `controller/OpsSession.js` | Session CRUD over Redis (extends RedisCommon); prefix from `SESSION_PREFIX`. |
| `controller/RedisCommon.js` | ioredis connection; get/save/delete/override/restart session. |
| `controller/Logger.js` | Chalk-based console logger (INFO, ERROR, DEBUG, WARNING). |
| `routes/index.js` | Express routes on `receiver.router`: POST `/send-message`, POST `/send-thread-message`. |

## Flow (simplified)

1. **Mention in channel (no thread)**  
   `app_mention` → auth (session or DB Gateway login) → check project association → register thread (origin OPS) → `mentionMessage` → command or `sendToLLM` → stack-backend posts reply via callback to `/send-thread-message`.

2. **Message in thread**  
   `message` (with `thread_ts`) → auth → skip if not project-associated → `isRegisteredMessage` → allow only origins OPS, PAYLOADS, CRON → `mentionMessage` (same flow as above).

3. **DM**  
   `message` in `channel_type === 'im'` → `dmMessage` → command or `sendToLLM` (project from user’s first project if no channel).

4. **Callback**  
   Stack-backend calls `POST /send-thread-message` with `channelID`, `ts`, `message`; ops-bot posts the message into that thread.

## Related docs

- [Dependencies and integrations](./dependencies-and-integrations.md)
- [Features and capabilities](./features-and-capabilities.md)
- [Environment and configuration](./environment-and-configuration.md)
- [Express routes and callbacks](./express-routes-and-callbacks.md)
