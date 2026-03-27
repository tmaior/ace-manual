# ACE Service Catalog

Catalog of every ACE service that agents and developers may need to touch. For each service: name, repository (or path), tech stack, main responsibility, default port when applicable, key env vars, and dependencies (which services it calls).

---

## Active Services

| Service | Repo / path | Tech stack | Main responsibility | Default port | Key env vars | Dependencies |
|---------|-------------|------------|---------------------|-------------|--------------|--------------|
| ace-dashboard-frontend | ace-dashboard-frontend/ | React, TypeScript, Vite | User-facing UI; calls backend APIs | (dev server varies) | VITE_API_URL, backend base URL | ace-stack-backend |
| ace-stack-backend | ace-stack-backend/ | NestJS, Node.js, TypeScript | Auth (JWT), business logic, orchestration | 3000 (typical) | **ACE_GATEWAY_URL** (ace-db-gateway), DB, JWT, OAuth secrets, etc. | ace-db-gateway (incl. `/api/configurations`), optionally ace-commands-api |
| ace-db-gateway | ace-db-gateway/ | Node.js, TypeScript | Centralized DB access; validates JWT | (per deploy) | DB connections, JWT validation | PostgreSQL / DBs |
| ace-configuration | ace-configuration/ | (see repo) | Configuration management | (per deploy) | (see service docs) | (see repo) |
| ace-slackbot | ace-slackbot/ | Node.js, Slack API, Redis | Slack bot; sessions via Redis | (per deploy) | Slack tokens, Redis, backend/commands URL | ace-stack-backend and/or ace-commands-api |
| ace-sec-bot | ace-sec-bot/ | Node.js, Slack API, Redis | Security bot | (per deploy) | Slack, Redis, APIs | (see repo) |
| ace-ops-bot | ace-ops-bot/ | Node.js, Slack API, Redis | Operations bot | (per deploy) | Slack, Redis, APIs | (see repo) |
| ace-commands-api | ace-commands-api/ | Node.js, TypeScript | Commands API for bots and automation | (per deploy) | (see service docs) | (see repo) |
| ace-ops-scheduler | ace-ops-scheduler/ | (see repo) | Operations scheduler | (per deploy) | (see service docs) | ace-db-gateway, ace-commands-api; consumes payloads (e.g. from Jira flow) |
| ace-jira-integration | ace-jira-integration/ | Node.js, Express, Atlassian Connect | Jira webhooks; project links via db-gateway; forward payloads to ops-scheduler | (per deploy) | APP_URL, DATABASE_URL, ACE_DB_GATEWAY_*, OPS_SCHEDULER_URL | ace-db-gateway, ace-ops-scheduler, Jira Cloud |

---

## Infrastructure and Local

| Item | Repo / path | Tech stack | Main responsibility | Key env vars / notes |
|------|-------------|------------|---------------------|----------------------|
| ace-infra | ace-infra/ | Terraform, Kubernetes, AWS | Infra as code; EKS, ECR, manifests per service | (Terraform vars, K8s namespaces) |

**Local development**: docker-compose and configs are documented in [environments/local.md](../environments/local.md), not as a separate service repo.

---

## Deprecated (do not use for new work)

- **ace-docs-api** – Documentation API; deprecated.
- **ace-passwordbot** – Password management bot; deprecated.

---

## Notes for Agents

- **Ports and env vars**: Default ports and env names can vary by environment; check each service’s `docs/` or `.env.example` in the repo. This table gives a quick reference; service-level docs are authoritative.
- **Dependencies**: “Dependencies” means which other ACE services (or external systems) this service calls. When you change an API or contract, update the caller and the docs.
- **New service**: When adding a new service, add a row here and update [overview.md](./overview.md), [data-flow.md](./data-flow.md), [integrations.md](./integrations.md), [deployment.md](./deployment.md), and the architecture index/START_HERE/README.
