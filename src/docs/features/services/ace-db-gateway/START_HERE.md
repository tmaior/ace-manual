# Start Here – ace-db-gateway

**ace-db-gateway** is the centralized database gateway for ACE. All configuration and related data access from the backend, bots, commands-api, ops-scheduler, and other services goes through this API. It validates JWT or internal service tokens on every protected request and uses a single PostgreSQL database plus Redis for sessions.

## Contents

- **[README.md](./README.md)** – Overview of this folder, purpose, and how to use the documentation.
- **[index.md](./index.md)** – Simple list of contents of this directory.
- **[overview.md](./overview.md)** – What the service is, purpose, high-level architecture (Express, PostgreSQL, Redis, auth), and main components.
- **[authentication.md](./authentication.md)** – JWT (login, external login, refresh) and internal service tokens; which routes are public and which require auth.
- **[api-endpoints.md](./api-endpoints.md)** – Reference list of all API endpoints by resource (public, auth, and `/api` routes).
- **[environment-variables.md](./environment-variables.md)** – Environment variables used by the service (database, JWT, Redis, internal tokens, AWS Secrets Manager).
- **[database-and-models.md](./database-and-models.md)** – Single PostgreSQL database, connection config, and list of Sequelize models; schema is owned by ace-configuration.

## Where to find more

- **Repository**: `ace-db-gateway/` (sibling to ace-manual). Application code (Express, routes, controllers, models). Prefer reading the code for current behavior; repo `docs/` may be outdated.
- **Swagger**: When the service is running, interactive API docs are at `/api/docs`.
- **Architecture**: [../../architecture/service-catalog.md](../../architecture/service-catalog.md), [../../architecture/data-flow.md](../../architecture/data-flow.md).
