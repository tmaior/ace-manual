# ace-ops-scheduler

Operations scheduler service for ACE. It runs scheduled jobs (regular events and resource health events), ingests webhook payloads, and integrates with ace-db-gateway, ace-stack-backend (omnichannel), and the LLM service.

## Purpose

- **Regular events**: Run on a 60 s loop; due events send payloads to omnichannel and update execution result via DB Gateway.
- **Resource health**: Run **RESOURCE_DISCOVERY** (discover resources from project docs/CI-CD, upsert monitored resources, optional HEALTH_CHECK event creation) and **HEALTH_CHECK** (execute health check scripts via LLM sandbox, update resource status).
- **Payloads**: POST /api/payloads accepts webhooks with project_id/project_name/channel_id and forwards to omnichannel with callback for result updates.
- **Manual trigger and callback**: POST /api/resource-health/trigger for on-demand discovery or health check; POST /scheduler-callback for omnichannel to report execution results.

## Documentation in this folder

| Document | Description |
|----------|-------------|
| [overview.md](./overview.md) | What the app is, architecture, main components, capabilities. |
| [api-and-routes.md](./api-and-routes.md) | All HTTP endpoints (health, events, payloads, resource-discovery, trigger, callback). |
| [events-and-scheduling.md](./events-and-scheduling.md) | Event types, SchedulerBLL, regular and resource health loops, due-time logic. |
| [resource-health.md](./resource-health.md) | Resource discovery pipeline and health check flow, handlers, env/feature flags. |
| [dependencies-and-integrations.md](./dependencies-and-integrations.md) | ace-db-gateway, stack backend, LLM service, Redis, env vars. |

## How to use

- **New to the service**: Read [START_HERE.md](./START_HERE.md), then [overview.md](./overview.md).
- **Implementing or calling APIs**: Use [api-and-routes.md](./api-and-routes.md) and [dependencies-and-integrations.md](./dependencies-and-integrations.md).
- **Understanding scheduling and resource health**: Use [events-and-scheduling.md](./events-and-scheduling.md) and [resource-health.md](./resource-health.md).

## Related

- [Architecture service catalog](../../architecture/service-catalog.md)
- **Repository**: `ace-ops-scheduler/` (sibling to ace-manual)
