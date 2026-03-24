# Database and migrations

## Overview

ace-configuration owns the **configuration database** schema in PostgreSQL. All tables use **soft deletes** (paranoid mode) with `deletedAt`; unique constraints include `deletedAt` so the same business key can be reused after a soft delete.

## Main tables (summary)

| Table | Purpose |
|-------|---------|
| **Clients** | Top-level organizations; have many Projects, Users, UserTypes. |
| **Users** | System users; password hashed with bcrypt; belong to Client and UserType; many-to-many with Projects (Project_User). |
| **UserTypes** | Roles; hierarchy (e.g. 1=admin, 2=user); many-to-many with Permissions. |
| **Permissions** | Permission definitions. |
| **Projects** | Belong to Client; have Channels, Configurations, MonitoredResources, etc. |
| **Project_User** | Join table: Users ↔ Projects. |
| **Channels** | Slack channels; belong to Project; optional preferredModel. |
| **Channel_Threads** | Thread tracking per channel; optional origin, conversaId. |
| **Configurations** | Key-value per project; optional configuredByUserId. |
| **MonitoredResources** | Resource monitoring; projectId, resourceType, checker script, status, consecutiveMisses, etc. |
| **Schedulers** | Scheduled task config; name, schedule (cron), enabled, lastRun, nextRun, createdBy. |
| **Documentations** | Documentation records; optional channelId, projectId. |
| **SecretsDescriptions** | Project secret keys and descriptions. |
| **Users_Providers** | User ↔ external provider (e.g. Slack) link; providerId, providerType, email. |
| **GithubToken** | GitHub token storage. |
| **ProjectKnowledgeBase** | Project ↔ Knowledge Base association. |
| **ProjectJiraLink** | Project ↔ Jira link (e.g. for ace-jira-integration). |
| **KbDeletionBlacklist** | KB deletion blacklist (name, optional Bedrock KB id). |
| **CommandsHistory**, **ChatHistories**, **ConversationChannels** | Command/chat and conversation tracking. |

Full schema and relationships are in the repo: **ace-configuration/docs/architecture/database-schema.md**, **docs/architecture/data-models.md**. Note: **models/index.js** only requires and exports a subset of these models (Channel, Client, GithubToken, Permission, Project_User, Project, ProjectKnowledgeBase, ProjectJiraLink, User, UserType_Permission, UserType, UsersProviders). Add other model files to models/index.js if other services need them.

## Migrations

- **Location**: `migrations/` with timestamped filenames (e.g. `20250525163848-create-client.js`).
- **Run**: `yarn db:migrate` or as part of `yarn start:configuration`.
- **Rollback**: `yarn sequelize-cli db:migrate:undo` (last) or `db:migrate:undo:all --to <name>`.
- **Status**: `yarn sequelize-cli db:migrate:status`.
- **Create new**: `yarn sequelize-cli migration:generate --name description-of-change`.
- **Convention**: Unique indexes must include `deletedAt` for soft-delete support (see migration `20250910220935-update-unique-indexes-with-deletedat.js`).

## Seeders

- **Location**: `seeders/` with timestamped filenames.
- **Run**: `yarn db:seed` or as part of `yarn start:configuration`.
- **Order**: By filename/timestamp. One seeder (`kb-deletion-blacklist-defaults`) may call AWS (Bedrock KB list) when AWS CLI is configured.
- **Create new**: `yarn sequelize-cli seed:generate --name seeder-name`.

## Database connection

- **Config**: `config/db.js` uses `DATABASE_URL` if set; otherwise `config/cliConnection.js` by `NODE_ENV` (development, local, production, demo).
- **TLS**: When `DB_USE_SSL=true`, certificate path can be `us-east-1-bundle.pem` in project root.
- **Local**: Often `DB_HOST=127.0.0.1`, `DB_DATABASE=configuration`, no SSL.

## Repository docs

- **docs/setup/database-setup.md**: Creation, migrations, seeders, backup, troubleshooting.
- **docs/development/database-changes.md**: Creating migrations, models, seeders, best practices.
- **docs/architecture/database-schema.md**: Full table definitions and relationships.
- **docs/architecture/data-models.md**: Model definitions and usage examples.
