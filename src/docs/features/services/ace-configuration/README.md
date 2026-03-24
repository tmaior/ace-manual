# ace-configuration

Node.js service that owns the ACE configuration database schema and seed data. It runs migrations and seeders (in EKS as a Job) and provides Sequelize models consumed by ace-stack-backend, ace-db-gateway, ace-slackbot, ace-ops-scheduler, and other ACE services. Configuration data is read and written via **ace-db-gateway** or direct DB access, not via an ace-configuration REST API.

## Purpose

- **Schema ownership**: Define and evolve the configuration database (PostgreSQL) via Sequelize migrations; all tables use soft deletes and unique constraints that include `deletedAt`.
- **Migrations and seeders**: Create/update tables and seed initial data (clients, permissions, user types, projects, channels, KB blacklist, etc.). In EKS this runs as a one-off Job.
- **Models for other services**: Export Sequelize models via models/index.js (Channel, Client, GithubToken, Permission, Project_User, Project, ProjectKnowledgeBase, ProjectJiraLink, User, UserType_Permission, UserType, UsersProviders) for direct use or via ace-db-gateway. Other model files exist in the repo; wire them in models/index.js if needed.

## Documentation in this folder

| Document | Description |
|----------|-------------|
| [overview.md](./overview.md) | What the app is, architecture, components, capabilities. |
| [dependencies-and-integrations.md](./dependencies-and-integrations.md) | External deps, libraries, who uses ace-configuration, integration pattern. |
| [features-and-capabilities.md](./features-and-capabilities.md) | Schema ownership, seed, runtime modes, health check, no config CRUD API. |
| [environment-and-configuration.md](./environment-and-configuration.md) | Required/optional env vars, per-env behavior, secrets. |
| [build-deploy-and-cicd.md](./build-deploy-and-cicd.md) | Local build, Docker, Kubernetes Job, GitHub Actions (dev/demo/prod). |
| [database-and-migrations.md](./database-and-migrations.md) | Tables summary, migrations, seeders, connection, repo docs. |

## Repository docs

- **ace-configuration/docs/** – e.g. architecture (integration-patterns, database-schema, data-models, system-overview), setup (database-setup, environment-setup, local-development), development (database-changes).

## Related

- [Architecture service catalog](../../architecture/service-catalog.md)
- [Data flow](../../architecture/data-flow.md)
- [Architecture diagrams](../../architecture/diagrams.md)
- [Environments](../../environments/) for local, demo, production
