# Start Here – Services

This directory holds **per-service documentation** for every ACE app. Each subfolder is dedicated to one service and contains a short overview and links to the service repository docs. Detailed API, setup, and debugging stay in each repo's `docs/` folder.

---

## Subdirectories

**[ace-dashboard-frontend/](./ace-dashboard-frontend/)**  
React/Vite UI; calls backend APIs. Entry point for user and admin flows.

**[ace-stack-backend/](./ace-stack-backend/)**  
NestJS API: JWT auth, business logic, orchestration; calls db-gateway and other services.

**[ace-db-gateway/](./ace-db-gateway/)**  
Centralized DB access; validates JWT on every request.

**[ace-configuration/](./ace-configuration/)**  
Configuration management service.

**[ace-slackbot/](./ace-slackbot/)**  
Slack bot; sessions via Redis; may call backend or commands-api.

**[ace-sec-bot/](./ace-sec-bot/)**  
Security bot service (Slack, Redis, APIs).

**[ace-ops-bot/](./ace-ops-bot/)**  
Operations bot service (Slack, Redis, APIs).

**[ace-commands-api/](./ace-commands-api/)**  
Commands API for bots and automation.

**[ace-ops-scheduler/](./ace-ops-scheduler/)**  
Operations scheduler service.

**[ace-infra/](./ace-infra/)**  
Infrastructure as code: Terraform, K8s, AWS; see also [../infrastructure/](../infrastructure/).

**[local-env/](./local-env/)**  
Local development setup (docker-compose, configs); see also [../environments/local.md](../environments/local.md).

---

## Files

**[README.md](./README.md)**  
Overview of this directory: purpose, service list, and how to use it.

**[index.md](./index.md)**  
Simple list of contents (files and subdirectories).

---

*For architecture and service catalog, see [../architecture/](../architecture/). For deployment and environments, see [../environments/](../environments/).*
