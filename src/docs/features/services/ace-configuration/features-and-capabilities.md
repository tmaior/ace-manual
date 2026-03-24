# Features and capabilities

## 1. Database schema ownership

- **Migrations**: Versioned schema changes (create tables, add/remove columns, indexes, unique constraints). All unique constraints include `deletedAt` to support soft deletes. Migrations run via `yarn db:migrate` or `yarn start:configuration`.
- **Models**: Sequelize model files for Clients, Users, UserTypes, Permissions, Projects, Project_User, Channels, Channel_Threads, Configurations, MonitoredResources, Schedulers, Documentations, SecretsDescriptions, UsersProviders, GithubToken, ProjectKnowledgeBase, ProjectJiraLink, and related join tables; all use `paranoid: true` (soft delete) and timestamps. **models/index.js** currently loads and exports: Channel, Client, GithubToken, Permission, Project_User, Project, ProjectKnowledgeBase, ProjectJiraLink, User, UserType_Permission, UserType, UsersProviders. Other model files exist in the repo and can be added to models/index.js for use by other services.
- **Associations**: Defined in each model’s `associate()`; loaded and wired in `models/index.js`.

## 2. Seed data

- **Seeders**: Initial and reference data (clients, permissions, user types, users, projects, channels, user-project links). One seeder populates KB deletion blacklist defaults and may call AWS (Bedrock KB list) when run in an environment with AWS CLI configured.
- **Execution**: `yarn db:seed` or as part of `yarn start:configuration`. Order is defined by seeder timestamps/filenames.

## 3. Runtime modes

- **Job (EKS)**: Script `start:configuration` runs `db:create`, `db:migrate`, `db:seed` and exits. Used in CI/CD to apply schema and seed after deploy; no long-lived HTTP server.
- **Server (local/optional)**: `index.js` starts Express on `PORT` (default 3000), registers `GET /health`. Useful for local health checks and liveness/readiness when deployed as a server.

## 4. Health check

- **Health**: `GET /health` returns `200` and `{ ok: true, status: 'healthy' }`. Used by liveness/readiness when the service is deployed as a server.

## 5. Configuration data and consumers

- **Configurations table**: Key-value storage per project (and optional `configuredByUserId`). Consumed by other services via **ace-db-gateway** or direct model import; no dedicated “configuration API” in ace-configuration.
- **Multi-tenant**: Data is organized by Client → Projects, Users, UserTypes; Projects have Channels, Configurations, MonitoredResources, etc. Isolation is enforced by application logic and DB constraints in the services that use the data.

## 6. No REST API for configuration CRUD

- The backend and other services do **not** call ace-configuration for configuration CRUD. They use **ace-db-gateway** (e.g. `/api/configurations`) or direct DB access via imported models. ace-configuration’s responsibility is schema and seed, not serving configuration over HTTP.

## 7. Graceful behavior

- When run as a server: normal process lifecycle (listen on PORT, respond to /health). When run as a Job: process exits after migrations and seed complete (success or failure).
