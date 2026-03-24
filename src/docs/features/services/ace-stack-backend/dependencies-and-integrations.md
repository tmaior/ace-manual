# Dependencies and integrations

## ace-db-gateway

- **Role**: Primary data gateway. The backend does not expose a global `/api` prefix; routes are defined per controller (e.g. `api/admin`, `api/users`). All domain data (users, clients, projects, configurations, channels, monitored resources, documentations, secrets, external linkages, etc.) is read and written via **ace-db-gateway** using the user's JWT as the DB Gateway token (`Authorization: Bearer <token>`).
- **Configuration**: Base URL from `ACE_GATEWAY_URL` (default `http://ace-db-gateway-service`). Used in `DBGateway.ts` (static methods and instance methods with token) and in several services (e.g. user, external-linkages, chat) that call the gateway directly.
- **Usage**: Static methods for auth flows (userExists, providerExists, signupUser, linkProviderToUser, loginEmail, loginProvider); instance methods (with token) for authenticated CRUD. Admin, user, channels, configurations, external-linkages, monitored-resources, documentations, secrets, and other modules delegate to the gateway.

## Redis

- **Role**: (1) WebSocket adapter for Socket.IO so multiple backend instances share connection state; (2) OAuth state storage for Google and GitHub (login and connect flows) with TTL.
- **Configuration**: `REDIS_URL` (e.g. `redis://redis-service:6379` or `redis://localhost:6379`). Used in `shared/redis-io.adapter.ts`, `auth/auth.controller.ts`, `user/user.controller.ts`, `github-integrations/github-integrations.controller.ts`.
- **Libraries**: `ioredis`, `redis`, `@socket.io/redis-adapter`.

## PostgreSQL (TypeORM)

- **Role**: Local persistence for entities such as user, user-provider, password-reset-token, email-verification-token; and for TypeORM migrations. Most business data lives in the configuration/application DB accessed via ace-db-gateway, not in this direct connection.
- **Configuration**: `DB_HOST`, `DB_PORT`, `DB_USER`, `DB_PASSWORD`, `DB_NAME`; optional SSL: `DB_SSL_ENABLED`, `DB_SSL_REJECT_UNAUTHORIZED`, `DB_SSL_CA`. See [environment-and-configuration.md](./environment-and-configuration.md).
- **Module**: `DatabaseModule` in `database/database.module.ts`; datasource in `shared/database/datasource.ts` (used by CLI migrations).

## AWS

- **Secrets Manager**: At bootstrap (non-development, non-test), secrets are loaded from the secret named `AWS_SECRET_NAME` in `AWS_REGION` and merged into `process.env` via `shared/fetch-secrets.ts` (e.g. JWT_SECRET, DB_*, REDIS_URL, OAuth credentials).
- **SES**: Email sending (verification, password reset, notifications) via `notification/ses.service.ts`; requires `AWS_REGION`, `SES_FROM_EMAIL`.
- **S3 / Bedrock / S3 Vectors**: Used by the knowledge-base module (provisioning, sync, Bedrock Knowledge Bases). Optional `AWS_ACCOUNT_ID` in knowledge-base provisioning.
- **SQS / IAM / STS**: Dependencies present in package.json for queue and AWS identity usage where applicable.
- **Libraries**: `@aws-sdk/client-secrets-manager`, `@aws-sdk/client-ses`, `@aws-sdk/client-s3`, `@aws-sdk/client-bedrock-agent`, `@aws-sdk/client-s3vectors`, `@aws-sdk/client-sqs`, `@aws-sdk/client-iam`, `@aws-sdk/client-sts`.

## Slack

- **Role**: Optional; Slack API client used by `slack/slack.service.ts` for notifications or bot-related calls.
- **Configuration**: `SLACK_BOT_TOKEN` (required by the service when used).

## LLM service

- **Role**: Chat module calls an LLM HTTP API for conversational responses.
- **Configuration**: `LLM_URL` (default `http://llm-service`) in `chat/chat.service.ts`.

## Frontend and CORS

- **Consumers**: **ace-dashboard-frontend** (and optionally other UIs) call the backend for auth and all API operations.
- **CORS**: Allowed origins are `FRONTEND_HOST`, `http://localhost:3042`, `http://localhost:3000`, `http://localhost:8080`, `http://localhost:8081`. Other origins are rejected. `FRONTEND_HOST` is also used for OAuth redirects (login success, provider callback).

## Other ACE services

- **ace-commands-api**: Referenced in project rules as an optional integration; any usage would be via HTTP from backend to commands-api.
- **ace-configuration**: Schema and seed data; the backend accesses configuration data via ace-db-gateway or shared models, not by calling ace-configuration HTTP API.

## Main libraries (summary)

- **NestJS**: core, common, config, jwt, passport, platform-express, typeorm, websockets, platform-socket.io.
- **Passport**: passport-jwt, passport-local, passport-google-oauth20, passport-github2.
- **Database**: typeorm, pg.
- **Real-time**: socket.io, @socket.io/redis-adapter, ioredis, redis.
- **Validation**: class-validator.
- **HTTP**: axios, graphql-request (where used).
- **Queue**: bullmq (if used by a feature).
- **Slack**: @slack/web-api.
