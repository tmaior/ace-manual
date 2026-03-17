# ace-db-gateway

Centralized database gateway for the ACE platform. It exposes a single HTTP API for configuration and operational data; validates JWT or internal service tokens on every protected request; and uses one PostgreSQL database (configuration DB) plus Redis for sessions.

## Purpose

- **Single point of DB access**: Backend, bots, ace-commands-api, ace-ops-scheduler, ace-jira-integration, and other callers access this data only through ace-db-gateway, not by connecting directly to the database.
- **Authentication**: Login (email/password and external provider), JWT issue and refresh, and internal tokens for services. All `/api` routes require a valid Bearer token (JWT or service token).
- **Tech stack**: Node.js, Express 5, Sequelize (PostgreSQL), Redis (sessions), AWS Secrets Manager (project/channel secrets). Documentation is in English.

## What you will find here

| Document | Description |
|----------|-------------|
| [overview.md](./overview.md) | What the service is, architecture, main components, and what you can do with it. |
| [authentication.md](./authentication.md) | JWT and internal token auth; public vs protected routes. |
| [api-endpoints.md](./api-endpoints.md) | List of all endpoints by resource. |
| [environment-variables.md](./environment-variables.md) | Env vars for server, DB, auth, Redis, AWS, internal tokens. |
| [database-and-models.md](./database-and-models.md) | Single PostgreSQL DB, connection, and Sequelize models; schema owned by ace-configuration. |

## How to use

- **New to the service**: Start with [START_HERE.md](./START_HERE.md), then [overview.md](./overview.md) and [authentication.md](./authentication.md).
- **Integrating or calling the API**: Use [api-endpoints.md](./api-endpoints.md) and the live Swagger UI at `/api/docs` when the service is running.
- **Deploying or configuring**: Use [environment-variables.md](./environment-variables.md) and the repository code (e.g. `config/database.js`, `index.js`).

## Related

- [Architecture service catalog](../../architecture/service-catalog.md)
- [Data flow](../../architecture/data-flow.md)
- **Repository**: `ace-db-gateway/` – read the application code for up-to-date behavior; repo `docs/` may be outdated.
