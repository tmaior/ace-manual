# API endpoints

Summary of HTTP routes and WebSocket path, derived from the application controllers. Base URL is the backend root (e.g. `http://localhost:3000`). No global prefix is set; each controller defines its own path. Authentication: JWT via `Authorization: Bearer <token>` unless noted as public or OAuth.

## Auth (no JWT for login/OAuth entrypoints)

| Method | Path | Description |
|--------|------|-------------|
| POST | `/auth/login` | Local login (email/password); returns JWT. |
| GET | `/auth/google` | Redirect to Google OAuth. |
| GET | `/auth/google-redirect` | Google OAuth callback (login or connect). |
| GET | `/auth/github` | Redirect to GitHub OAuth. |
| GET | `/auth/github-redirect` | Legacy GitHub redirect. |
| GET | `/auth/github/callback` | GitHub OAuth callback (login or connect). |
| GET | `/auth/refresh-token` | Refresh JWT (requires valid JWT). |

## Root

| Method | Path | Description |
|--------|------|-------------|
| GET | `/` | Root (e.g. health or welcome); see app.controller. |

## Admin (`api/admin`) – JWT + SuperUser where applied

| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/admin/users` | List users (pagination, search, hierarchy, clientId, sort). |
| GET | `/api/admin/users/user-types` | List user types. |
| GET | `/api/admin/users/:id` | Get user by id. |
| POST | `/api/admin/users` | Create user. |
| PUT | `/api/admin/users/:id` | Update user. |
| DELETE | `/api/admin/users/:id` | Delete user. |
| PUT | `/api/admin/users/:id/move-to-unknown` | Move user to unknown client. |
| GET | `/api/admin/clients` | List clients. |
| GET | `/api/admin/clients/:id` | Get client. |
| POST | `/api/admin/clients` | Create client. |
| PUT | `/api/admin/clients/:id` | Update client. |
| DELETE | `/api/admin/clients/:id` | Delete client. |
| GET | `/api/admin/projects` | List projects (pagination, name, clientId, allow, sort). |
| GET | `/api/admin/projects-ordered` | List projects ordered. |
| GET | `/api/admin/projects/:id` | Get project. |
| POST | `/api/admin/projects` | Create project. |
| PUT | `/api/admin/projects/:id` | Update project. |
| DELETE | `/api/admin/projects/:id` | Delete project. |
| POST | `/api/admin/projects/:projectId/users` | Add users to project. |
| POST | `/api/admin/projects/:projectId/channels` | Add channels to project. |
| POST | `/api/admin/users/:id/projects` | Add projects to user. |
| DELETE | `/api/admin/users/:id/projects` | Remove user from projects. |
| GET | `/api/admin/knowledge-bases` | List knowledge bases. |
| DELETE | `/api/admin/knowledge-bases/:knowledgeBaseId` | Delete knowledge base. |
| GET | `/api/admin/kb-deletion-blacklist` | List KB deletion blacklist. |
| POST | `/api/admin/kb-deletion-blacklist` | Add to blacklist. |
| DELETE | `/api/admin/kb-deletion-blacklist/:id` | Remove from blacklist. |
| GET | `/api/admin/project-jira-links` | List project Jira links. |
| GET | `/api/admin/project-jira-links/:id` | Get project Jira link. |
| POST | `/api/admin/project-jira-links` | Create project Jira link. |
| PUT | `/api/admin/project-jira-links/:id` | Update project Jira link. |
| DELETE | `/api/admin/project-jira-links/:id` | Delete project Jira link. |
| GET | `/api/admin/projects/:id/knowledge-base` | Get project knowledge base. |
| GET | `/api/admin/configurations` | List configurations (with query params). |
| GET | `/api/admin/configurations/:id` | Get configuration. |
| POST | `/api/admin/configurations` | Create configuration. |
| PUT | `/api/admin/configurations/:id` | Update configuration. |
| DELETE | `/api/admin/configurations/:id` | Delete configuration. |
| GET | `/api/admin/bridges` | List bridges. |
| POST | `/api/admin/bridges` | Create bridge. |
| PUT | `/api/admin/bridges/:id` | Update bridge. |
| PATCH | `/api/admin/bridges/:id/toggle` | Toggle bridge. |
| DELETE | `/api/admin/bridges/:id` | Delete bridge. |

## Admin channels (`api/admin/channels`)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/admin/channels` | List channels. |
| GET | `/api/admin/channels/project` | List by project (query). |
| GET | `/api/admin/channels/:id` | Get channel. |
| POST | `/api/admin/channels` | Create channel. |
| PUT | `/api/admin/channels/:id` | Update channel. |
| DELETE | `/api/admin/channels/:id` | Delete channel. |
| GET | `/api/admin/channels/:id/name` | Get channel name. |
| POST | `/api/admin/channels/names/batch` | Batch get names. |

## Admin knowledge-base and configurations

| Method | Path | Description |
|--------|------|-------------|
| POST | `/api/admin/knowledge-base/sync` | Trigger KB sync. |
| POST | `/api/admin/configurations/validate-repo` | Validate repo (e.g. GitHub). |

