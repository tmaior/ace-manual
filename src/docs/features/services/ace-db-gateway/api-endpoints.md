# API endpoints

All endpoints below are relative to the service base URL (e.g. `http://localhost:3000`). The service exposes Swagger UI at `/api/docs` for interactive documentation. Routes under `/api` require authentication (JWT or internal token); see [authentication.md](./authentication.md).

## Public and auth endpoints (no JWT required for `/login`, `/login/external`, `/check`, `/signup`)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/` | API name, version, link to `/api/docs`. |
| POST | `/login` | Login with email and password; returns JWT and user info. |
| POST | `/login/external` | Login with providerType and providerId; returns JWT and user. |
| GET | `/auth/refresh-token` | Refresh JWT (requires valid Bearer token). |
| GET | `/check/user-project/slack-id/:channel/:user` | Check if Slack user is in a project that has the given Slack channel. |
| GET | `/check/debug-project/:channel` | Check if channel has debug enabled in configuration. |
| GET | `/check/user-id/slack-id/:slackId` | Get user id by Slack ID. |
| GET | `/check/email-registered/:email` | Check if email is registered. |
| GET | `/check/provider-registered/:providerType/:providerId` | Check if provider is registered. |
| POST | `/signup/provider` | Sign up external provider. |

## API-prefixed routes (prefix `/api`; all require Auth)

Base path for the following is `/api`. Example: list projects = `GET /api/projects`.

### Admin

| Method | Path | Description |
|--------|------|-------------|
| GET | `/admin/projects-ordered` | Projects full information ordered (admin panel). |
| * | `/admin/configurations` | Configuration CRUD (admin). |

### Clients

| Method | Path | Description |
|--------|------|-------------|
| GET | `/clients` | List clients. |
| GET | `/clients/users` | Clients and users. |
| GET | `/clients/projects` | Clients and projects. |
| GET | `/clients/usertypes` | Clients and user types. |
| GET | `/clients/full` | Clients full information. |
| GET | `/clients/:id` | Client by id. |
| POST | `/clients` | Create client. |
| PUT | `/clients/:id` | Update client. |
| DELETE | `/clients/:id` | Delete client. |
| GET | `/clients/:id/users` | Users by client id. |
| GET | `/clients/:id/projects` | Projects by client id. |
| POST | `/clients/bulk` | Bulk create clients. |

### Users

| Method | Path | Description |
|--------|------|-------------|
| GET | `/users` | List users. |
| GET | `/users/search-by-email` | Search by email. |
| GET | `/users/search-by-email-exact` | Search by exact email. |
| GET | `/users/client` | Users and client. |
| GET | `/users/client/:id` | Users by client. |
| GET | `/users/user_type` | Users and user type. |
| GET | `/users/projects` | Users and projects. |
| GET | `/users/full` | Users full information. |
| GET | `/users/:id` | User by id. |
| GET | `/users/numeric/:id` | User by numeric id. |
| POST | `/users` | Create user. |
| PUT | `/users/:id` | Update user. |
| PUT | `/users/numeric/:id` | Update user by numeric id. |
| DELETE | `/users/:id` | Delete user. |
| DELETE | `/users/numeric/:id` | Delete user by numeric id. |
| GET | `/users/:id/projects` | User projects. |
| POST | `/users/external` | Create external user. |
| PUT | `/users/external/:id` | Update external user. |
| GET | `/users/external` | List external users. |
| POST | `/users/bulk` | Bulk create users. |
| POST | `/users/link-provider` | Link user to provider. |
| POST | `/users/:userId/projects` | Link user to projects. |
| POST | `/users/:userId/projects/:projectId` | Link user to project. |
| DELETE | `/users/:userId/projects/:projectId` | Remove project from user. |
| DELETE | `/users/:userId/projects` | Unlink user from projects. |
| POST | `/users/names/batch` | Get user names batch. |
| POST | `/users/names/by-ids/batch` | Get user names by ids batch. |
| POST | `/users/onboarding` | Onboarding. |
| POST | `/users/numeric/:id/verify-password` | Verify password (numeric id). |

### User types and permissions

