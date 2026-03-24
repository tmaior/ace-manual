# Start Here – ace-stack-backend

**ace-stack-backend** is the NestJS API for the ACE platform: authentication (JWT and OAuth), business logic, and orchestration. It calls **ace-db-gateway** for data and uses Redis for WebSockets and OAuth state. Documentation in this folder is based on the application code and is the central reference for this service.

## Contents

- **[README.md](./README.md)** – Overview of this folder, purpose, and how to use the docs.
- **[index.md](./index.md)** – Simple list of contents of this directory.
- **[overview.md](./overview.md)** – What the app is, purpose, high-level architecture, main components, and what you can do with it.
- **[dependencies-and-integrations.md](./dependencies-and-integrations.md)** – ace-db-gateway, Redis, PostgreSQL (TypeORM), AWS (Secrets Manager, SES, S3, Bedrock), Slack, LLM service, frontend, and main libraries.
- **[features-and-capabilities.md](./features-and-capabilities.md)** – Auth (JWT, local, Google, GitHub), admin API, user API, channels, configurations, monitored resources, documentations, chat, omnichannel (REST + WebSocket), integrations, secrets, notifications, logging, migrations, security.
- **[api-endpoints.md](./api-endpoints.md)** – Summary of HTTP routes and WebSocket path by controller (auth, admin, users, external-linkages, documentations, chat, monitored-resources, omnichannel, webhooks, secrets, integrations, commands-history).
- **[environment-and-configuration.md](./environment-and-configuration.md)** – Bootstrap (dotenv vs AWS Secrets Manager), required and optional env vars (PORT, JWT, DB, REDIS, ACE_GATEWAY_URL, OAuth, AWS, SES, FRONTEND_HOST, API_HOST), environment-specific notes, secrets.
- **[build-deploy-and-cicd.md](./build-deploy-and-cicd.md)** – Local build and run, TypeORM migrations, lint/format, Docker/Kubernetes (ace-infra), CI/CD (GitHub Actions), notes for local and EKS.

## Where to find more

- **Repository**: `ace-stack-backend/` (sibling to ace-manual). Source code, migrations, and repo-specific config; prefer this documentation over outdated repo `docs/` when they differ.
- **Architecture**: [../../architecture/service-catalog.md](../../architecture/service-catalog.md), [../../architecture/data-flow.md](../../architecture/data-flow.md), [../../architecture/security.md](../../architecture/security.md).
- **Environments**: [../../environments/](../../environments/) for local, demo, and production setup.
