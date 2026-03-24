# Start Here – ace-slackbot

**ace-slackbot** is the ACE Slack bot service. It receives Slack events (mentions, thread messages, DMs), authenticates users via ace-db-gateway, keeps sessions in Redis, and sends messages to ace-stack-backend for LLM processing. Other services can post to Slack via its HTTP endpoints.

## Contents

- **[README.md](./README.md)** – Overview of this folder, purpose, and links to all docs.
- **[index.md](./index.md)** – Simple list of contents of this directory.
- **[overview.md](./overview.md)** – What the app is, high-level architecture, main components, and event/callback flows.
- **[dependencies-and-integrations.md](./dependencies-and-integrations.md)** – External dependencies (Slack, Redis, ace-db-gateway, ace-stack-backend), main libraries, and who calls ace-slackbot.
- **[features-and-capabilities.md](./features-and-capabilities.md)** – Slack events, built-in commands (reset session, get/set model, get docs), HTTP routes (health, send-message, send-thread-message), auth, debug mode, mock mode.
- **[environment-and-configuration.md](./environment-and-configuration.md)** – Required and optional env vars, defaults, startup behavior, and secrets.

## Where to find more

- **Repository**: `ace-slackbot/` (sibling to ace-manual). Code: `index.js`, `controller/`, `routes/`. Repo docs in `ace-slackbot/docs/` may be outdated; this manual is derived from the code.
- **Architecture**: [../../architecture/service-catalog.md](../../architecture/service-catalog.md), [../../architecture/data-flow.md](../../architecture/data-flow.md).