| Method | Path | Description |
|--------|------|-------------|
| GET | `/user_types` | List user types. |
| GET | `/user_types/users` | User types and users. |
| GET | `/user_types/permissions` | User types and permissions. |
| GET | `/user_types/full` | User types full information. |
| GET | `/user_types/:id` | User type by id. |
| POST | `/user_types` | Create user type. |
| PUT | `/user_types/:id` | Update user type. |
| DELETE | `/user_types/:id` | Delete user type. |
| GET | `/permissions` | List permissions. |
| GET | `/permissions/:id` | Permission by id. |
| POST | `/permissions` | Create permission. |
| PUT | `/permissions/:id` | Update permission. |
| DELETE | `/permissions/:id` | Delete permission. |
| GET | `/user_type_permissions` | List user type permissions. |
| GET | `/user_type_permissions/:id` | User type permission by id. |
| POST | `/user_type_permissions` | Create user type permission. |
| PUT | `/user_type_permissions/:id` | Update. |
| DELETE | `/user_type_permissions/:id` | Delete. |

### Projects

| Method | Path | Description |
|--------|------|-------------|
| GET | `/projects` | List projects. |
| GET | `/projects/by-name` | Project by exact name. |
| GET | `/projects/client` | Projects and client. |
| GET | `/projects/channels` | Projects and channels. |
| GET | `/projects/users` | Projects and users. |
| GET | `/projects/full` | Projects full information. |
| GET | `/projects/full-ordered` | Projects full information ordered. |
| GET | `/projects/knowledge-bases` | List knowledge bases. |
| GET | `/projects/:id` | Project by id. |
| GET | `/projects/:id/docs-repository-token` | Project docs repository token. |
| GET | `/projects/:id/knowledge-base` | Project knowledge base. |
| PUT | `/projects/:id/knowledge-base` | Set project knowledge base. |
| DELETE | `/projects/:id/knowledge-base` | Delete project knowledge base. |
| GET | `/projects/:id/jira-link` | Project Jira link. |
| PUT | `/projects/:id/jira-link` | Set project Jira link. |
| DELETE | `/projects/:id/jira-link` | Delete project Jira link. |
| POST | `/projects` | Create project. |
| PUT | `/projects/:id` | Update project. |
| DELETE | `/projects/:id` | Delete project. |
| POST | `/projects/:projectId/users` | Link project to users. |
| POST | `/projects/:projectId/channels` | Link project to channels. |

### Channels and project-channels

| Method | Path | Description |
|--------|------|-------------|
| GET | `/channels` | List channels. |
| GET | `/channels/project` | Channel and project. |
| GET | `/channels/:id` | Channel by id. |
| POST | `/channels` | Create channel. |
| PUT | `/channels/:id` | Update channel. |
| DELETE | `/channels/:id` | Delete channel. |
| POST | `/channels/:slackId/model` | Set channel model. |
| GET | `/channels/:slackId/model` | Get channel model. |
| GET | `/projects_channels` | List project-channel links. |
| GET | `/projects_channels/:id` | By id. |
| POST | `/projects_channels` | Create. |
| PUT | `/projects_channels/:id` | Update. |
| DELETE | `/projects_channels/:id` | Delete. |

### Documentations, schedulers, configurations

| Method | Path | Description |
|--------|------|-------------|
| GET | `/documentations` | List documentations. |
| GET | `/documentations/filter` | Filtered documentations. |
| GET | `/documentations/:id` | By id. |
| GET | `/documentations/channel/:channelId` | By channel id. |
| POST | `/documentations` | Create. |
| PUT | `/documentations/:id` | Update. |
| DELETE | `/documentations/:id` | Delete. |
| POST | `/documentations/bulk` | Bulk create. |
| GET | `/schedulers` | List schedulers. |
| GET | `/schedulers/:id` | By id. |
| GET | `/schedulers/channel/:channelId` | By channel id. |
| POST | `/schedulers` | Create. |
| PUT | `/schedulers/:id` | Update. |
| PATCH | `/schedulers/:id/toggle` | Toggle. |
| PUT | `/schedulers/:id/run-status` | Update run status. |
| DELETE | `/schedulers/:id` | Delete. |
| POST | `/schedulers/bulk` | Bulk create. |
| GET | `/configurations` | Get configuration. |
| POST | `/configurations` | Create configuration. |
| PUT | `/configurations/:id` | Update configuration. |
| DELETE | `/configurations/:id` | Delete configuration. |

