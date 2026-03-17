# Start Here – ace-sec-bot

**ace-sec-bot** is the ACE security bot service. It runs as a Slack bot (Node.js, @slack/bolt), uses Redis for sessions and Knowledge Base cache, integrates with **ace-db-gateway** for auth and project/channel data, and sends user messages to an external **AI/LLM API** for chat replies. It also exposes HTTP routes so other services can post messages or thread replies to Slack.

## Contents

- **[README.md](./README.md)** – Overview of this folder, purpose of the service, and links to all docs.
- **[index.md](./index.md)** – Simple list of contents of this directory.
- **[overview.md](./overview.md)** – What the app is, purpose, high-level architecture, main components, and what you can do with it.
- **[dependencies-and-integrations.md](./dependencies-and-integrations.md)** – External dependencies (Slack, Redis, ace-db-gateway, AI API), main libraries, who uses the bot, and integration patterns.
- **[features-and-capabilities.md](./features-and-capabilities.md)** – Slack event handling (mention, thread, DM), built-in commands (reset session, get docs), HTTP endpoints, session and auth, limitations.
- **[environment-and-configuration.md](./environment-and-configuration.md)** – Required and optional env vars, defaults, secrets, and per-environment notes.
- **[architecture-and-message-flow.md](./architecture-and-message-flow.md)** – Startup, mention/thread/DM flows, HTTP routes, session and Redis, DB Gateway endpoints used, project structure.

## Where to find more

- **Repository**: `ace-sec-bot/` (sibling to ace-manual). Prefer reading the application code; repo `docs/` may be outdated.
- **Architecture**: [../../architecture/service-catalog.md](../../architecture/service-catalog.md).
