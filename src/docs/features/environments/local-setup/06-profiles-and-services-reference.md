# Step 6: Profiles and Services Reference

Docker Compose in local-env uses **profiles** so you can start only the services you need. This document lists profiles and which services belong to each, and summarizes ports and dependencies.

---

## Profiles (summary)

| Profile | Use case | Main services included |
|---------|----------|-------------------------|
| **database** | DBs and tools only | psql, pgadmin, redis, redisinsight, mongo, configuration |
| **infra** | LocalStack (SQS) | localstack (+ init-scripts) |
| **application** / **dashboard** | Dashboard (front + back) | dash-back, dash-front + database deps; dashboard also adds Redis, configuration, redisinsight, llm |
| **apps** | All ACE apps (no Jira) | configuration, db-gateway, slackbot, commands-api, ops-scheduler, ops-bot, llm, dash-back, dash-front, litellm |
| **bot** | Bots only | slackbot, ops-bot + db-gateway, redis, llm |
| **scheduler** | Scheduler + commands-api | ops-scheduler, commands-api + db-gateway, redis, localstack |
| **llm** | LLM stack | llm, litellm + psql, redis |
| **ops** | Ops scheduler and deps | ops-scheduler + db-gateway, redis, localstack |
| **tool** | Dev tools | pgadmin, redisinsight, configuration |
| **jira** | Jira integration | jira app, nginx (HTTPS) |
| **cert** | One-shot TLS cert | letsencrypt (Route53 wildcard for Jira) |
| **all** | Full stack | Everything: database, infra, apps, dashboard, bots, llm, jira (if configured) |

---

## Services by port (quick reference)

| Port (host) | Service | Image / build | Notes |
|-------------|---------|----------------|-------|
| 3017 | mongo | mongo | MongoDB (commands-api) |
| 3030 | configuration | node:20 | Configuration service; health at /health |
| 3031 | db-gateway | node:20 | Central DB gateway |
| 3033 | slackbot | node:20 | Slack bot |
| 3035 | commands-api | build (ace-infra) | Commands API; needs Mongo, LocalStack or AWS SQS |
| 3036 | ops-scheduler | node:20 | Ops scheduler; depends on localstack |
| 3038 | ops-bot | node:20 | Ops Slack bot |
| 3040 | llm | build (ezrael-bot-llm + ace-infra) | LLM service |
| 3041 | dash-back | node:20 | NestJS backend (dashboard API) |
| 3042 | dash-front | node:20 | React/Vite dashboard (preview port 4173) |
| 3055 | jira | node:20 | ACE–Jira app (optional) |
| 3066 | localstack | localstack/localstack | SQS (and other AWS services if enabled) |
| 3080 | pgadmin | dpage/pgadmin4 | PostgreSQL UI |
| 3100 | litellm | ghcr.io/berriai/litellm | LLM proxy (optional) |
| 3379 | redis | redis:7-alpine | Redis |
| 3432 | psql | postgres:17 | PostgreSQL |
| 3540 | redisinsight | redis/redisinsight | Redis UI |
| 80, 443 | nginx | nginx:alpine | Reverse proxy for Jira (optional) |

---

## Dependencies (order matters)

- **configuration**: Uses psql and redis; no Compose `depends_on` in some setups, but DB must exist.
- **db-gateway**: Uses configuration DB, LLM DB, redis; can start after configuration.
- **ops-scheduler**: **depends_on: localstack**; LocalStack must be up and init scripts must have created queues.
- **commands-api**: Uses Mongo, db-gateway, and SQS (LocalStack or AWS); start after LocalStack if using LocalStack.
- **dash-back**: Uses psql (dashboard DB), redis, db-gateway; start after db-gateway.
- **dash-front**: Build/runtime uses env vars pointing to dash-back, db-gateway, LLM; start after dash-back if you need full UI.
- **slackbot**, **ops-bot**: Use redis, db-gateway, llm; start after db-gateway and llm if you need bot features.
- **llm**: Uses psql (llmdatabase), redis, db-gateway, litellm; start after db-gateway and optionally litellm.

---

## Minimal vs full

- **Minimal (dashboard only)**: `docker compose --profile dashboard up -d` — database + configuration + db-gateway + dash-back + dash-front + redis + redisinsight (+ llm in some profiles). Good for UI and API work.
- **Full (equal to local-env)**: `docker compose --profile all up -d` — everything including bots, scheduler, commands-api, LocalStack, LLM, LiteLLM, optional Jira/nginx.

Use [05-startup-sequence.md](./05-startup-sequence.md) for the exact startup order and [07-health-checks-and-validation.md](./07-health-checks-and-validation.md) to verify.
