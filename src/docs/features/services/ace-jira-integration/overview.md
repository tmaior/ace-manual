# ace-jira-integration – Overview

## What it is

**ace-jira-integration** is the ACE Atlassian Connect app for Jira. The platform **depends on it** for everything that is driven from Jira into ACE (linked projects). It is a Node.js Express application that uses `@atlassian/atlassian-connect-express` to register as a Jira add-on, expose a Connect descriptor (`atlassian-connect.json`), and receive Jira webhooks. It does **not** expose a general-purpose REST API; its main role is to receive webhook events from Jira, resolve the corresponding ACE project via **ace-db-gateway**, and forward payloads to **ace-ops-scheduler** for the omnichannel pipeline.

## Purpose

- **Jira webhooks**: Subscribe to `jira:issue_created`, `jira:issue_updated`, `jira:issue_deleted`, and `comment_created`. All webhook handlers use JWT authentication provided by Atlassian Connect.
- **ACE project resolution**: For each event, extract Jira project key/name from the payload and call ace-db-gateway `GET /api/project-jira-links?jiraProjectKey=...` or `?jiraProjectName=...`. If no link exists, respond 200 and do not forward.
- **Forward to ops-scheduler**: When an ACE project link exists, POST the issue payload plus `ace_metadata` (event, clientId, projectId, jiraIntegrationId, defaultBoardId) to ace-ops-scheduler `POST /api/payloads?project_id=...`.
- **Assign-to-ACE filter**: For `issue_updated`, forward only when the issue is assigned to the ACE user (env `ACE_USER`); otherwise respond without forwarding.
- **Comment mention**: For `comment_created`, forward only when the comment body mentions the app (Jira app accountId) or the ACE user accountId; optionally fetch full issue from Jira REST API and include recent comments in the payload.

## High-level architecture

- **Entry point**: `app.js` runs `scripts/resolve-descriptor.js` (replace `{{APP_KEY}}`, `{{APP_NAME}}`, `{{APP_URL}}` in atlassian-connect.json from env), then boots Express with atlassian-connect-express middleware. Port from `addon.config.port()` (config.json: development 3000, production `$PORT`).
- **Routes**: `routes/index.js` mounts `/` (redirect to descriptor), `/hello-world` (Connect generalPages example), and `/webhook` from `routes/webhook.js`. Webhook routes: `/webhook/issue_created`, `/webhook/issue_updated`, `/webhook/issue_deleted`, `/webhook/comment_created`; in non-production, test bypass routes `/webhook/test/*` without auth.
- **Storage**: atlassian-connect-express uses Sequelize for host client storage (config.json: development SQLite, production PostgreSQL via `$DATABASE_URL`). Production Postgres can use RDS with SSL (`us-east-1-bundle.pem` in app.js).
- **Outbound**: ace-db-gateway (project-jira-links), Jira REST API (user search, myself, issue by key), ace-ops-scheduler (payloads). All use env: `ACE_DB_GATEWAY_ENDPOINT`, `ACE_DB_GATEWAY_TOKEN`, `OPS_SCHEDULER_URL`, `ACE_USER`.

## Main components

| Component | Role |
|-----------|------|
| **app.js** | Entry: resolve descriptor, Express + atlassian-connect-express, helmet/nocache, body-parser, routes, HTTP listen. In production with Postgres, injects SSL options for store. |
| **scripts/resolve-descriptor.js** | Replaces `{{APP_KEY}}`, `{{APP_NAME}}`, `{{APP_URL}}` in atlassian-connect.json from env; runs at startup before addon loads. |
| **routes/index.js** | Mounts `/`, `/hello-world` (addon.authenticate()), `/webhook` (webhook router). |
| **routes/webhook.js** | Webhook handlers: get ACE link via getAceProjectLinkForJiraProject(); for issue_updated, verifyAssign (ACE_USER assignee); for comment_created, isAppMentionedInComment and optional fetchIssueFromJiraApi(); forward to OPS_SCHEDULER_URL /api/payloads. Writes raw body to `logs/` per event. |
| **atlassian-connect.json** | Connect descriptor: key, name, baseUrl, authentication (JWT), lifecycle (installed), scopes, webhooks (issue_created, issue_updated, issue_deleted, comment_created), generalPages (hello-world). baseUrl can use placeholders resolved at startup. |
| **config.json** | Per-env port, localBaseUrl, store (development: sqlite/database.sqlite; production: postgres `$DATABASE_URL`), production whitelist. |
| **server-side-rendering.js** | Optional: Handlebars engine for .jsx views (e.g. hello-world); React SSR with styled-components. |

## What you can do with this app

- Install the add-on in a Jira Cloud (or Connect-compatible) instance and receive issue and comment webhooks.
- Link Jira projects to ACE projects via ace-db-gateway (project-jira-links); only events for linked projects are forwarded to ace-ops-scheduler.
- Use issue_updated to trigger ACE only when the issue is assigned to the configured ACE user.
- Use comment_created to trigger ACE when the app or ACE user is mentioned in a comment; payload can include full issue and recent comments.
- Run locally with `npm start` (after `npm run build` if using JSX) or with ngrok/tunnel for Connect registration; in EKS, deploy via ace-infra manifests and GitHub Actions.

## Related documentation

- [Webhooks and routes](./webhooks-and-routes.md)
- [Dependencies and integrations](./dependencies-and-integrations.md)
- [Features and capabilities](./features-and-capabilities.md)
- [Environment and configuration](./environment-and-configuration.md)
- [Build, deploy and CI/CD](./build-deploy-and-cicd.md)
- [ace-ops-scheduler payloads](../ace-ops-scheduler/api-and-routes.md)
- [ace-db-gateway project-jira-links](../ace-db-gateway/api-endpoints.md)
