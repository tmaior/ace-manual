# Environment Variables and Secrets

This document describes **where environment variables and secrets come from** in each ACE environment and lists important vars by service or concern. Never commit secrets or hardcode them in code or config.

---

## Where vars come from per environment

| Environment | Source of env vars | Notes |
|-------------|--------------------|--------|
| **Local** | `.env` files, `docker-compose` env section, or shell export | Copy from `.env.example`; do not commit `.env` with secrets. |
| **Development (dev)** | AWS Secrets Manager path **`ace/dev/<service>-secrets`** | Injected into pods (e.g. by CI/CD or K8s external secrets). |
| **Staging (stg)** | AWS Secrets Manager path **`ace/stg/<service>-secrets`** | Same injection pattern as dev. |
| **Demo** | **`ace/demo/<service>-secrets`** | Injected into pods. Demo apps use **production** RDS, Redis, DB Gateway, SQS; see [demo.md](./demo.md). |
| **Production (prod)** | AWS Secrets Manager path **`ace/prod/<service>-secrets`** | Injected into pods; never in repo or workflow content. |

**Rule**: Secrets Manager paths **`ace/<env>/<service>-secrets`** are **only for environment variables** that the application uses at runtime. They are not for arbitrary config or data. See [../rules/infrastructure-rules.md](../rules/infrastructure-rules.md).

---

## CI/CD secrets

- **GitHub Environments** — deploy credentials, GitHub tokens, and other pipeline secrets. Use the Environment (e.g. `development`, `staging`, `production`) to scope secrets.
- **Do not** store production (or sensitive) secrets in repository variables or in workflow file content. See [../rules/infrastructure-rules.md](../rules/infrastructure-rules.md) and [../rules/security-rules.md](../rules/security-rules.md).

---

## Important vars by concern

### Backend (ace-stack-backend)

| Var (example) | Purpose | Local | EKS |
|---------------|---------|-------|-----|
| **`ACE_GATEWAY_URL`** (primary in ace-stack-backend) | DB Gateway base URL | e.g. `http://localhost:4xxx` | From secrets |
| `DB_GATEWAY_URL` | Legacy/alternate name in some services or compose | Same as gateway URL | Prefer **`ACE_GATEWAY_URL`** for backend consistency |
| `REDIS_HOST`, `REDIS_PORT` | Redis for omnichannel, cache | e.g. `localhost`, `6379` | From secrets |
| JWT secret / auth vars | Token signing and validation | .env | From secrets |
| DB connection (if any direct) | Backend DB; often via DB Gateway only | .env | From secrets |

### DB Gateway (ace-db-gateway)

| Var (example) | Purpose | Local | EKS |
|---------------|---------|-------|-----|
| DB connection strings | PostgreSQL (or other DBs) | .env | From secrets |
| JWT validation config | Validate tokens from backend | .env | From secrets |

### Frontend (ace-dashboard-frontend)

| Var (example) | Purpose | Local | EKS |
|---------------|---------|-------|-----|
| `VITE_API_URL` (or equivalent) | Backend API base URL | e.g. `http://localhost:3000` | Build-time / deploy config |

### Bots (ace-slackbot, ace-sec-bot, ace-ops-bot)

| Var (example) | Purpose | Local | EKS |
|---------------|---------|-------|-----|
| `REDIS_HOST`, `REDIS_PORT` | Session store | e.g. `redis`, `6379` | From secrets |
| Slack tokens / signing secret | Slack API | .env | From secrets |
| Backend or Commands API URL | Call backend/commands-api | .env | From secrets |

### ace-ops-scheduler

| Var (example) | Purpose | Local | EKS |
|---------------|---------|-------|-----|
| `REDIS_HOST` | Cache (e.g. KnowledgeBase) | .env | From secrets |
| Backend or API URLs | Trigger or call services | .env | From secrets |

---

## Conventions

- **Base URLs**: Always from **environment variables**; never hardcode full URLs or hosts in code or config that can differ per environment. See [../rules/standardization-rules.md](../rules/standardization-rules.md).
- **Naming**: Use clear, consistent names (e.g. `ACE_GATEWAY_URL`, `REDIS_HOST`). Document new vars in the service’s docs and in this file when they are shared or critical.
- **Adding a new service**: Define the required keys and the Secrets Manager path (e.g. `ace/dev/<new-service>-secrets`) in the service README or ace-infra; add a row to the table in the relevant env doc and here if needed.

---

## Links

- **Infrastructure rules** — [../rules/infrastructure-rules.md](../rules/infrastructure-rules.md) (region, tags, secrets path)
- **Security rules** — [../rules/security-rules.md](../rules/security-rules.md) (no commit/hardcode, validation)
- **Architecture deployment** — [../architecture/deployment.md](../architecture/deployment.md)
- **Per-environment docs** — [local.md](./local.md), [development.md](./development.md), [staging.md](./staging.md), [demo.md](./demo.md), [production.md](./production.md)
