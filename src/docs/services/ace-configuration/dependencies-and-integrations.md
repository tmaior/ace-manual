# Dependencies and integrations

## External dependencies (runtime)

| Dependency | Purpose |
|------------|---------|
| **PostgreSQL** | Primary data store for configuration schema (clients, users, projects, channels, configurations, monitored resources, schedulers, etc.). Connection via `DATABASE_URL` or `DB_HOST`, `DB_DATABASE`, `DB_USERNAME`, `DB_PASSWORD`; TLS when `DB_USE_SSL=true`. |

No mandatory outbound HTTP calls from ace-configuration to other ACE services; other services depend on ace-configuration (models and schema), not the other way around.

## Main libraries (package.json)

| Package | Use |
|---------|-----|
| **express** | HTTP server when run in server mode (health check only). |
| **sequelize** | ORM: models, migrations, connection to PostgreSQL. |
| **sequelize-cli** | CLI for migrations and seeders (db:create, db:migrate, db:seed). |
| **pg** | PostgreSQL driver for Sequelize. |
| **dotenv** | Load `.env` for local development. |
| **bcryptjs** | Password hashing in User model. |
| **nodemon** | Dev dependency for auto-restart in local mode. |

Slack/OpenAI dependencies (`@slack/bolt`, `openai`) are present in package.json but Slack/LLM features are not active in the current entrypoint; the app’s main role is schema and seed.

## Who uses ace-configuration

| Consumer | How |
|----------|-----|
| **ace-db-gateway** | Connects to the same configuration database; exposes configuration and related data via its API. Backend and frontend access configuration **via DB Gateway**, not by calling ace-configuration. |
| **ace-stack-backend** | Imports models (e.g. User, Client, Project) or calls DB Gateway for auth and configuration. |
| **ace-slackbot, ace-sec-bot, ace-ops-bot** | Import models (User, Channel, Project, etc.) or use DB Gateway for channel/project and user context. |
| **ace-commands-api** | Uses configuration (e.g. project context, secrets metadata) via DB Gateway or model import. |
| **ace-ops-scheduler** | Imports Scheduler, MonitoredResource and related models or uses DB Gateway for scheduled jobs and resource status. |

## Integration pattern (architecture)

- **ace-configuration** owns the schema and runs migrations/seeders. It does **not** provide a general-purpose REST API for CRUD on configuration entities in production.
- **Configuration data access**: Application reads/writes go through **ace-db-gateway** (e.g. admin `/api/configurations`) or by importing ace-configuration models and connecting to the same database. See [architecture diagrams](../../architecture/diagrams.md) and [integration patterns](https://github.com/ezops-br/ace-configuration/blob/main/docs/architecture/integration-patterns.md) in the repo.

## Requirements

- **Node.js**: Compatible with project tooling (e.g. Node 16+); check repo and Dockerfile for exact version.
- **PostgreSQL**: Version compatible with Sequelize (e.g. 13+). For DocumentDB/RDS, TLS and certificate path may be required (`us-east-1-bundle.pem` in repo).
- **Network**: Database reachability from the environment where migrations/seed or the server run (local, EKS Job, or long-running pod).
