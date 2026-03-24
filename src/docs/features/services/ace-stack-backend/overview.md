# ace-stack-backend – Overview

## What it is

**ace-stack-backend** is the main NestJS API for the ACE (Automation, Control & Enablement) platform. It handles authentication (JWT and OAuth with Google and GitHub), business logic, and orchestration. The frontend and other clients call this service; the backend in turn calls **ace-db-gateway** for most data operations and optionally integrates with **ace-commands-api**, LLM services, AWS (Secrets Manager, SES, S3, Bedrock), and Slack.

## Purpose

- **Authentication**: Issues and validates JWT; supports local (email/password), Google OAuth, and GitHub OAuth; supports "connect" flows to link providers to existing users; refresh token endpoint.
- **Orchestration**: Central API for the dashboard and bots; delegates reads/writes to **ace-db-gateway** using the user's DB Gateway token (JWT-driven).
- **Real-time**: WebSocket gateway (Socket.IO over Redis adapter) at `/api/omni/ws` for omnichannel conversations and system alerts.
- **Tech stack**: NestJS 11, Node.js, TypeScript, TypeORM (PostgreSQL), Redis, Passport (JWT, local, Google, GitHub). Typical HTTP port: 3000.

## High-level architecture

- **HTTP API**: No global route prefix; controllers use paths such as `auth`, `api/admin`, `api/users`, `api/chat`, etc. CORS is restricted to `FRONTEND_HOST` and a few localhost origins.
- **Secrets**: In non-development/non-test, secrets are loaded from AWS Secrets Manager (`AWS_SECRET_NAME`, `AWS_REGION`) at bootstrap and merged into `process.env`; in development/test, `.env.development` or `.env.test` is loaded via dotenv.
- **Database**: TypeORM connects to PostgreSQL (env: `DB_HOST`, `DB_PORT`, `DB_USER`, `DB_PASSWORD`, `DB_NAME`); optional SSL via `DB_SSL_ENABLED`, `DB_SSL_REJECT_UNAUTHORIZED`, `DB_SSL_CA`. Used for local entities (e.g. user-related entities) and migrations; bulk of domain data is via ace-db-gateway.
- **WebSockets**: Socket.IO with Redis adapter (`REDIS_URL`) for horizontal scaling; gateway path `/api/omni/ws`.

## Main components

| Component | Role |
|-----------|------|
| **main.ts** | Bootstrap: load secrets (AWS or dotenv), create Nest app with LokiLogger, Redis WebSocket adapter, CORS, listen on PORT. |
| **app.module.ts** | Root module: ConfigModule (global), DatabaseModule, UserModule, AuthModule, NotificationModule, SecretsModule, DocumentationsModule, CommandsHistoryModule, AdminModule, ChannelsModule, SlackModule, ExternalLinkagesModule, ChatModule, MonitoredResourcesModule, GithubIntegrationsModule, ConfigurationsModule, OmnichannelModule, KnowledgeBaseModule, WebhooksModule. |
| **auth/** | JWT issue/validate, Passport strategies (local, jwt, google, github), guards; login, OAuth redirects, refresh token. |
| **db-gateway/DBGateway.ts** | Static helper to call ace-db-gateway (GET/POST); base URL from `ACE_GATEWAY_URL`; used for user existence, signup, login, and other gateway endpoints. |
| **user/** | User registration, email verification, password reset, profile, providers, onboarding; uses DB Gateway and local TypeORM entities. |
| **admin/** | CRUD for users, clients, projects, configurations, channels, bridges, knowledge bases, project-jira-links, kb-deletion-blacklist; all via DB Gateway with JWT + SuperUserGuard where required. |
| **omnichannel/** | REST ingest/callback/system-alert/test-bridge and WebSocket gateway for real-time messaging. |
| **shared/** | Redis Io adapter, fetch-secrets (AWS Secrets Manager), TypeORM datasource, Loki logger. |

## What you can do with this app

- Run the main ACE API locally or in EKS (HTTP + WebSockets).
- Authenticate users via email/password or Google/GitHub OAuth and issue JWT for ace-db-gateway and frontend.
- Manage clients, projects, users, configurations, channels, monitored resources, documentations, and integrations via the documented API.
- Use the omnichannel WebSocket for real-time conversation and alert delivery.
- Run TypeORM migrations for local schema (e.g. user, password-reset-token, email-verification-token) via `migration:run` scripts.

## Related documentation

- [Dependencies and integrations](./dependencies-and-integrations.md)
- [Features and capabilities](./features-and-capabilities.md)
- [API endpoints](./api-endpoints.md)
- [Environment and configuration](./environment-and-configuration.md)
- [Build, deploy and CI/CD](./build-deploy-and-cicd.md)
- Architecture: [service-catalog](../../architecture/service-catalog.md), [data-flow](../../architecture/data-flow.md).
