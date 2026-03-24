# Dependencies and integrations

## External dependencies (runtime)

| Dependency | Purpose |
|------------|---------|
| **Jira (Atlassian Cloud or Connect)** | Host product. The app registers as an add-on via the Connect descriptor; Jira sends webhooks (issue and comment events) with JWT. The app also calls Jira REST API (user search, /rest/api/3/myself, issue by key) using the addon's authenticated http client. |
| **PostgreSQL (production)** | atlassian-connect-express store for host client data (client key, host public key, etc.). Used when `NODE_ENV=production` and `config.json` production.store uses `dialect: postgres`, `url: $DATABASE_URL`. RDS with SSL is supported (see app.js and us-east-1-bundle.pem). |
| **SQLite (development)** | Default store in config.json development: `dialect: sqlite`, `storage: database.sqlite`. |

## ACE services

| Service | Integration |
|---------|-------------|
| **ace-db-gateway** | **GET /api/project-jira-links** with query `jiraProjectKey` or `jiraProjectName`. Used to resolve ACE project (projectId, jiraIntegrationId, defaultBoardId, project.clientId) for each webhook. Requires `ACE_DB_GATEWAY_ENDPOINT` and `ACE_DB_GATEWAY_TOKEN`. |
| **ace-ops-scheduler** | **POST /api/payloads?project_id=...** to forward Jira issue payload and ace_metadata (event, clientId, projectId, jiraIntegrationId, defaultBoardId). ops-scheduler then sends to omnichannel ingest. Requires `OPS_SCHEDULER_URL`. |

## Main libraries (package.json)

| Package | Use |
|---------|-----|
| **express** | HTTP server. |
| **@atlassian/atlassian-connect-express** | Atlassian Connect middleware: descriptor, JWT auth, host client storage (Sequelize), registration, authenticated http client for Jira API. |
| **body-parser, cookie-parser, compression, morgan** | Request parsing and logging. |
| **helmet, nocache** | Security (HSTS, referrer policy, no-cache). |
| **express-hbs** | Handlebars view engine for generalPages. |
| **pg, sequelize** | PostgreSQL driver and ORM for Connect store (production). |
| **react, react-dom, styled-components** | Optional: JSX generalPages and server-side rendering (server-side-rendering.js). |
| **parcel-bundler** | Build views/*.jsx for browser and node (dev dependency). |

## Integration flow (summary)

1. Jira sends webhook to app (e.g. POST /webhook/issue_created) with JWT.
2. App authenticates via atlassian-connect-express and reads issue/project from body.
3. App calls ace-db-gateway GET /api/project-jira-links?jiraProjectKey=... (or jiraProjectName).
4. If no link: respond 200, do not forward.
5. If link exists (and for issue_updated, if assigned to ACE_USER): POST to ace-ops-scheduler /api/payloads?project_id=... with issue + ace_metadata.
6. ops-scheduler forwards to omnichannel ingest and can update execution via callback.

## Requirements

- **Node.js**: ES modules (`type: "module"` in package.json); Node version per repo/Dockerfile.
- **Network**: App must be reachable by Jira (public URL for webhooks and descriptor). Outbound to ace-db-gateway, ace-ops-scheduler, and Jira REST API.
- **Secrets**: DATABASE_URL (production), ACE_DB_GATEWAY_TOKEN, and optionally ACE_USER; stored in AWS Secrets Manager per env and mounted in EKS.