### Secrets, commands history, channel thread, chat, conversations

| Method | Path | Description |
|--------|------|-------------|
| POST | `/secrets/create-or-update/:projectId` | Create or update secrets. |
| PUT | `/secrets/update-single-secret/:projectId` | Update single secret. |
| DELETE | `/secrets/delete-single-secret/:projectId` | Delete single secret. |
| GET | `/secrets/secrets-description/:projectId` | Secrets description. |
| GET | `/secrets/:channelID` | Get secrets by channel. |
| POST | `/secrets/:channelID` | Create secrets for channel. |
| PUT | `/secrets/:channelID` | Update secrets. |
| DELETE | `/secrets/:channelID` | Delete secrets. |
| GET | `/commands-history` | List commands history. |
| GET | `/commands-history/:id` | By id. |
| POST | `/commands-history` | Create. |
| PUT | `/commands-history/:id` | Update. |
| DELETE | `/commands-history/:id` | Delete. |
| POST | `/channel-thread/register` | Register message. |
| GET | `/channel-thread/is-registered-message/:channelSlackId/:ts/:origin` | Check if message registered. |
| GET | `/channel-thread/by-conversa-id/:conversaId` | By conversa id. |
| POST | `/chat-history` | Create chat history. |
| GET | `/chat-history/conversations` | List conversations. |
| GET | `/chat-history/:conversa_id` | By conversa id. |
| POST | `/conversation-channels/register` | Register conversation channel. |
| GET | `/conversation-channels/by-conversa/:conversaId` | By conversa id. |
| GET | `/conversation-channels/resolve-thread/:channelId/:threadTs` | Resolve by thread. |
| PATCH | `/conversation-channels/:id/deactivate` | Deactivate. |

### External linkages, monitored resources, GitHub tokens

| Method | Path | Description |
|--------|------|-------------|
| POST | `/external-linkages/invite-user` | Invite user. |
| GET | `/monitored-resources` | List monitored resources. |
| GET | `/monitored-resources/:id` | By id. |
| GET | `/monitored-resources/project/:projectId` | By project. |
| POST | `/monitored-resources` | Create. |
| PUT | `/monitored-resources/:id` | Update. |
| PUT | `/monitored-resources/batch` | Batch upsert. |
| PUT | `/monitored-resources/increment-misses` | Increment misses. |
| PATCH | `/monitored-resources/:id/status` | Update status. |
| DELETE | `/monitored-resources/:id` | Delete. |
| POST | `/github-tokens` | Create or update GitHub token. |
| GET | `/github-tokens/user/:userId` | By user id. |
| GET | `/github-tokens/user/:userId/exists` | Check exists. |
| DELETE | `/github-tokens/user/:userId` | Delete by user id. |

### KB deletion blacklist, project Jira links

| Method | Path | Description |
|--------|------|-------------|
| GET | `/kb-deletion-blacklist` | List entries. |
| POST | `/kb-deletion-blacklist` | Create. |
| DELETE | `/kb-deletion-blacklist/:id` | Delete. |
| GET | `/project-jira-links` | List project Jira links. |
| GET | `/project-jira-links/:id` | By id. |
| POST | `/project-jira-links` | Create. |
| PUT | `/project-jira-links/:id` | Update. |
| DELETE | `/project-jira-links/:id` | Delete. |

## Project users

| Method | Path | Description |
|--------|------|-------------|
| GET | `/project_users` | List. |
| GET | `/project_users/:id` | By id. |
| POST | `/project_users` | Create. |
| PUT | `/project_users/:id` | Update. |
| DELETE | `/project_users/:id` | Delete. |

---

For request/response schemas and examples, use the service’s Swagger UI at `/api/docs` or the `@swagger` JSDoc in the repository routes.
