# Webhooks and routes

## Descriptor (atlassian-connect.json)

The app is registered with Jira via the Atlassian Connect descriptor. Key fields:

- **key**: App key (e.g. `jira-ace-integration`); can be overridden at startup via `APP_KEY` (see [environment-and-configuration.md](./environment-and-configuration.md)).
- **name**, **description**: Display name and description.
- **baseUrl**: Public URL where Jira reaches the app (e.g. `https://jira.ace.ezops.cloud`). Resolved at startup from `APP_URL` or `AC_LOCAL_BASE_URL` via `scripts/resolve-descriptor.js`; the descriptor file can use placeholders `{{APP_KEY}}`, `{{APP_NAME}}`, `{{APP_URL}}` which are replaced before the addon loads.
- **authentication**: `type: "jwt"`.
- **lifecycle**: `installed` → `GET /installed` (handled by atlassian-connect-express).
- **scopes**: e.g. `READ`.
- **modules.webhooks**: Define the webhook events and URLs (see below).
- **modules.generalPages**: Example UI (e.g. hello-world in top nav and header); routes are authenticated with `addon.authenticate()`.

Descriptor is served at `/atlassian-connect.json` by atlassian-connect-express. Root path `/` redirects to it.

## Webhook endpoints (authenticated)

All webhook routes are mounted under `/webhook` and use `addon.authenticate()`. Jira sends POST requests with JWT; the addon validates the request and provides `req.context.http` (authenticated Jira REST client).

| Method | Path | Jira event | Description |
|--------|------|------------|-------------|
| POST | `/webhook/issue_created` | jira:issue_created | On issue create: resolve ACE link by project key/name; if linked, POST to ops-scheduler /api/payloads with event `issue_created`. |
| POST | `/webhook/issue_updated` | jira:issue_updated | On issue update: resolve ACE link; **only if** issue is assigned to ACE user (`ACE_USER`), POST to ops-scheduler with event `issue_updated`. |
| POST | `/webhook/issue_deleted` | jira:issue_deleted | On issue delete: resolve ACE link; if linked, POST to ops-scheduler with event `issue_deleted`. |
| POST | `/webhook/comment_created` | comment_created | On comment create: if comment mentions app or ACE user, resolve ACE link; optionally fetch full issue from Jira API; if linked, POST to ops-scheduler with event `comment_created` (includes trigger comment and recent_comments). |

Response: always 200 with a JSON body (e.g. `{ message: 'jira issue created', aceLinked: true|false, aceProjectId? }`). Payloads are also written to `logs/<event>.<timestamp>.json` for debugging.

## Querying ACE project link

The app calls **ace-db-gateway** to resolve an ACE project from the Jira project:

- **Endpoint**: `GET {ACE_DB_GATEWAY_ENDPOINT}/api/project-jira-links`
- **Query**: `jiraProjectKey=<key>` or `jiraProjectName=<name>` (tries key first, then name).
- **Auth**: `Authorization: Bearer {ACE_DB_GATEWAY_TOKEN}`.
- **Response**: Expects `{ rows: [ link ] }`; each link has `projectId`, `jiraIntegrationId`, `defaultBoardId`, and optionally `project.clientId`. If no link is found, the webhook handler does not forward to ops-scheduler.

## Forwarding to ace-ops-scheduler

When an ACE link exists (and for issue_updated when assigned to ACE):

- **Endpoint**: `POST {OPS_SCHEDULER_URL}/api/payloads?project_id={aceProjectId}`
- **Headers**: `Content-Type: application/json`, `Accept: application/json`.
- **Body**: Jira issue object plus:
  - `ace_metadata`: `{ event, clientId, projectId, jiraIntegrationId, defaultBoardId }`
  - For comment_created: `comment` (trigger comment), `recent_comments` (condensed list).

If `OPS_SCHEDULER_URL` is not set, no forward is performed (response is still 200).

## Test bypass routes (non-production only)

When `NODE_ENV !== 'production'`, the app registers **unauthenticated** test routes so QA can trigger the same flow without Jira webhooks:

| Method | Path | Description |
|--------|------|-------------|
| POST | `/webhook/test/issue_created` | Same logic as issue_created; no JWT. |
| POST | `/webhook/test/issue_updated` | Same logic as issue_updated; no JWT. |
| POST | `/webhook/test/issue_deleted` | Same logic as issue_deleted; no JWT. |
| POST | `/webhook/test/comment_created` | Same logic as comment_created (without mention check); no JWT. |

Request body must match the shape Jira would send (e.g. `issue.fields.project.key`, `issue.fields.project.name`). Response includes `bypass: true` and same ace-linked and project info. **These routes must not be exposed in production.**

## Other routes

| Method | Path | Description |
|--------|------|-------------|
| GET | `/` | Redirect to `/atlassian-connect.json`. |
| GET | `/hello-world` | Example Connect generalPage; requires `addon.authenticate()`. Renders Handlebars or JSX (hello-world.hbs / hello-world.jsx). |
| GET | `/atlassian-connect.json` | Served by atlassian-connect-express (descriptor). |

No health endpoint is defined in the application code; liveness/readiness in Kubernetes may use TCP port or a path added elsewhere.
