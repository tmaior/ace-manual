# ace-jira-integration

Node.js Atlassian Connect app for Jira. It runs as an Express server using `@atlassian/atlassian-connect-express`, receives Jira webhooks (issue and comment events), resolves ACE project links via **ace-db-gateway**, and forwards payloads to **ace-ops-scheduler** for the omnichannel/LLM pipeline. It does not expose a general-purpose REST API; its main surface is the Connect descriptor and webhook endpoints.

## Purpose

- **Jira webhooks**: Handle `jira:issue_created`, `jira:issue_updated`, `jira:issue_deleted`, and `comment_created` with JWT authentication.
- **ACE project resolution**: For each webhook, look up the project Jira link by `jiraProjectKey` or `jiraProjectName` via ace-db-gateway (`GET /api/project-jira-links`). If no link exists, respond 200 and do not forward.
- **Forward to ops-scheduler**: When an ACE link exists (and for issue_updated when the issue is assigned to ACE), POST the payload to ace-ops-scheduler `POST /api/payloads?project_id=...` with `ace_metadata` (event, clientId, projectId, jiraIntegrationId, defaultBoardId).
- **Comment mention**: For comment_created, only forward when the comment mentions the app or the ACE user (ACE_USER accountId); optionally fetch full issue from Jira API and include recent comments.

## Documentation in this folder

| Document | Description |
|----------|-------------|
| [overview.md](./overview.md) | What the app is, architecture, components, capabilities. |
| [webhooks-and-routes.md](./webhooks-and-routes.md) | Descriptor, webhook URLs, auth, test bypass routes (non-production). |
| [dependencies-and-integrations.md](./dependencies-and-integrations.md) | Jira/Connect, ace-db-gateway, ace-ops-scheduler, main libraries. |
| [features-and-capabilities.md](./features-and-capabilities.md) | Webhook flow, assign-to-ACE, comment mention, descriptor resolution, logs. |
| [environment-and-configuration.md](./environment-and-configuration.md) | Env vars, config.json, APP_KEY/APP_NAME/APP_URL. |
| [build-deploy-and-cicd.md](./build-deploy-and-cicd.md) | Local build, Docker (ace-infra), Kubernetes, GitHub Actions. |

## Repository

- **ace-jira-integration/** – Application code, `atlassian-connect.json`, `routes/webhook.js`, `app.js`, `scripts/resolve-descriptor.js`. Prefer reading the code over repo `docs/` when docs may be outdated.

## Related

- [Architecture service catalog](../../architecture/service-catalog.md)
- [Data flow](../../architecture/data-flow.md)
- [ace-db-gateway project-jira-links](../ace-db-gateway/api-endpoints.md)
- [ace-ops-scheduler payloads](../ace-ops-scheduler/api-and-routes.md)
