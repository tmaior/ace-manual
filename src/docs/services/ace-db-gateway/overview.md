# ace-db-gateway – Overview

## What it is

**ace-db-gateway** is the centralized database gateway for the ACE platform. It is the single HTTP API through which the backend and other authorized services access configuration and operational data. All requests (except login, external login, signup, and check endpoints) require authentication via JWT or internal service token. The service uses a single PostgreSQL database (configuration DB) and Sequelize for persistence; sessions are stored in Redis.

## Purpose

- **Single point of DB access**: Backend (ace-stack-backend), bots (ace-slackbot, ace-sec-bot, ace-ops-bot), ace-commands-api, ace-ops-scheduler, ace-jira-integration, and other callers access configuration and related data only through this API. No direct database access from those apps for this data.
- **Authentication**: Validates JWT (from login or external login) or internal service tokens on every protected request. Issues and refreshes JWT for dashboard/users.
- **Centralized data API**: Exposes REST endpoints for clients, users, projects, channels, documentations, schedulers, configurations, secrets (via AWS Secrets Manager), commands history, chat history, conversation channels, monitored resources, GitHub tokens, project–Jira links, KB deletion blacklist, and related resources.

## High-level architecture

- **Runtime**: Node.js, Express 5. Application entry point is `index.js`; it mounts routes from `routes/index.js` and serves Swagger UI at `/api/docs`.
- **Database**: Single PostgreSQL database. Connection URL from `CONFIGURATION_DB_URL`. Sequelize with optional SSL (when `DB_USE_SSL=true`). All entities (clients, users, projects, channels, schedulers, configurations, etc.) live in this database; see [database-and-models.md](./database-and-models.md).
- **Session store**: Redis (host/port from `REDIS_HOST` / `REDIS_PORT`) for user sessions; used after login and external login.
- **Secrets**: Project/channel secrets are stored in AWS Secrets Manager; the gateway reads/writes them using `SECRETS_REGION` and `SECRET_PATH_PREFIX`. Encryption key for local handling from `ENCRYPTION_KEY`.
- **Auth**: JWT signed with `JWT_SECRET`; internal callers can use fixed tokens (`BACKEND_DB_GATEWAY_TOKEN`, `COMMANDS_API_DB_GATEWAY_TOKEN`, `SCHEDULLER_DB_GATEWAY_TOKEN`). See [authentication.md](./authentication.md).

## Main components

| Component | Role |
|-----------|------|
| **index.js** | Express server: CORS, JSON body, request logging, Swagger, routes, global error and 404 handlers. Listens on `PORT` (default 3000). |
| **routes/index.js** | Mounts `/login`, `/login/external`, `/check`, `/signup`, `/auth/refresh-token`; all other routes under `/api` with `Auth.validator` middleware. |
| **controllers/Auth.js** | Login, external login, JWT generation, session save, validator (JWT or internal token), refresh token, check user–project by Slack IDs. |
| **config/database.js** | Sequelize config for PostgreSQL; builds `configurationDB` from `CONFIGURATION_DB_URL`. |
| **config/db.js** | Re-exports `configurationDB` for app use. |
| **models/** | Sequelize models (Client, User, Project, Channel, Scheduler, Configuration, etc.) and associations; all use `configurationDB`. |
| **config/swagger.js** | Swagger/OpenAPI setup; routes document endpoints with `@swagger` JSDoc. |

## What you can do with this app

- Authenticate users (email/password or external provider) and obtain/refresh JWT.
- Call REST APIs to manage clients, users, projects, channels, documentations, schedulers, configurations, secrets, commands history, chat history, conversation channels, monitored resources, GitHub tokens, project–Jira links, KB deletion blacklist, and related entities.
- Use internal service tokens from ace-stack-backend, ace-commands-api, and ace-ops-scheduler to access the same APIs without user JWT.
- Use `/check/*` endpoints (e.g. user–project by Slack IDs, debug-project, email/provider registered) without JWT for integration checks.

## Related documentation

- [Authentication](./authentication.md)
- [API endpoints](./api-endpoints.md)
- [Environment variables](./environment-variables.md)
- [Database and models](./database-and-models.md)
- Architecture: [service-catalog](../../architecture/service-catalog.md), [data-flow](../../architecture/data-flow.md).
