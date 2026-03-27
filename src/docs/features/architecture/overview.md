# ACE System Overview

This document is the single source of truth for **what ACE is** and **how it is structured**. Use it to understand boundaries, main components, and where each piece of the system lives.

---

## What is ACE?

**ACE** (Automated Cloud Engineer) is a **distributed system of microservices** that provides:

- A **dashboard** (React/Vite frontend) for management and visibility
- A **backend** (NestJS) for business logic and authentication (JWT)
- A **DB Gateway** for centralized, secure database access
- **Bot services** (Slack, security, operations) and supporting APIs
- **Infrastructure** (Terraform, Kubernetes, AWS) for deployment
- **Jira integration** (Atlassian Connect) for issue/comment-driven workflows into ACE

Most user-facing and service-to-service traffic uses **REST**. Automation also uses **message queues (e.g. SQS)**, **Slack events**, and **Jira webhooks**. Dashboard login and API access use **JWT** issued via **ace-stack-backend** (which validates credentials with **ace-db-gateway**); **ace-db-gateway** and other protected services validate JWT or internal tokens as designed.

---

## Main Boundaries

| Boundary | Description | Main repos / components |
|----------|-------------|--------------------------|
| **Frontend** | User-facing dashboard; calls backend APIs | ace-dashboard-frontend |
| **Backend** | Auth (JWT), business logic, orchestration; calls db-gateway and other services | ace-stack-backend |
| **Data access** | Centralized DB access; all DB traffic from backend (or authorized services) goes through it | ace-db-gateway |
| **Bots** | Slack bots and operational/security automation; may call backend or commands-api | ace-slackbot, ace-sec-bot, ace-ops-bot |
| **Supporting APIs** | Commands, schema/migrations, scheduling | ace-commands-api, ace-configuration (migrations/models), ace-ops-scheduler |
| **Jira → ACE** | Connect app: Jira webhooks, project links, payloads to scheduler | ace-jira-integration |
| **Infrastructure** | Terraform, K8s manifests, AWS (EKS, ECR, Secrets Manager) | ace-infra, local-env |

---

## High-Level Structure

See [diagrams.md](./diagrams.md) for Mermaid component and deployment diagrams. In short:

- **User** → **Frontend** → **Backend** (JWT issued after login).
- **Backend** → **DB Gateway** (with JWT) → **Database(s)**.
- **Backend** reads and writes project data through **ace-db-gateway** (including configuration rows such as `/api/configurations`) and may call other internal HTTP APIs (e.g. **ace-commands-api**) as needed. **ace-configuration** provides schema/migrations/models—not a separate REST surface for dashboard config in normal flows.
- **Jira** sends webhooks to **ace-jira-integration** → **ace-db-gateway** (project–Jira links) → **ace-ops-scheduler** (`/api/payloads`) for linked projects.
- **Bots** interact via Slack; they may call **Backend** or **Commands API** with appropriate auth.

---

## Service List (one-line purpose)

| Service | Purpose |
|---------|---------|
| ace-dashboard-frontend | React/Vite UI for ACE; user and admin flows |
| ace-stack-backend | NestJS API, auth (JWT), business logic, orchestration |
| ace-db-gateway | Centralized DB access; validates JWT on every request |
| ace-configuration | Configuration management |
| ace-slackbot | Slack bot service |
| ace-sec-bot | Security bot service |
| ace-ops-bot | Operations bot service |
| ace-commands-api | Commands API for bots and automation |
| ace-ops-scheduler | Operations scheduler service |
| ace-jira-integration | Jira Connect app; webhooks and forwarding to ops-scheduler (required for Jira-driven ACE) |
| ace-infra | Terraform, K8s, AWS; deployment and infra as code |
| local-env | Local development configs (e.g. docker-compose) |

Deprecated (do not use for new work): ace-docs-api, ace-passwordbot.

---

## Where to Go Next

- **Full service details** (ports, env vars, dependencies): [service-catalog.md](./service-catalog.md)
- **How data and requests flow** (auth, sequences): [data-flow.md](./data-flow.md)
- **How services integrate** (URLs, JWT, contracts): [integrations.md](./integrations.md)
- **Where ACE runs** (local, K8s, AWS): [deployment.md](./deployment.md)
- **Security model**: [security.md](./security.md)