## Users (`api/users`)

| Method | Path | Description |
|--------|------|-------------|
| POST | `/api/users` | Register user. |
| POST | `/api/users/verify-email` | Verify email. |
| POST | `/api/users/resend-verification` | Resend verification email. |
| POST | `/api/users/request-password-reset` | Request password reset. |
| POST | `/api/users/reset-password` | Reset password. |
| POST | `/api/users/onboarding` | Onboarding. |
| GET | `/api/users/search-by-email` | Search by email. |
| POST | `/api/users/names/batch` | Batch get names. |
| GET | `/api/users/:userId/projects` | User projects. |
| GET | `/api/users/client/:clientId` | Users by client. |
| GET | `/api/users/github-integration` | GitHub integration status. |
| GET | `/api/users/profile` | Current user profile. |
| POST | `/api/users/profile` | Update profile. |
| GET | `/api/users/providers` | List providers. |
| POST | `/api/users/providers/link` | Start link provider (e.g. OAuth connect). |

## External linkages (`api/external-linkages`)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/external-linkages/users` | List users. |
| GET | `/api/external-linkages/external-users` | List external users. |
| GET | `/api/external-linkages/clients` | List clients. |
| GET | `/api/external-linkages/projects` | List projects. |
| POST | `/api/external-linkages/link-user-to-project` | Link user to project. |
| POST | `/api/external-linkages/link-users-to-project` | Link users to project. |
| DELETE | `/api/external-linkages/users/:userId/projects/:projectId` | Unlink. |
| POST | `/api/external-linkages/invite-user` | Invite user. |

## Documentations (`api/documentations`)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/documentations/discovery` | Discovery doc. |
| GET | `/api/documentations/policy` | Policy doc. |
| GET | `/api/documentations/monitoring` | Monitoring doc. |
| GET | `/api/documentations/overview` | Overview doc. |
| GET | `/api/documentations/security` | Security doc. |
| GET | `/api/documentations/schedules` | Schedules doc. |
| GET | `/api/documentations` | List documentations. |
| GET | `/api/documentations/:id` | Get documentation. |
| POST | `/api/documentations` | Create. |
| PUT | `/api/documentations/:id` | Update. |
| DELETE | `/api/documentations/:id` | Delete. |

## Wikidocs (`api/documentations/wikidocs`)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/documentations/wikidocs/pages` | List pages. |
| GET | `/api/documentations/wikidocs/pages/:id` | Get page. |
| GET | `/api/documentations/wikidocs/search` | Search. |

## Chat (`api/chat`)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/chat/conversations` | List conversations. |
| GET | `/api/chat/messages/:conversationId` | Messages for conversation. |
| POST | `/api/chat/send` | Send message. |
| POST | `/api/chat/view` | View (e.g. mark viewed). |

## Monitored resources (`api/monitored-resources`)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/monitored-resources` | List (with filters). |
| GET | `/api/monitored-resources/project/:projectId` | By project. |
| GET | `/api/monitored-resources/:id` | Get one. |
| POST | `/api/monitored-resources` | Create. |
| PUT | `/api/monitored-resources/:id` | Update. |
| PATCH | `/api/monitored-resources/:id/status` | Update status. |
| DELETE | `/api/monitored-resources/:id` | Delete. |

## Omnichannel REST (`api/omni`)

| Method | Path | Description |
|--------|------|-------------|
| POST | `/api/omni/ingest` | Ingest message. |
| POST | `/api/omni/callback` | Callback. |
| POST | `/api/omni/system-alert` | System alert. |
| POST | `/api/omni/test-bridge` | Test bridge. |

## Webhooks and secrets

| Method | Path | Description |
|--------|------|-------------|
| POST | `/api/webhooks/docs/:projectId` | Documentation webhook. |
| GET | `/api/secrets` | List secrets. |
| GET | `/api/secrets/secrets-description/:projectId` | Secrets description by project. |
| PUT | `/api/secrets/:projectId` | Update secrets. |
| POST | `/api/secrets/:projectId` | Create. |
| POST | `/api/secrets/create-or-update/:projectId` | Create or update. |
| PUT | `/api/secrets/update-single-secret/:projectId` | Update single secret. |
| DELETE | `/api/secrets/delete-single-secret/:projectId` | Delete single secret. |

## Integrations and commands history

| Method | Path | Description |
|--------|------|-------------|
| POST | `/integrations/github/temp-token` | Create temp token for GitHub flow. |
| GET | `/integrations/github/login` | Start GitHub OAuth (redirect). |
| GET | `/integrations/github/callback` | GitHub OAuth callback (integrations). |
| GET | `/integrations/github/status` | GitHub integration status. |
| DELETE | `/integrations/github/revoke` | Revoke GitHub integration. |
| GET | `/commands-history` | List commands history. |

## WebSocket

- **Path**: `/api/omni/ws` (Socket.IO).
- **Events**: `join_conversation`, `leave_conversation` (and others as implemented in omnichannel gateway). Redis adapter used for multi-instance support.
