# Features and capabilities

## Webhook handling

- **issue_created**: On every create, resolve ACE project link by Jira project key/name. If linked, forward full issue + ace_metadata (event: `issue_created`) to ops-scheduler. No assignee filter.
- **issue_updated**: Resolve link; **only if** the issue assignee is the ACE user (accountId from Jira user search for `ACE_USER`), forward issue + ace_metadata (event: `issue_updated`). Otherwise respond 200 without forwarding.
- **issue_deleted**: Resolve link; if linked, forward issue + ace_metadata (event: `issue_deleted`).
- **comment_created**: Check if comment body mentions the app (Jira app accountId from `/rest/api/3/myself`) or the ACE user (accountId for `ACE_USER`). Supports Wiki Markup string (`[~accountid:...]`) and ADF (mention node with attrs.id). If not mentioned, respond 200 without forwarding. If mentioned, resolve link; if linked, optionally fetch full issue from Jira REST API (fields: summary, status, priority, issuetype, project, assignee, labels, description, comment), condense last 10 comments, and forward issue + trigger comment + recent_comments + ace_metadata (event: `comment_created`).

## Assign-to-ACE filter (issue_updated)

- **ACE_USER**: Env var (e.g. `ace@ezops.cloud`). The app looks up this user in Jira via `GET /rest/api/3/user/search?query=...` and gets `accountId`.
- **verifyAssign**: Middleware runs before issue_updated handler; sets `req.assignedToACE = true` only when `issue.fields.assignee.accountId === aceAccountId`.
- Forward to ops-scheduler only when `req.assignedToACE` is true and link exists.

## Comment mention detection

- **getMyselfAccountId**: Uses addon http client `GET /rest/api/3/myself` to get the app's Jira accountId.
- **isAppMentionedInComment(commentBody, appAccountId, aceUserAccountId)**: Returns true if the comment (string or ADF) contains a mention of either accountId (Wiki `[~accountid:...]` or ADF `type: 'mention'`, `attrs.id`).

## Descriptor resolution (multi-env)

- **resolve-descriptor.js**: Runs at startup (imported first in app.js). Reads `atlassian-connect.json`, replaces `{{APP_KEY}}`, `{{APP_NAME}}`, `{{APP_URL}}` with env values (or defaults), writes file back. Enables one descriptor in repo for all environments (local, dev, stg, prod) with different baseUrl.
- **Defaults**: APP_KEY `jira-ace-integration`, APP_NAME `Jira ACE Integration`, APP_URL from `APP_URL` or `AC_LOCAL_BASE_URL` or `https://jira.localdash.ace.ezops.cloud`.

## Logging and debugging

- **Raw webhook bodies**: Each webhook handler writes the request body to `logs/<event>.<timestamp>.json` (e.g. `logs/issue_created.1773763570964.json`). Directory is created if missing.
- **Console**: Logs ace user accountId lookup, assign-to-ACE result, ops-scheduler send errors, and comment_created flow (mention, link, issueFromApi). JWT in URLs are redacted in morgan (redactJwtTokens).
- **Test bypass**: In non-production, POST /webhook/test/* routes allow triggering the same flow without Jira; requests are also logged to `logs/test_<event>.<timestamp>.json`.

## Connect and UI

- **Atlassian Connect**: JWT auth, lifecycle (installed), scopes (READ), webhook registration, generalPages (hello-world example).
- **Hello-world**: Authenticated route `/hello-world`; can render Handlebars (hello-world.hbs) or JSX (hello-world.jsx with Parcel build and server-side-rendering).
- **Storage**: Host client data in Sequelize store (SQLite dev, Postgres production). Production can use RDS with SSL (app.js injects dialectOptions.ssl with us-east-1-bundle.pem).

## What is not included

- No health endpoint in application code (can be added for K8s probes).
- No CRUD API for project-jira-links; links are managed via ace-db-gateway (and configuration/backend).
- Test bypass routes are disabled in production (`NODE_ENV === 'production'`).
