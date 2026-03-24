# Start Here – ace-commands-api

**ace-commands-api** is the ACE service that securely executes shell commands: from Slack (via SQS), for resource health checks, and for Knowledge Base docs-sync. It talks to ace-db-gateway (secrets, debug, safety level), ace-slackbot (notifications), and AWS (SQS, optional Bedrock for safety).

## Contents

- **[README.md](./README.md)** – Overview of this folder and links to all docs.
- **[overview.md](./overview.md)** – What the app is, purpose, high-level architecture, main components, and what you can do with it.
- **[dependencies-and-integrations.md](./dependencies-and-integrations.md)** – External dependencies (MongoDB, SQS, DB Gateway, Slackbot, Safety API/Bedrock), main libraries, and who the app talks to.
- **[features-and-capabilities.md](./features-and-capabilities.md)** – Slack command execution, resource health checks, docs-sync worker, logging, HTTP API, feature flags, graceful shutdown.
- **[environment-and-configuration.md](./environment-and-configuration.md)** – Required and optional env vars, paths, execution environment, MongoDB and secrets.
- **[build-deploy-and-cicd.md](./build-deploy-and-cicd.md)** – Local build, Docker image (ace-infra), Kubernetes deploy, GitHub Actions (dev and prod).
- **[api-and-routes.md](./api-and-routes.md)** – Health, logs, resource health debug endpoints, dashboard, and what is not exposed.
- **[queues-and-workers.md](./queues-and-workers.md)** – SQS queues (CommandsList, CommandsOutputs, ResourceHealthChecks, ResourceHealthCheckResults, DocsSync), main process listeners, docs-sync worker, Queue class.
- **[index.md](./index.md)** – Simple list of contents of this directory.

## Where to find more

- **Repository**: `ace-commands-api/` (sibling to ace-manual). Code, tests, and repo-specific docs (e.g. `docs/docs-sync-worker.md`, `docs/testing/resource-health-testing-guide.md`).
- **Architecture**: [../../architecture/service-catalog.md](../../architecture/service-catalog.md), [../../architecture/data-flow.md](../../architecture/data-flow.md).
- **Infrastructure (Knowledge Base, DocsSync queue)**: [../../infrastructure/knowledge-base-and-resources.md](../../infrastructure/knowledge-base-and-resources.md).
