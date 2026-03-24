# Database and models

ace-db-gateway uses a **single PostgreSQL database** called the configuration database. All Sequelize models are attached to the same Sequelize instance (`configurationDB`) defined in `config/database.js` and exported from `config/db.js`. There are no separate “docs” or “schedule” databases in the current codebase (any reference to `docsDB` or `scheduleDB` in the repo is legacy or unused).

## Connection

- **Config file**: `config/database.js`
- **Connection URL**: `process.env.CONFIGURATION_DB_URL`
- **Dialect**: PostgreSQL (`pg` driver)
- **SSL**: Optional; when `DB_USE_SSL` is `true`, SSL is enabled (e.g. with RDS CA bundle).
- **Pool**: max 10, min 0, acquire 30000 ms, idle 10000 ms; logging disabled in config.

Models are loaded in `models/index.js`; associations are set up there so that relations (e.g. User–Client, Project–Channel) work across models.

## Models (entities)

The following Sequelize models are registered and used by the gateway. All use `configurationDB`.

| Model | Purpose |
|-------|---------|
| **Channel** | Slack (or other) channels; can be linked to projects. |
| **Client** | Tenants/organizations. |
| **GithubToken** | GitHub tokens per user. |
| **Permission** | Permission definitions. |
| **Project_User** | Join table: user–project membership. |
| **Project** | Projects; can have channels, users, knowledge base, Jira link. |
| **User** | Users; belong to client and user type, can have providers (e.g. Slack). |
| **UserType_Permission** | Join: user type – permission. |
| **UserType** | User type (role-like); has permissions. |
| **UsersProviders** | External provider links (e.g. Slack id) to users. |
| **Documentation** | Documentation records; can be scoped by channel. |
| **Scheduler** | Scheduler definitions (e.g. ops schedules). |
| **Configuration** | Key-value configuration (e.g. per project). |
| **ChannelThread** | Channel thread / message registration. |
| **SecretsDescription** | Metadata for secrets (actual secrets in AWS Secrets Manager). |
| **MonitoredResource** | Monitored resources (e.g. for health checks). |
| **ProjectKnowledgeBase** | Project–knowledge base association. |
| **ProjectJiraLink** | Project–Jira link (e.g. for ace-jira-integration). |
| **KbDeletionBlacklist** | Blacklist for KB deletion. |
| **CommandsHistory** | History of commands. |
| **ConversationChannel** | Conversation–channel mapping. |
| **ChatHistory** | Chat history records. |

Schema (tables, columns, indexes) is defined and evolved in the **ace-configuration** service via migrations and seeders. ace-db-gateway only reads and writes data through these models; it does not run migrations. For table details and migration workflow, see ace-configuration documentation (e.g. [database-and-migrations.md](../ace-configuration/database-and-migrations.md) in ace-manual) and the ace-configuration repo.

## Related documentation

- [Overview](./overview.md)
- [Environment variables](./environment-variables.md) for `CONFIGURATION_DB_URL` and `DB_USE_SSL`
- ace-configuration: schema ownership, migrations, seeders
