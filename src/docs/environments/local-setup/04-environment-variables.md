# Step 4: Environment Variables

Environment variables are set in `local-env/docker-compose.yaml` (per-service `environment:` blocks) and/or in `local-env/.env`. **Never commit real secrets.** Use placeholders below and obtain real values from the team or from AWS Secrets Manager (e.g. dev secrets) for local use only.

---

## Where variables are used

- **Compose file**: Many values are in the YAML; some use substitution from `.env` (e.g. `${POSTGRES_PASSWORD:-root_password}`, `${ACE_ROOT}`).
- **`.env` in local-env**: Override or provide secrets: `ACE_ROOT`, `POSTGRES_PASSWORD`, `JWT_SECRET`, Slack tokens, API keys, OAuth client secrets, etc. Docker Compose loads `.env` from the project directory (where `docker-compose.yaml` is).

---

## Critical variables (minimum to run)

These are the main variables you must set or verify for a working local stack. Use safe defaults or placeholders; replace with real values only in `.env` (not committed).

| Variable | Used by | Purpose | Example (placeholder) |
|----------|---------|---------|----------------------|
| **ACE_ROOT** | Compose (if paths use it) | Parent directory of all ACE repos | `/path/to/ace` |
| **POSTGRES_PASSWORD** | psql, configuration, db-gateway, dash-back, llm | PostgreSQL password | `root_password` (default in many Compose files) |
| **MONGO_ROOT_PASSWORD** | mongo, commands-api | Mongo root password | `root2025` |
| **BACKEND_DB_GATEWAY_TOKEN** | dash-back, db-gateway | JWT/token for backend to call db-gateway | `BACKEND-DB_GATEWAY-TOKEN` |
| **SCHEDULLER_DB_GATEWAY_TOKEN** | ops-scheduler, db-gateway, ops-bot | Token for scheduler/bot to call db-gateway | `SCHEDULLER-DB_GATEWAY-TOKEN` |
| **ENCRYPTION_KEY** | db-gateway | Encryption key (hex) | Obtain from team; 32-char hex typical |
| **JWT_SECRET** | dash-back | JWT signing secret | e.g. `a1b2c3d4-e5f6-7890-1234-567890abcdef` |
| **UID**, **GID** | All Node services | Run as host user so volume files have correct owner | `1000`, `1000` (default) |

---

## Service-specific variables

### configuration (3030)

- DB and Redis are usually set in the Compose file: `DATABASE_URL`, `DB_HOST`, `DB_USERNAME`, `DB_PASSWORD`, `DB_DATABASE`, `REDIS_HOST`, `REDIS_PORT`, `REDIS_PREFIX`. Match `POSTGRES_PASSWORD` with psql.

### db-gateway (3031)

- `CONFIGURATION_DB_URL`, `LLM_DB_URL`, `BACKEND_DB_GATEWAY_TOKEN`, `SCHEDULLER_DB_GATEWAY_TOKEN`, `ENCRYPTION_KEY`, `REDIS_HOST`. Optional: `SECRET_PATH_PREFIX`, `AWS_*` if using Secrets Manager. For local-only, tokens can be placeholder values.

### slackbot (3033), ops-bot (3038)

- **Required**: `SLACK_BOT_TOKEN`, `SLACK_SIGNING_SECRET` (per bot). Without real Slack app credentials, bots will not connect to Slack; you can still start containers.
- `AI_API_URL=http://llm:8080`, `REDIS_HOST=redis`, `ACE_DB_GATEWAY_ENDPOINT=http://db-gateway:3031`, `STACK_BACKEND_URL`, `SELF_URL` are set in Compose.

### commands-api (3035)

- `MONGO_URL`, `ACE_DB_GATEWAY_ENDPOINT`. For SQS: either LocalStack URLs (e.g. `http://localdash.ace.ezops.cloud:3066/000000000000/...`) or real AWS SQS URLs. `LOCAL_AWS_ENDPOINT` when using LocalStack. Queue URLs must match queues created by `init-scripts/sqs.sh` when using LocalStack.

### ops-scheduler (3036)

- `LLM_API_URI`, `ACE_DB_GATEWAY_URI`, `ACE_DB_GATEWAY_ENDPOINT`, `SCHEDULLER_DB_GATEWAY_TOKEN`, `RESOURCE_HEALTH_CHECKS_QUEUE_URL`, `RESOURCE_HEALTH_CHECK_RESULTS_QUEUE_URL`, `LOCAL_AWS_ENDPOINT`, `REDIS_HOST`. Use LocalStack queue URLs and endpoint when running fully local.

### llm (3040)

- `DATABASE_URL` (PostgreSQL for llm), `REDIS_URL`, `DB_GATEWAY_URL`, `LITELLM_BASE_URL`, `ANTHROPIC_API_KEY`, `GEMINI_API_KEY`, `OPENROUTER_API_KEY`, etc. API keys are required for LLM to call providers; get from team or use placeholders and do not start LLM until you have keys if you need it to work.

