# ACE Repository URLs

This document lists the **Git repository URLs** for each ACE component. Use it to clone repos or open them in the browser. All repositories are under the **ezops-br** organization on GitHub.

Clone with HTTPS or SSH depending on your setup. For local development, clone all required repos into a common parent directory (e.g. `~/ace`); see [environments/local.md](./environments/local.md).

---

## Active services

| Repository | Purpose | URL |
|------------|---------|-----|
| ace-dashboard-frontend | React/Vite UI; user and admin flows | https://github.com/ezops-br/ace-dashboard-frontend |
| ace-stack-backend | NestJS API; JWT auth, business logic, orchestration | https://github.com/ezops-br/ace-stack-backend |
| ace-db-gateway | Centralized DB access; validates JWT | https://github.com/ezops-br/ace-db-gateway |
| ace-configuration | Configuration management | https://github.com/ezops-br/ace-configuration |
| ace-slackbot | Slack bot; sessions via Redis | https://github.com/ezops-br/ace-slackbot |
| ace-sec-bot | Security bot service | https://github.com/ezops-br/ace-sec-bot |
| ace-ops-bot | Operations bot service | https://github.com/ezops-br/ace-ops-bot |
| ace-commands-api | Commands API for bots and automation | https://github.com/ezops-br/ace-commands-api |
| ace-ops-scheduler | Operations scheduler service | https://github.com/ezops-br/ace-ops-scheduler |
| ace-jira-integration | Atlassian Connect app for Jira (webhooks → db-gateway → ops-scheduler); **required** for Jira-driven ACE capabilities | https://github.com/ezops-br/ace-jira-integration |

---

## Infrastructure and documentation

| Repository | Purpose | URL |
|------------|---------|-----|
| ace-infra | Terraform, Kubernetes, AWS; Dockerfiles; deployment manifests | https://github.com/ezops-br/ace-infra |
| ace-manual | Central system documentation (this repo) | https://github.com/ezops-br/ace-manual |

---

## Local development

| Item | Notes | URL |
|------|-------|-----|
| local-env | Local docker-compose and configs. May live under ace-infra or at ACE root; see [environments/local.md](./environments/local.md). | (part of ace-infra or local setup) |

---

## Deprecated (do not use for new work)

| Repository | Notes | URL |
|------------|-------|-----|
| ace-docs-api | Documentation API; deprecated | (see org; repo may be archived or renamed) |
| ace-passwordbot | Password management bot; deprecated | https://github.com/ezops-br/ace-passwordbot |

---

## Clone all active repos (example)

From a parent directory (e.g. `~/ace`):

```bash
git clone https://github.com/ezops-br/ace-dashboard-frontend.git
git clone https://github.com/ezops-br/ace-stack-backend.git
git clone https://github.com/ezops-br/ace-db-gateway.git
git clone https://github.com/ezops-br/ace-configuration.git
git clone https://github.com/ezops-br/ace-slackbot.git
git clone https://github.com/ezops-br/ace-sec-bot.git
git clone https://github.com/ezops-br/ace-ops-bot.git
git clone https://github.com/ezops-br/ace-commands-api.git
git clone https://github.com/ezops-br/ace-ops-scheduler.git
git clone https://github.com/ezops-br/ace-jira-integration.git
git clone https://github.com/ezops-br/ace-infra.git
git clone https://github.com/ezops-br/ace-manual.git
```

SSH alternative: replace `https://github.com/ezops-br/REPO` with `git@github.com:ezops-br/REPO.git` if you use SSH keys.

