# Start Here – ace-ops-bot

**ace-ops-bot** is the ACE operations bot service. It runs a Slack app (Bolt + Express), stores user sessions in Redis, and integrates with ace-db-gateway (auth, project/channel data) and ace-stack-backend (LLM/omni ingest). It handles channel mentions, threaded replies, and DMs, and exposes HTTP endpoints for posting messages and receiving LLM callbacks.

## Contents

- **[README.md](./README.md)** – Overview of this folder, purpose of the service, and how to use the docs.
- **[index.md](./index.md)** – Simple list of contents of this directory.
- **[overview.md](./overview.md)** – What the app is, high-level architecture, main components, and request flow (mention, thread, DM, callback).
- **[dependencies-and-integrations.md](./dependencies-and-integrations.md)** – Runtime and npm dependencies; Slack, Redis, ace-db-gateway, ace-stack-backend; who uses ace-ops-bot.
- **[features-and-capabilities.md](./features-and-capabilities.md)** – Slack event handling, built-in commands (reset session, get docs), LLM/omni integration, thread registration and origin filtering, mock mode, HTTP endpoints summary.
- **[environment-and-configuration.md](./environment-and-configuration.md)** – Required and optional environment variables, defaults, secrets, per-environment notes.
- **[express-routes-and-callbacks.md](./express-routes-and-callbacks.md)** – POST /send-message and POST /send-thread-message: request/response and callback flow with ace-stack-backend.

## Where to find more

- **Repository**: `ace-ops-bot/` (sibling to ace-manual). Application code lives there; prefer reading the code over the repo’s `docs/` as it may be outdated.
- **Architecture**: [../../architecture/service-catalog.md](../../architecture/service-catalog.md) and related architecture docs.
