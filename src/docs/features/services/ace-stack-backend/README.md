# ace-stack-backend

## Purpose

This folder holds the **central documentation** for **ace-stack-backend**, the main NestJS API of the ACE platform. The content is derived from the application code and follows the ace-manual structure (index, START_HERE, README in every directory; docs only under `src/docs/features`).

## What you will find

| Document | Purpose |
|----------|---------|
| **START_HERE.md** | Entry point: list of all docs in this folder with short descriptions. |
| **index.md** | Simple list of files and links (no descriptions). |
| **overview.md** | What the service is, purpose, architecture, main components, and what you can do with it. |
| **dependencies-and-integrations.md** | ace-db-gateway, Redis, PostgreSQL, AWS, Slack, LLM, frontend, and main libraries. |
| **features-and-capabilities.md** | Auth, admin, user, channels, configurations, monitored resources, documentations, chat, omnichannel, integrations, secrets, notifications, security. |
| **api-endpoints.md** | HTTP routes and WebSocket path by controller. |
| **environment-and-configuration.md** | Env vars, bootstrap (dotenv vs AWS Secrets Manager), DB, OAuth, AWS, CORS. |
| **build-deploy-and-cicd.md** | Local build/run, migrations, Docker/Kubernetes, CI/CD. |

## How to use

1. **New to the service**: Start with [START_HERE.md](./START_HERE.md), then read [overview.md](./overview.md) and [environment-and-configuration.md](./environment-and-configuration.md) for local setup.
2. **Integrating or calling the API**: Use [api-endpoints.md](./api-endpoints.md) and [features-and-capabilities.md](./features-and-capabilities.md).
3. **Deploying or debugging**: Use [build-deploy-and-cicd.md](./build-deploy-and-cicd.md) and [dependencies-and-integrations.md](./dependencies-and-integrations.md).

## Related

- **Repository**: `ace-stack-backend/` (sibling to ace-manual). Prefer this documentation over outdated repo `docs/` when they conflict.
- **Architecture**: [../../architecture/service-catalog.md](../../architecture/service-catalog.md), [../../architecture/data-flow.md](../../architecture/data-flow.md), [../../architecture/security.md](../../architecture/security.md).
