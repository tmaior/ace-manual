# Environment and configuration

## Required (production)

| Variable | Purpose |
|----------|---------|
| **PORT** | HTTP server port. In production, config.json uses `"port": "$PORT"`. atlassian-connect-express reads this from config. |
| **DATABASE_URL** | PostgreSQL connection string for atlassian-connect-express store (host client data). Used when NODE_ENV=production and config.json production.store has `url: "$DATABASE_URL"`. |
| **NODE_ENV** | Set to `production` in deployed environments. Enables production config (port, store, whitelist) and disables test bypass routes. |

For ACE integration (optional but needed for forwarding):

| Variable | Purpose |
|----------|---------|
| **ACE_DB_GATEWAY_ENDPOINT** | Base URL of ace-db-gateway (e.g. `http://ace-db-gateway-service`). Used for GET /api/project-jira-links. |
| **ACE_DB_GATEWAY_TOKEN** | Bearer token for ace-db-gateway. Sent as `Authorization: Bearer ...`. |
| **OPS_SCHEDULER_URL** | Base URL of ace-ops-scheduler. Used for POST /api/payloads. If unset, webhooks still respond 200 but do not forward. |

## Descriptor (multi-env)

| Variable | Purpose | Default |
|----------|---------|---------|
| **APP_KEY** | Connect app key written into atlassian-connect.json (replace {{APP_KEY}}). | `jira-ace-integration` |
| **APP_NAME** | Connect app name (replace {{APP_NAME}}). | `Jira ACE Integration` |
| **APP_URL** | Public base URL of the app (replace {{APP_URL}}). Must be reachable by Jira for webhooks and descriptor. | - |
| **AC_LOCAL_BASE_URL** | Fallback for APP_URL if APP_URL not set (e.g. local tunnel). | `https://jira.localdash.ace.ezops.cloud` |

Resolve-descriptor.js runs at startup and strips trailing slash from APP_URL. Per-environment URLs (from ace-infra README):

- Local: `https://jira.localdash.ace.ezops.cloud`
- Dev: `https://dev-jira.ace.ezops.cloud`
- Stg: `https://stg-jira.ace.ezops.cloud`
- Prod: `https://jira.ace.ezops.cloud`

## Optional

| Variable | Purpose | Default |
|----------|---------|---------|
| **ACE_USER** | Jira user (e.g. email) used for "assign-to-ACE" check on issue_updated. App searches Jira for this user and uses accountId; only forwards when issue assignee equals this accountId. | `ace@ezops.cloud` |

## config.json

- **development**: port 3000, localBaseUrl, store SQLite (`database.sqlite`). Used when NODE_ENV is not production.
- **production**: port `$PORT`, localBaseUrl `$APP_URL`, store postgres `$DATABASE_URL`, whitelist for Jira host domains (`*.atlassian.net`, etc.).
- **product**: `jira`.

Port and store are read by atlassian-connect-express via `addon.config.port()` and store config. RDS/Postgres in production can use SSL; app.js injects `dialectOptions.ssl` with `us-east-1-bundle.pem` when `NODE_ENV === 'production'` and `DATABASE_URL` starts with `postgres`.

## Secrets (EKS)

In ace-infra, secrets are stored in AWS Secrets Manager and mounted as Kubernetes Secret `jira-integration-secrets`:

- **ace/dev/jira-integration-secrets**
- **ace/stg/jira-integration-secrets**
- **ace/prod/jira-integration-secrets**

Deployment uses `envFrom: secretRef: jira-integration-secrets`. Required: `DATABASE_URL`. Optional: `ACE_DB_GATEWAY_ENDPOINT`, `ACE_DB_GATEWAY_TOKEN`, `OPS_SCHEDULER_URL`, `APP_URL`, `ACE_USER`. See [build-deploy-and-cicd.md](./build-deploy-and-cicd.md) and ace-infra `ace-jira-integration/README.md`.
