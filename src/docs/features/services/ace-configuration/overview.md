# ace-configuration – Overview

## What it is

**ace-configuration** is the configuration and schema management service for the ACE (Automation, Control & Enablement) platform. It owns the PostgreSQL schema and Sequelize models for core ACE data (clients, users, projects, channels, configurations, monitored resources, schedulers, etc.). It does **not** act as an HTTP API gateway for configuration data: the backend and other services read and write configuration data via **ace-db-gateway** (or by importing models and connecting to the same database). ace-configuration’s main runtime roles are **running migrations and seeders** and, when run as a server, exposing a **health** check endpoint.

## Purpose

- **Schema ownership**: Defines and evolves the configuration database schema via Sequelize migrations; all tables use soft deletes (paranoid) and unique constraints that include `deletedAt`.
- **Migrations and seeders**: Creates/updates tables and seeds initial or reference data (clients, permissions, user types, projects, channels, KB deletion blacklist, etc.). In EKS, the service is typically run as a **Job** that executes `db:create`, `db:migrate`, and `db:seed` then exits.
- **Models for other services**: Exports Sequelize models via **models/index.js** (Channel, Client, GithubToken, Permission, Project_User, Project, ProjectKnowledgeBase, ProjectJiraLink, User, UserType_Permission, UserType, UsersProviders). Other model files (e.g. Configuration, MonitoredResource, Scheduler, Documentations, SecretsDescription, Channel_Thread) exist in the repo; add them to models/index.js if consumers need them. ace-stack-backend, ace-db-gateway, ace-slackbot, ace-ops-scheduler and other Node services import these models to access the database directly.
- **Configuration storage**: The **Configurations** table stores key-value configuration per project; application code reads/writes it via DB Gateway or direct model use, not via an ace-configuration REST API.

## High-level architecture

- **Database**: Single PostgreSQL database (connection via `DATABASE_URL` or `DB_*` env vars). TLS when `DB_USE_SSL=true`; local/dev can use no SSL. Config in `config/db.js` and `config/cliConnection.js`.
- **Runtime modes**:
  - **Job (EKS)**: Run `yarn start:configuration` (db:create, db:migrate, db:seed). No long-running HTTP server; used for schema and data setup.
  - **Server (local/optional)**: `index.js` starts an Express server on `PORT` (default 3000), exposes `GET /health` for liveness/readiness.
- **Consumers**: ace-stack-backend, ace-db-gateway, ace-slackbot, ace-sec-bot, ace-ops-bot, ace-commands-api, ace-ops-scheduler import models from ace-configuration and connect to the same DB; configuration data is accessed via DB Gateway or direct queries, not by calling ace-configuration HTTP API.

## Main components

| Component | Role |
|-----------|------|
| **config/db.js** | Sequelize instance: `DATABASE_URL` or env-based config from cliConnection; TLS options; connection test on load. |
| **config/cliConnection.js** | Per-environment DB config (development, local, production, demo): host, database, SSL, logging. |
| **models/** | Sequelize models loaded in models/index.js and associations; paranoid (soft delete) and timestamps. Full schema (all tables) is created by migrations. |
| **migrations/** | Versioned schema changes (create tables, add columns/indexes, unique constraints with deletedAt). |
| **seeders/** | Initial data: clients, permissions, user types, projects, channels, user-project links, KB deletion blacklist defaults (requires AWS CLI for Bedrock KB resolution when used). |
| **index.js** | Entry point: Express server with `/health`; or start:configuration runs only db create/migrate/seed. |

## What you can do with this app

- Run database migrations and seeders for the ACE configuration database (locally or via EKS Job).
- Import and use Sequelize models in other ACE services to query clients, users, projects, channels, configurations, monitored resources, schedulers, and related entities.
- Expose a health check endpoint when the server is started (e.g. local development).
- Extend the schema via new migrations and seeders following the repo’s patterns (see docs in ace-configuration repo).

## Related documentation

- [Dependencies and integrations](./dependencies-and-integrations.md)
- [Features and capabilities](./features-and-capabilities.md)
- [Environment and configuration](./environment-and-configuration.md)
- [Build, deploy and CI/CD](./build-deploy-and-cicd.md)
- [Database and migrations](./database-and-migrations.md)
- Repository docs: `ace-configuration/docs/` (architecture, setup, development).
- Architecture: [service-catalog](../../architecture/service-catalog.md), [data-flow](../../architecture/data-flow.md), [diagrams](../../architecture/diagrams.md).
