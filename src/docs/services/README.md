# Services

Per-service documentation for ACE microservices.

## Purpose

This directory holds documentation that spans or summarizes multiple services, or points to service-specific docs:

- Index of all services and their responsibilities
- Cross-service flows and integration points
- Links to each service's own `docs/` (e.g. `ace-db-gateway/docs/`, `ace-stack-backend/docs/`)
- Common patterns (auth, logging, error handling) across services

## Service List (reference)

| Service | Role |
|--------|------|
| ace-dashboard-frontend | React/Vite UI |
| ace-stack-backend | NestJS API, JWT auth |
| ace-db-gateway | Database gateway |
| ace-configuration | Configuration management |
| ace-infra | Terraform, K8s, AWS |
| ace-slackbot | Slack bot |
| ace-sec-bot | Security bot |
| ace-ops-bot | Operations bot |
| ace-commands-api | Commands API |
| ace-ops-scheduler | Operations scheduler |

---

*Add service overview and cross-service docs here; detailed API/setup stay in each repo's `docs/`.*
