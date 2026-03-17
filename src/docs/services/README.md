# Services

Per-service documentation for ACE microservices.

## Purpose

This directory holds **one subfolder per ACE app**, each with a short overview and links to the service repository. It provides:

- Index of all services and their responsibilities
- Cross-service flows and integration points (see [architecture/](../architecture/))
- Links to each service's own `docs/` (e.g. `ace-db-gateway/docs/`, `ace-stack-backend/docs/`)
- Common patterns (auth, logging, error handling) in [rules/](../rules/) and architecture docs

## Service subdirectories

| Service | Role | Folder |
|--------|------|--------|
| ace-dashboard-frontend | React/Vite UI | [ace-dashboard-frontend/](./ace-dashboard-frontend/) |
| ace-stack-backend | NestJS API, JWT auth | [ace-stack-backend/](./ace-stack-backend/) |
| ace-db-gateway | Database gateway | [ace-db-gateway/](./ace-db-gateway/) |
| ace-configuration | Configuration management | [ace-configuration/](./ace-configuration/) |
| ace-slackbot | Slack bot | [ace-slackbot/](./ace-slackbot/) |
| ace-sec-bot | Security bot | [ace-sec-bot/](./ace-sec-bot/) |
| ace-ops-bot | Operations bot | [ace-ops-bot/](./ace-ops-bot/) |
| ace-commands-api | Commands API | [ace-commands-api/](./ace-commands-api/) |
| ace-ops-scheduler | Operations scheduler | [ace-ops-scheduler/](./ace-ops-scheduler/) |
| ace-jira-integration | Jira Connect (webhooks, payloads) | [ace-jira-integration/](./ace-jira-integration/) |
| ace-infra | Terraform, K8s, AWS | [ace-infra/](./ace-infra/) |

Local development (docker-compose, configs) is documented in [environments/local.md](../environments/local.md), not as a service subfolder.

## How to use

- **Find a service**: Open the subfolder (e.g. [ace-db-gateway/](./ace-db-gateway/)) and read START_HERE.md, then the service repo `docs/` for details.
- **Add a new service**: Create a new subfolder with **index.md**, **START_HERE.md**, and **README.md** (see [REPO_RULES.md](../REPO_RULES.md)), then update this README, [index.md](./index.md), and [START_HERE.md](./START_HERE.md).

---

*Detailed API/setup stay in each repo's `docs/`. Architecture and catalog: [architecture/service-catalog.md](../architecture/service-catalog.md).*
