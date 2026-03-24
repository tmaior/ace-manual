# Start Here – ace-ops-scheduler

**ace-ops-scheduler** is the ACE operations scheduler service. It runs regular scheduler events (via omnichannel) and resource health events (discovery and health checks), and exposes webhook payload ingest and manual triggers.

## Contents

- **[README.md](./README.md)** – Overview of this folder, purpose, and how to use the documentation.
- **[index.md](./index.md)** – Simple list of contents of this directory.
- **[overview.md](./overview.md)** – What the service is, purpose, high-level architecture, and main components.
- **[api-and-routes.md](./api-and-routes.md)** – All HTTP endpoints: health, events, payloads, resource-discovery timing, manual trigger, scheduler callback.
- **[events-and-scheduling.md](./events-and-scheduling.md)** – How regular and resource health events are scheduled (two setInterval loops, SchedulerBLL, due-time logic).
- **[resource-health.md](./resource-health.md)** – Resource discovery pipeline (CI/CD seeds, agentic exploration, reconcile, batch upsert) and health checks (LLM execute-health-checks, status updates).
- **[dependencies-and-integrations.md](./dependencies-and-integrations.md)** – ace-db-gateway, ace-stack-backend (omnichannel), LLM service, Redis, and environment variables.

## Where to find more

- **Repository**: `ace-ops-scheduler/` (sibling to ace-manual). Implementation and repo-specific docs.
- **Architecture**: [../../architecture/service-catalog.md](../../architecture/service-catalog.md).
