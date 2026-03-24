# Features and capabilities

## Authentication

- **JWT**: Issued after local login or OAuth; validated by `JwtAuthGuard`; used as Bearer token for protected routes and forwarded to ace-db-gateway as the DB Gateway token. Secret and expiry from `JWT_SECRET`, `JWT_EXPIRES_IN`.
- **Local (email/password)**: `POST /auth/login` with `LocalAuthGuard`; credentials validated via DB Gateway `/login`; JWT returned.
- **Google OAuth**: `GET /auth/google` (redirect to Google); callback at `GET /auth/google-redirect`. Supports login and "connect" (link Google to existing user); state stored in Redis.
- **GitHub OAuth**: `GET /auth/github` (redirect to GitHub); callback at `GET /auth/github/callback`. Supports login and "connect"; optional "integrations" flow (shared GitHub account). State and token storage via Redis and GitHub integrations service.
- **Refresh token**: `GET /auth/refresh-token` with `JwtAuthGuard`; returns new JWT.
- **Guards**: `JwtAuthGuard`, `LocalAuthGuard`, `GoogleAuthGuard`, `GithubAuthGuard`, `SuperUserGuard` (admin-only routes).

## Admin API

- **Scope**: Users, clients, projects, configurations, channels, bridges, knowledge bases, project-jira-links, kb-deletion-blacklist. All under `api/admin` (and `api/admin/channels`, `api/admin/knowledge-base`); protected by JWT; many routes also require `SuperUserGuard`.
- **Behaviour**: CRUD and list endpoints delegate to AdminService, which calls ace-db-gateway with the user's token. Pagination, search, and filters supported where applicable (e.g. users, projects).

## User API

- **Scope**: Registration, email verification, password reset, onboarding, profile, providers (link OAuth), search by email, batch names, user projects, client, GitHub integration. Base path `api/users`; mix of public (e.g. signup, verify-email, request-password-reset, reset-password) and JWT-protected routes.

## Channels, configurations, monitored resources

- **Channels**: `api/admin/channels` – CRUD, list by project, batch names; JWT + admin.
- **Configurations**: `api/admin/configurations` – validate-repo (e.g. GitHub); admin.
- **Monitored resources**: `api/monitored-resources` – CRUD, list by project, status patch; JWT.

## Documentations and wikidocs

- **Documentations**: `api/documentations` – discovery, policy, monitoring, overview, security, schedules; CRUD for docs. `api/documentations/wikidocs` – pages, search. JWT where required.
- **Webhooks**: `api/webhooks/docs` – POST by projectId for documentation webhooks.

## Chat and omnichannel

- **Chat**: `api/chat` – conversations, messages by conversationId, send, view; uses LLM service and DB Gateway; JWT.
- **Omnichannel**: REST – `api/omni/ingest`, `api/omni/callback`, `api/omni/system-alert`, `api/omni/test-bridge`. WebSocket at path `/api/omni/ws` (Socket.IO): events such as `join_conversation`, `leave_conversation` for real-time delivery; Redis adapter for multi-instance scaling.

## Integrations and secrets

- **GitHub integrations**: `integrations/github` – temp-token, login (redirect), callback, status, revoke; OAuth and token storage; uses Redis for state.
- **Secrets**: `api/secrets` – list, secrets-description by projectId, create/update/delete operations; JWT; delegates to gateway or internal logic as implemented.
- **External linkages**: `api/external-linkages` – users, external-users, clients, projects, link-user-to-project, link-users-to-project, delete link, invite-user; JWT.

## Knowledge base and commands history

- **Knowledge base**: `api/admin/knowledge-base` – sync (e.g. Bedrock KB sync); admin. Provisioning and docs-webhook logic in dedicated services.
- **Commands history**: `commands-history` – GET list; used for history of commands.

## Notifications

- **Email**: SES for verification, password reset, and other transactional emails; `SES_FROM_EMAIL`, `AWS_REGION` required when notification module is used.

## Logging and behaviour

- **Logger**: Custom `LokiLogger`; structured logging for debugging and operations.
- **Raw body**: `rawBody: true` on the Express app for webhooks that need signature verification (e.g. raw body for signing).
- **Migrations**: TypeORM migrations for local entities; run via `migration:run`, `migration:run:prod`, etc., using `shared/database/datasource.ts`.

## Security and validation

- **Input validation**: Use of class-validator and DTOs where applied; validation and error handling in controllers and services.
- **CORS**: Restrictive; only allowed origins (see [dependencies-and-integrations.md](./dependencies-and-integrations.md)) are accepted.
- **Secrets**: No secrets in code; env from AWS Secrets Manager in non-dev or from `.env.development` / `.env.test` in dev/test.
