# ace-ops-scheduler – Overview

## What it is

**ace-ops-scheduler** is the ACE operations scheduler service. It runs scheduled jobs (events) and handles webhook payloads. It executes two kinds of work: **regular scheduler events** (sent to the omnichannel/LLM pipeline) and **resource health events** (resource discovery and health checks). The service is a Node.js Express application; scheduling is implemented with in-process `setInterval` loops (every 60 seconds), not an external job queue.

## Purpose

- **Regular events**: Fetch scheduler events from ace-db-gateway (excluding resource health), and when due by period/lastRunTime, send payloads to ace-stack-backend omnichannel ingest. Execution result is written back via DB Gateway.
- **Resource health events**: Run **RESOURCE_DISCOVERY** (discover resources from project docs/CI-CD and upsert monitored resources) and **HEALTH_CHECK** (run health check scripts for project resources via LLM/Daytona sandbox and update status in DB Gateway).
- **Payloads webhook**: Accept `POST /api/payloads` with optional `project_id`, `project_name`, or `channel_id`; resolve project and channel, then forward the payload to omnichannel ingest with a scheduler callback URL for result updates.
- **Manual trigger and callbacks**: Expose manual trigger for health/discovery and a callback endpoint used by omnichannel to update event execution results.

## High-level architecture

- **Entry point**: `index.sequelize.js` starts two independent `setInterval` loops (60 s) and an Express server.
- **Loops**: (1) Regular events → `SchedulerBLL.getRegularEvents()` → for each due event, `sendToLLM()` (omnichannel ingest) → `SchedulerBLL.updateEventExecution()`. (2) Resource health events → `SchedulerBLL.getResourceHealthEvents()` → for each due event, either `executeResourceDiscovery()` (fire-and-forget) or `executeHealthChecks()` (LLM execute-health-checks) → update event execution and resource status.
- **HTTP API**: Express routes under `/api/events`, `/api/payloads`, `/health`, `/api/resource-discovery/timing`, `/api/resource-health/trigger`, `/scheduler-callback`; static dashboard under `/dashboard`.
- **Data**: Scheduler events, monitored resources, project/channel/config and knowledge base IDs are read/written via **ace-db-gateway**. Redis is used for KB ID caching and resource-discovery timing; optional JSON file for discovery timing history.

## Main components

| Component | Role |
|-----------|------|
| **index.sequelize.js** | Entry point: two setInterval loops (regular + resource health), Express server listen. |
| **controller/SchedulerBLL.js** | Business logic: getRegularEvents, getResourceHealthEvents, saveEvent, updateEventExecution; uses DBGatewayClient and contextResolver. |
| **routes/index.js** | Mounts `/api/events`, `/api/payloads`; defines `/health`, `/api/resource-discovery/timing`, `/api/resource-health/trigger`, `/scheduler-callback`. |
| **routes/events.js** | POST /api/events: create scheduler event (channel_id, period, check); resolves client_id/project_id via contextResolver. |
| **routes/payloads.js** | POST /api/payloads: webhook ingest; resolve project by project_id, project_name, or channel_id; forward to stack backend omnichannel with callback_url. |
| **handlers/resourceDiscoveryHandler.js** | executeResourceDiscovery (concurrency lock, discoveryOrchestrator), updateResourceDiscoveryEvent (backoff/success period), ensureHealthCheckEventExists. |
| **handlers/healthCheckHandler.js** | executeHealthChecks: fetch resources from DB Gateway, call LLM execute-health-checks, update monitored resource status. |
| **handlers/discoveryOrchestrator.js** | executeDiscovery: project config → CI/CD seeds + agentic exploration → reconcile → script generation → batch upsert; reconcile stale resources (consecutive misses). |
| **handlers/cicdSeedExtractor.js** | Parse infra files (GitHub Actions, Terraform, etc.) to extract resource seeds (no LLM). |
| **services/DBGatewayClient.js** | HTTP client for ace-db-gateway: schedulers CRUD, monitored resources, channels, projects, knowledge base (with Redis cache). |
| **services/llmAIClient.js** | HTTP client for LLM service: generate-scripts, explore-resources, execute-health-checks (Daytona sandbox). |
| **services/ResourceDiscoveryTiming.js** | Redis start time + optional JSON file for discovery timing metrics. |
| **services/githubDocsFetcher.js** | Fetches infra files from project docs repo/branch. |
| **utils/contextResolver.js** | Resolves client_id and project_id from channel_id via DB Gateway. |

## What you can do with this app

- Create scheduler events via POST /api/events (channel_id, period, check).
- Ingest webhook payloads via POST /api/payloads and have them processed by omnichannel with callback to update event execution.
- Trigger resource discovery or health checks manually via POST /api/resource-health/trigger (projectId, type).
- Query resource discovery timing via GET /api/resource-discovery/timing.
- Rely on the two in-process loops to run regular events and resource health events on their configured periods.

## Related documentation

- [API and routes](./api-and-routes.md)
- [Events and scheduling](./events-and-scheduling.md)
- [Resource health (discovery and health checks)](./resource-health.md)
- [Dependencies and integrations](./dependencies-and-integrations.md)
- [Architecture service catalog](../../architecture/service-catalog.md)
