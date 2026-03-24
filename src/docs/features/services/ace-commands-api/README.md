# ace-commands-api

Node.js microservice that executes shell commands for the ACE platform: Slack-driven commands, resource health checks, and Knowledge Base docs-sync, with safety checks and project secrets from ace-db-gateway.

## Purpose

- **Slack commands**: Consume SQS (CommandsList), run script with safety check and project env vars, send output to CommandsOutputs for the bot.
- **Resource health**: Consume ResourceHealthChecks queue, run health scripts per resource, send results to ResourceHealthCheckResults for ace-ops-scheduler.
- **Docs-sync**: Optional worker (in-process or separate) consumes DocsSync queue and runs KB sync scripts (clone, S3, Bedrock ingestion).
- **Logging**: All command/queue/message activity logged to MongoDB; optional file-based log for resource health debug.

## Documentation in this folder

| Document | Description |
|----------|-------------|
| [overview.md](./overview.md) | What the app is, architecture, components, capabilities. |
| [dependencies-and-integrations.md](./dependencies-and-integrations.md) | External deps, libraries, integrations (DB Gateway, Slackbot, SQS, Bedrock). |
| [features-and-capabilities.md](./features-and-capabilities.md) | Slack commands, resource health, docs-sync, logging, API, feature flags. |
| [environment-and-configuration.md](./environment-and-configuration.md) | Required/optional env vars, paths, execution env, secrets. |
| [build-deploy-and-cicd.md](./build-deploy-and-cicd.md) | Local build, Docker, Kubernetes, GitHub Actions (dev/prod). |
| [api-and-routes.md](./api-and-routes.md) | Health, logs, resource health debug, dashboard; what is not exposed. |
| [queues-and-workers.md](./queues-and-workers.md) | SQS queues, listeners, docs-sync worker, Queue class. |

## Repository docs

- **ace-commands-api/docs/** – e.g. `docs-sync-worker.md`, `testing/resource-health-testing-guide.md`.

## Related

- [Architecture service catalog](../../architecture/service-catalog.md)
- [Data flow](../../architecture/data-flow.md)
- [Knowledge Base and resources](../../infrastructure/knowledge-base-and-resources.md)
- [Environments](../../environments/) for per-env setup and secrets