### dash-back (3041)

- `DB_*`, `JWT_SECRET`, `ACE_GATEWAY_URL`, `ACE_GATEWAY_TOKEN` (same as BACKEND_DB_GATEWAY_TOKEN), `REDIS_URL`, `FRONTEND_URL`, `LLM_URL`, `QUEUE_DOCS_SYNC_URL`, optional OAuth (`GOOGLE_CLIENT_ID`, `GITHUB_CLIENT_ID`, etc.). Frontend/API URLs use your chosen hostname and ports (3042, 3041, 3031, 3040, 3066); for the agent use **ace-development.ace.ezops.cloud** or the Daytona proxy URL — see [09-hostname-and-dns.md](./09-hostname-and-dns.md) and [Host name and DNS](../local.md#host-name-and-dns).

### dash-front (3042)

- `VITE_API_URL`, `VITE_FRONTEND_URL`, `VITE_ACE_GATEWAY_URL`, `VITE_LLM_URL` — must point to backend, frontend, db-gateway, and LLM as reachable from the **browser** (e.g. `http://ace-development.ace.ezops.cloud:3041` for the agent, or `http://localhost:3041`). See [09-hostname-and-dns.md](./09-hostname-and-dns.md).

### litellm

- API keys (Anthropic, OpenRouter, etc.) and optional `LITELLM_MASTER_KEY`; config file can reference env vars. See [../local.md](../local.md) for config structure.

### LocalStack / SQS

- Queue URLs must be reachable from host and from containers. Use your chosen hostname and port 3066 (e.g. `http://ace-development.ace.ezops.cloud:3066/000000000000/ResourceHealthChecks-local.fifo`) or `localhost:3066`; from another container use `http://localstack:4566/000000000000/...`.

---

## Hostname for URLs (developer vs agent)

Many Compose env vars need a hostname for frontend, API, gateway, LLM, and LocalStack URLs so that the browser and external callbacks work. **Which hostname to use depends on who is running the environment:**

- **Developer (personal machine)**: **`localdash.ace.ezops.cloud`** is often used in the team's local-env; it points to that developer's machine IP. You can use `/etc/hosts` (`127.0.0.1 localdash.ace.ezops.cloud`) or a DNS record.
- **Agent / automated environment**: **Do not** use `localdash.ace.ezops.cloud`. Use either the **Daytona proxy URL** (if running in Daytona) or **`ace-development.ace.ezops.cloud`**. The latter can be registered in **Route53**, in the hosted zone **ace.ezops.cloud**, pointing to the IP of the environment (e.g. Daytona proxy or dev VM). See [09-hostname-and-dns.md](./09-hostname-and-dns.md).
- **localhost**: You can replace the hostname with `localhost` in the Compose file or `.env` for every URL the browser or host uses; keep internal URLs (e.g. `http://dash-back:8080`) as service names for container-to-container communication.

---

## Example .env (no real secrets)

```bash
# Path to parent of ace-* repos and local-env
ACE_ROOT=/path/to/your/ace

# DB (match what is in docker-compose for psql)
POSTGRES_PASSWORD=root_password
MONGO_ROOT_PASSWORD=root2025

# Tokens (obtain from team for real use)
BACKEND_DB_GATEWAY_TOKEN=your-backend-token
SCHEDULLER_DB_GATEWAY_TOKEN=your-scheduler-token
ENCRYPTION_KEY=your-32-char-hex-key
JWT_SECRET=your-jwt-secret

# Optional: Slack (leave empty to skip bot features)
# SLACKBOT_SLACK_BOT_TOKEN=xoxb-...
# SLACKBOT_SLACK_SIGNING_SECRET=...
# OPSBOT_SLACK_BOT_TOKEN=xoxb-...
# OPSBOT_SLACK_SIGNING_SECRET=...

# Optional: LLM (leave empty to skip LLM)
# ANTHROPIC_API_KEY=sk-ant-...
# OPENROUTER_API_KEY=sk-or-...
# GEMINI_API_KEY=...

# Optional: OAuth for dashboard login
# GOOGLE_CLIENT_ID=...
# GOOGLE_CLIENT_SECRET=...
# GITHUB_CLIENT_ID=...
# GITHUB_CLIENT_SECRET=...
```

---

## Reference

- Full list of env vars per service in the team's **local-env** (e.g. `environment-variables.md`) — use as key reference; do not copy real secrets into docs.
- [../env-vars-and-secrets.md](../env-vars-and-secrets.md) for where secrets come from per environment (local vs EKS).

Next: [05-startup-sequence.md](./05-startup-sequence.md) to start services in the correct order.
