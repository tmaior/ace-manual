# Local Development Setup

This document describes how to run the full ACE stack **locally** using **local-env** (Docker Compose and configs). Following it allows you to bring up an environment equivalent to the one used by the team.

---

## Prerequisites

- **Docker** and **Docker Compose** (Compose V2: `docker compose`) — required for all services.
- **Node.js 20** — used by app containers (node:20); optional on host if you run services outside Docker.
- **Git** — to clone ACE repos.
- **ACE repos** — clone all required repos in a **common parent directory** (e.g. `~/ace`). The Compose file in local-env uses **absolute host paths** (e.g. `/home/admin/ace/ace-configuration`). If your parent directory is different, either copy local-env under the same path or do a find-and-replace in `docker-compose.yaml` for the paths to each repo.

Required sibling repos (relative to `local-env`):

- ace-configuration, ace-db-gateway, ace-stack-backend, ace-dashboard-frontend  
- ace-slackbot, ace-ops-bot, ace-commands-api, ace-ops-scheduler  
- **ace-jira-integration** (clone at the same parent level as other ACE repos; **required** for Jira→ACE webhooks and omnichannel payloads in real environments; local Compose may mount or reference it per `docker-compose.yaml`)  
- ace-infra (for Dockerfiles and commands-api build)  
- ezrael-bot-llm (for LLM service build)  
- **Jira local app**: `local-env/jira-app/meu-app-connect` plus nginx and letsencrypt in local-env for HTTPS (see Compose profiles, e.g. **jira**).
- Optional: ace-sec-bot; litellm-config.yaml (for litellm service)

---

## Where configs live

| Item | Location | Purpose |
|------|----------|---------|
| **Docker Compose** | `local-env/docker-compose.yaml` | Defines all services, profiles, ports, env, volumes, builds. |
| **Env file** | `local-env/.env` | Overrides for `CERT_DOMAIN`, `AWS_*`, `UID`, `GID`. **Do not commit secrets.** |
| **Env reference** | `local-env/environment-variables.md` | List of env vars per service (reference only; values from team/secrets). |
| **Init scripts** | `local-env/init-scripts/` | Scripts run when LocalStack is ready (e.g. create SQS queues). |
| **Nginx (Jira)** | `local-env/volumes/nginx/` | TLS certs and `nginx.conf.template` for reverse proxy (ACE–Jira integration over HTTPS). |
| **LiteLLM config** | e.g. `litellm-config.yaml` in repo root | Mounted into `litellm` service (used by LLM). |

---

## Host name and DNS

Many services need a hostname for callbacks and frontend/API URLs (e.g. `VITE_API_URL`, `FRONTEND_URL`, `QUEUE_DOCS_SYNC_URL`). Which hostname to use depends on who runs the environment:

- **Developer (personal machine)**: **`localdash.ace.ezops.cloud`** is often used in the team's local-env; it points to that developer's machine IP. Point it to your machine via DNS or `/etc/hosts` (`127.0.0.1 localdash.ace.ezops.cloud`).
- **Agent / automated environment (e.g. Daytona)**: Do **not** use `localdash.ace.ezops.cloud`. Use the URL configured in the **Daytona proxy** for the workspace, or use **`ace-development.ace.ezops.cloud`**. The latter can be registered in **Route53**, in the hosted zone **ace.ezops.cloud**, pointing to the IP of the environment (Daytona proxy or dev VM). See [local-setup/09-hostname-and-dns.md](./local-setup/09-hostname-and-dns.md).
- **Option: localhost**: Replace the hostname with `localhost` in env vars and Compose for every URL the browser or host uses; some callbacks (e.g. from LLM to backend) may need to stay as a hostname reachable from containers (e.g. host-gateway or your LAN IP).

---

## Profiles

Compose uses **profiles** so you can start only the services you need.

| Profile | Typical use |
|---------|-------------|
| **all** | Full ACE stack: DB, Redis, apps, bots, scheduler, LLM, dashboard, LocalStack, pgadmin, redisinsight, Jira app + nginx. |
| **database** | psql, pgadmin, redis, redisinsight, mongo. |
| **application** | dash-back, dash-front + database deps. |
| **dashboard** | Same as application + Redis, LLM, configuration, redisinsight. |
| **apps** | configuration, db-gateway, slackbot, commands-api, ops-scheduler, ops-bot, llm, dash-back, dash-front, litellm. |
| **bot** | Bots (slackbot, ops-bot) + db-gateway, redis, llm. |
| **scheduler** | ops-scheduler, commands-api + db-gateway, redis, localstack. |
| **infra** | localstack (+ init-scripts). |
| **llm** | llm, litellm + psql, redis. |
| **tool** | pgadmin, redisinsight, configuration. |
| **ops** | ops-scheduler + dependencies. |
| **jira** | Jira integration app + nginx (HTTPS). Requires cert for TLS. |
| **cert** | letsencrypt one-shot: generate wildcard SSL cert for `CERT_DOMAIN` (Route53). Run once before using Jira over HTTPS. |

Example — full stack:

```bash
cd local-env
docker compose --profile all up -d
```

Example — only database and dashboard:

```bash
docker compose --profile dashboard up -d
```

Example — only infra (LocalStack) for SQS:

```bash
docker compose --profile infra up -d localstack
```

---

## Services (Docker Compose)

All services are in `local-env/docker-compose.yaml`. Summary:

### Infrastructure and data stores

| Service | Image / Build | Port (host:container) | Profiles | Volumes | Notes |
|---------|----------------|------------------------|----------|---------|--------|
| **psql** | postgres:17 | 3432:5432 | database, all, application, apps, bot, scheduler, infra, llm, dashboard, ops, tool | psql_data | User `root`, password set in env. Healthcheck: pg_isready. |
| **pgadmin** | dpage/pgadmin4:latest | 3080:80 | database, tool, dashboard, all | — | Email/password in env. Depends on psql healthy. |
| **redis** | redis:7-alpine | 3379:6379 | database, all, application, apps, bot, scheduler, infra, llm, dashboard, ops | redis_data | No password by default. Healthcheck: redis-cli ping. |
| **redisinsight** | redis/redisinsight:latest | 3540:3540 | database, tool, dashboard, all | — | RI_APP_PORT=3540. Depends on redis. |
| **mongo** | mongo | 3017:27017 | database, all, application, apps, bot, scheduler, infra, llm, dashboard, ops | — | Root user/password in env. Used by commands-api (DocumentDB compat). |
| **localstack** | localstack/localstack | 3066:4566 | infra, all | ./init-scripts → /etc/localstack/init/ready.d | SERVICES=sqs. Init scripts create SQS queues (see init-scripts). |

### ACE applications (Node)

| Service | Image / Build | Port (host:container) | Profiles | Volumes | Entrypoint / Notes |
|---------|----------------|------------------------|----------|---------|--------------------|
| **configuration** | node:20 | 3030:3030 | database, all, application, apps, bot, bot-logs, scheduler, infra, llm, dashboard, ops, tool | <ACE_ROOT>/ace-configuration:/app | yarn start:configuration. DB: configuration on psql; Redis prefix local:. |
| **db-gateway** | node:20 | 3031:3031 | database, all, application, apps, bot, bot-logs, scheduler, infra, llm, dashboard, ops | <ACE_ROOT>/ace-db-gateway:/app | yarn dev. CONFIGURATION_DB_URL, LLM_DB_URL, REDIS_HOST, tokens, ENCRYPTION_KEY, AWS_* (Secrets Manager path optional for local). |
| **slackbot** | node:20 | 3033:3033 | apps, bot, bot-logs, all | <ACE_ROOT>/ace-slackbot:/app | yarn start:dev. SLACK_*, AI_API_URL→llm, REDIS_HOST, ACE_DB_GATEWAY_ENDPOINT, STACK_BACKEND_URL. |
| **commands-api** | **build** (see below) | 3035:3035 | apps, scheduler, all | <ACE_ROOT>/ace-commands-api:/app | /app/entrypoint.sh. Build from ace-infra context + local-ace.commands-api.Dockerfile. SQS URLs (LocalStack or AWS), MONGO_URL, ACE_DB_GATEWAY_ENDPOINT. |
| **ops-scheduler** | node:20 | 3036:3036 | apps, ops, scheduler, all | <ACE_ROOT>/ace-ops-scheduler:/app | yarn sequelize:dev. Depends on localstack. LLM_API_URI, ACE_DB_GATEWAY_*, REDIS_HOST, RESOURCE_HEALTH_*_QUEUE_URL, LOCAL_AWS_ENDPOINT. |
| **ops-bot** | node:20 | 3038:3038 | apps, bot, bot-logs, all | <ACE_ROOT>/ace-ops-bot:/app | yarn start:dev. SLACK_*, AI_API_URL→llm, REDIS_HOST, ACE_DB_GATEWAY_ENDPOINT, SCHEDULLER_DB_GATEWAY_TOKEN. |
| **llm** | **build** (see below) | 3040:8080 | apps, llm, scheduler, bot, bot-logs, dashboard, all | — | Build: context ezrael-bot-llm, dockerfile ace-infra/ace-llm/ace.llm.Dockerfile3. DATABASE_URL→psql/llmdatabase, REDIS_URL, API keys (Anthropic, Gemini, OpenRouter, etc.), DB_GATEWAY_URL, SLACKBOT_*, QUEUE_*, LITELLM_*. |
| **dash-back** | node:20 | 3041:8080 | application, dashboard, all | <ACE_ROOT>/ace-stack-backend:/app | sh -c "yarn install && yarn start:dev". DB_*, JWT_*, OAuth (Google/GitHub), ACE_GATEWAY_URL, REDIS_URL, LLM_URL, QUEUE_DOCS_SYNC_URL, SLACK_*, WIKI_JS_*, SELF_URL. |
| **dash-front** | node:20 | 3042:4173 | application, dashboard, all | <ACE_ROOT>/ace-dashboard-frontend:/app | /app/entrypoint.sh. VITE_* URLs point to localdash.ace.ezops.cloud:30xx (API, gateway, LLM). |

### Supporting (ACE stack)

| Service | Image / Build | Port (host:container) | Profiles | Volumes | Notes |
|---------|----------------|------------------------|----------|---------|--------|
| **litellm** | ghcr.io/berriai/litellm:main-latest | 3100:4100 | apps, llm, scheduler, bot, bot-logs, dashboard, all | litellm-config.yaml:/app/config.yaml | Used by LLM service. command: --config /app/config.yaml --port 4100 --host 0.0.0.0. API keys in env. |

**LiteLLM config (`litellm-config.yaml`)** — The file is mounted into the litellm container. Structure below; do not commit real keys. Use environment variables or a secrets manager and substitute in the file, or use LiteLLM’s support for env vars where available.

```yaml
model_list:
  - model_name: claude-4-5
    litellm_params:
      model: claude-sonnet-4-5-20250929
      api_key: ${ANTHROPIC_API_KEY}
  - model_name: gpt-oss-120b
    litellm_params:
      model: openrouter/openai/gpt-oss-120b
      api_key: ${OPENROUTER_API_KEY}
  - model_name: grok-code-fast-1
    litellm_params:
      model: openrouter/x-ai/grok-code-fast-1
      api_key: ${OPENROUTER_API_KEY}
  - model_name: gpt-5-nano
    litellm_params:
      model: openrouter/openai/gpt-5-nano
      api_key: ${OPENROUTER_API_KEY}
  - model_name: gpt-5-mini
    litellm_params:
      model: openrouter/openai/gpt-5-mini
      api_key: ${OPENROUTER_API_KEY}
  - model_name: gpt-5-pro
    litellm_params:
      model: openrouter/openai/gpt-5-pro
      api_key: ${OPENROUTER_API_KEY}
  - model_name: gemini-2.5-flash
    litellm_params:
      model: openrouter/google/gemini-2.5-flash
      api_key: ${OPENROUTER_API_KEY}
  - model_name: gemini-2.5-pro
    litellm_params:
      model: openrouter/google/gemini-2.5-pro
      api_key: ${OPENROUTER_API_KEY}
  - model_name: minimax-m2
    litellm_params:
      model: openrouter/minimax/minimax-m2
      api_key: ${OPENROUTER_API_KEY}
  - model_name: gpt-4
    litellm_params:
      model: openrouter/openai/gpt-4
      api_key: ${OPENROUTER_API_KEY}
  - model_name: minimax-m2.5
    litellm_params:
      model: openrouter/minimax/minimax-m2.5
      api_key: ${OPENROUTER_API_KEY}
  - model_name: gpt-5.1-codex-mini
    litellm_params:
      model: openrouter/openai/gpt-5.1-codex-mini
      api_key: ${OPENROUTER_API_KEY}

general_settings:
  master_key: ${LITELLM_MASTER_KEY}
  database_url: postgres://root:${POSTGRES_PASSWORD:-root_password}@psql:5432/litellm
```

Set `ANTHROPIC_API_KEY`, `OPENROUTER_API_KEY`, and `LITELLM_MASTER_KEY` in the litellm container env (e.g. in docker-compose); `POSTGRES_PASSWORD` for the DB URL. The real file may use literal keys; do not commit them.

### ACE–Jira integration

| Service | Image / Build | Port (host:container) | Profiles | Volumes | Notes |
|---------|----------------|------------------------|----------|---------|--------|
| **jira** | node:20 | 3055:3000 | jira, all | ./jira-app/meu-app-connect:/app | ACE–Jira integration app. ACE_DB_GATEWAY_ENDPOINT, ACE_DB_GATEWAY_TOKEN, OPS_SCHEDULER_URL. entrypoint: npm install && npm start. |
| **nginx** | nginx:alpine | 80:80, 443:443 | jira, all | ./volumes/nginx/certs, ./volumes/nginx/nginx.conf.template | Reverse proxy for Jira (HTTPS). CERT_DOMAIN from env. Depends on jira. |
| **letsencrypt** | build: ./letsencrypt | — | cert | ./volumes/nginx/certs | One-shot: generate wildcard SSL cert for CERT_DOMAIN (Route53 DNS challenge). Set CERT_DOMAIN, AWS_* in env. |

---

## Builds

- **commands-api**: Build context `ace-infra`, dockerfile `ace-infra/ace-commands-api/local-ace.commands-api.Dockerfile`. Ensure ace-infra and ace-commands-api are at the paths referenced in the compose file.
- **llm**: Build context `ezrael-bot-llm`, dockerfile `ace-infra/ace-llm/ace.llm.Dockerfile3`.
- **letsencrypt**: Build from `local-env/letsencrypt` (Dockerfile + entrypoint). One-shot container to generate wildcard SSL cert for ACE–Jira (CERT_DOMAIN + Route53).

All other Node services use the **node:20** image and mount the repo as a volume; they run `yarn` or `npm` inside the container (user `${UID:-1000}:${GID:-1000}` so files stay owned by the host user).

---

## Volumes (named and bind)

**Named volumes (persistence):**

- **psql_data** — PostgreSQL data.
- **redis_data** — Redis data.

**Bind mounts:**

- Each ACE app: host path to repo root → `/app` (e.g. `ace-configuration`, `ace-db-gateway`, `ace-stack-backend`, `ace-dashboard-frontend`, `ace-slackbot`, `ace-ops-bot`, `ace-commands-api`, `ace-ops-scheduler`).
- **init-scripts**: `./init-scripts` → `/etc/localstack/init/ready.d` (LocalStack runs these when ready).
- **litellm**: path to `litellm-config.yaml` → `/app/config.yaml`.
- **Jira**: `./jira-app/meu-app-connect` → `/app`. **nginx**: `./volumes/nginx/certs` (Letsencrypt certs), `./volumes/nginx/nginx.conf.template` (config template).

---

## Environment variables (overview)

Secrets (Slack tokens, API keys, DB passwords, JWT, OAuth client secrets, etc.) are set in the Compose file or in `local-env/.env`. **Do not commit real values.** Use `local-env/environment-variables.md` as a key reference; obtain values from the team or from AWS Secrets Manager (e.g. dev secrets).

**Common patterns:**

- **DB**: `POSTGRES_USER`/`POSTGRES_PASSWORD` (psql), `DATABASE_URL` / `DB_HOST`/`DB_USER`/`DB_PASSWORD`/`DB_DATABASE` (apps).
- **Redis**: `REDIS_HOST`, `REDIS_PORT`, `REDIS_URL` (e.g. `redis://redis:6379`).
- **Gateway**: `ACE_GATEWAY_URL` or `ACE_DB_GATEWAY_ENDPOINT`, `BACKEND_DB_GATEWAY_TOKEN`, `SCHEDULLER_DB_GATEWAY_TOKEN`.
- **AWS**: `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_REGION`, `SECRET_PATH_PREFIX` (optional for local).
- **Slack**: `SLACK_BOT_TOKEN`, `SLACK_SIGNING_SECRET` (per bot).
- **LLM / LiteLLM**: `ANTHROPIC_API_KEY`, `GEMINI_API_KEY`, `OPENROUTER_API_KEY`, `LITELLM_*`.
- **URLs**: Many services use a hostname and port for frontend, API, gateway, LLM, SQS (LocalStack). Developers often use `localdash.ace.ezops.cloud` (pointing to their machine); **agents must use the Daytona proxy URL or `ace-development.ace.ezops.cloud`** (see [local-setup/09-hostname-and-dns.md](./local-setup/09-hostname-and-dns.md)). Backend callback (e.g. LLM→backend) often uses Docker service name (e.g. `http://dash-back:8080`) so it works from inside the network.

**LocalStack / SQS:**  
If using LocalStack for SQS, set `LOCAL_AWS_ENDPOINT` to your hostname and port 3066 (e.g. `http://ace-development.ace.ezops.cloud:3066` for the agent, or `http://localdash.ace.ezops.cloud:3066` for a developer) or `http://localstack:4566` from another container, and set queue URLs to the LocalStack queue URLs (see init-scripts). Filas criadas pelo `init-scripts/sqs.sh`: e.g. `ResourceHealthChecks-local.fifo`, `ResourceHealthCheckResults-local.fifo`, `DocsSync-local.fifo`, etc.

**.env in local-env root:**  
Used for: `CERT_DOMAIN` (e.g. localdash.ace.ezops.cloud for a developer's Jira HTTPS, or ace-development.ace.ezops.cloud for the agent), `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_DEFAULT_REGION` (services that call AWS; letsencrypt needs AWS for Route53). Optional: `UID`, `GID` for container user (default 1000:1000).

---

## Init scripts (LocalStack)

In `local-env/init-scripts/`, scripts in `ready.d` run when LocalStack is ready. Example: `sqs.sh` creates FIFO queues:

- LocalCommandsQueue.fifo, LocalCommandsOutputQueue.fifo  
- ResourceHealthChecks-local.fifo, ResourceHealthCheckResults-local.fifo, ResourceHealthChecksDLQ-local.fifo  
- DocsSync-local.fifo  

Queue URLs (LocalStack default account): `http://localstack:4566/000000000000/<QueueName>`. From host: use your hostname and port 3066 (e.g. `http://ace-development.ace.ezops.cloud:3066/...` for the agent, or `http://localdash.ace.ezops.cloud:3066/...` if that DNS points to your machine).

---

## Order of startup (recommended)

1. **Infrastructure**: `docker compose --profile database up -d` (psql, redis, mongo, pgadmin, redisinsight). Wait for healthchecks.
2. **Optional LocalStack**: `docker compose --profile infra up -d localstack`; wait for init-scripts to create queues.
3. **Configuration** (if needed): `docker compose --profile all up -d configuration` (runs migrations/schema).
4. **Core apps**: `docker compose --profile all up -d db-gateway dash-back dash-front` (or use `--profile dashboard`).
5. **Bots / scheduler / LLM**: `docker compose --profile all up -d` to start everything, or select profiles (e.g. `apps`, `bot`, `scheduler`, `llm`).

Or start the full stack in one go:

```bash
cd local-env
docker compose --profile all up -d
```

Check logs: `docker compose logs -f <service>`.

---

## Health checks

- **psql**: `pg_isready -U root -d postgres` (in container).
- **redis**: `redis-cli ping` (in container or host if port 3379 mapped).
- **configuration**: `curl -f http://localhost:3030/health`.
- **dash-back**: Service health endpoint as defined in ace-stack-backend (e.g. `/health`).
- **db-gateway**: Health endpoint as per ace-db-gateway docs.

Other services may not define a healthcheck in Compose; check each repo’s docs.

---

## Docker Compose (ACE services only)

Below is the **docker-compose** content for the ACE system only (infrastructure + ACE apps + litellm). Secrets are shown as `${VAR}`; set them in `local-env/.env` or in the compose file (do not commit real values). Replace `${ACE_ROOT}` with your actual path (e.g. `/home/admin/ace` or `$HOME/ace`). In the example, `localdash.ace.ezops.cloud` appears in URLs; **for the agent**, use **ace-development.ace.ezops.cloud** (or the Daytona proxy URL) instead — see [local-setup/09-hostname-and-dns.md](./local-setup/09-hostname-and-dns.md).

```yaml
version: '3.8'

services:
  psql:
    image: postgres:17
    restart: always
    profiles: [database, all, application, apps, bot, scheduler, infra, llm, dashboard, ops, tool]
    environment:
      POSTGRES_USER: root
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD:-root_password}
    ports:
      - "3432:5432"
    volumes:
      - psql_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U root -d postgres"]
      interval: 5s
      timeout: 5s
      retries: 10

  pgadmin:
    image: dpage/pgadmin4:latest
    restart: always
    profiles: [database, tool, dashboard, all]
    environment:
      PGADMIN_DEFAULT_EMAIL: ${PGADMIN_EMAIL:-admin@example.com}
      PGADMIN_DEFAULT_PASSWORD: ${PGADMIN_PASSWORD:-admin}
    ports:
      - "3080:80"
    depends_on:
      psql:
        condition: service_healthy

  redis:
    image: redis:7-alpine
    container_name: redis
    restart: always
    profiles: [database, all, application, apps, bot, scheduler, infra, llm, dashboard, ops]
    ports:
      - "3379:6379"
    volumes:
      - redis_data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 3s
      retries: 5

  redisinsight:
    image: redis/redisinsight:latest
    profiles: [database, tool, dashboard, all]
    ports:
      - "3540:3540"
    environment:
      - RI_APP_PORT=3540
    restart: unless-stopped
    depends_on:
      redis:
        condition: service_healthy

  mongo:
    image: mongo
    profiles: [database, all, application, apps, bot, scheduler, infra, llm, dashboard, ops]
    environment:
      MONGO_INITDB_ROOT_USERNAME: root
      MONGO_INITDB_ROOT_PASSWORD: ${MONGO_ROOT_PASSWORD:-root2025}
    ports:
      - "3017:27017"

  configuration:
    image: node:20
    restart: no
    profiles: [database, all, application, apps, bot, bot-logs, scheduler, infra, llm, dashboard, ops, tool]
    environment:
      DATABASE_URL: postgres://root:${POSTGRES_PASSWORD:-root_password}@psql:5432/configuration
      DB_USE_SSL: "false"
      PORT: 3030
      DB_HOST: psql
      DB_USERNAME: root
      DB_PASSWORD: ${POSTGRES_PASSWORD:-root_password}
      DB_DATABASE: configuration
      NODE_ENV: local
      REDIS_HOST: redis
      REDIS_PORT: 6379
      REDIS_PASSWORD: ""
      REDIS_DB: configuration_redis_db
      REDIS_PREFIX: "local:"
      DB_LOGGING: "true"
      DB_SSL_STRICT: "false"
      DB_DIALECT: postgres
    volumes:
      - ${ACE_ROOT}/ace-configuration:/app
    ports:
      - "3030:3030"
    user: "${UID:-1000}:${GID:-1000}"
    working_dir: /app
    entrypoint: yarn start:configuration
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3030/health"]
      interval: 10s
      timeout: 10s
      retries: 3
      start_period: 30s

  db-gateway:
    image: node:20
    restart: always
    profiles: [database, all, application, apps, bot, bot-logs, scheduler, infra, llm, dashboard, ops]
    environment:
      SCHEDULLER_DB_GATEWAY_TOKEN: ${SCHEDULLER_DB_GATEWAY_TOKEN:-SCHEDULLER-DB_GATEWAY-TOKEN}
      CONFIGURATION_DB_URL: postgres://root:${POSTGRES_PASSWORD:-root_password}@psql:5432/configuration
      LLM_DB_URL: postgres://root:${POSTGRES_PASSWORD:-root_password}@psql:5432/llmdatabase
      PORT: 3031
      REDIS_HOST: redis
      SECRET_PATH_PREFIX: /ace/dev/secrets/local-env/
      BACKEND_DB_GATEWAY_TOKEN: ${BACKEND_DB_GATEWAY_TOKEN:-BACKEND-DB_GATEWAY-TOKEN}
      ENCRYPTION_KEY: ${ENCRYPTION_KEY}
      SECRETS_REGION: us-east-1
      AWS_ACCESS_KEY_ID: ${AWS_ACCESS_KEY_ID}
      AWS_SECRET_ACCESS_KEY: ${AWS_SECRET_ACCESS_KEY}
      DB_USE_SSL: "false"
    volumes:
      - ${ACE_ROOT}/ace-db-gateway:/app
    ports:
      - "3031:3031"
    user: "${UID:-1000}:${GID:-1000}"
    working_dir: /app
    entrypoint: yarn dev

  slackbot:
    image: node:20
    restart: always
    profiles: [apps, bot, bot-logs, all]
    environment:
      PORT: 3033
      SLACK_BOT_TOKEN: ${SLACKBOT_SLACK_BOT_TOKEN}
      SLACK_SIGNING_SECRET: ${SLACKBOT_SLACK_SIGNING_SECRET}
      AI_API_URL: http://llm:8080
      REDIS_HOST: redis
      AWS_ACCESS_KEY_ID: ${AWS_ACCESS_KEY_ID}
      AWS_SECRET_ACCESS_KEY: ${AWS_SECRET_ACCESS_KEY}
      ACE_DB_GATEWAY_ENDPOINT: http://db-gateway:3031
      ENV: local
      STACK_BACKEND_URL: http://dash-back:8080
      SELF_URL: http://slackbot:3033
    volumes:
      - ${ACE_ROOT}/ace-slackbot:/app
    ports:
      - "3033:3033"
    user: "${UID:-1000}:${GID:-1000}"
    working_dir: /app
    entrypoint: yarn start:dev

  commands-api:
    build:
      context: ${ACE_ROOT}/ace-infra
      dockerfile: ${ACE_ROOT}/ace-infra/ace-commands-api/local-ace.commands-api.Dockerfile
    restart: always
    profiles: [apps, scheduler, all]
    environment:
      PORT: 3035
      AWS_ACCESS_KEY_ID: ${AWS_ACCESS_KEY_ID}
      AWS_SECRET_ACCESS_KEY: ${AWS_SECRET_ACCESS_KEY}
      AWS_REGION: us-east-1
      SLACKBOT_URL: http://slackbot:3033
      QUEUE_COMMANDSLIST: ${QUEUE_COMMANDSLIST}
      QUEUE_COMMANDSOUTPUT: ${QUEUE_COMMANDSOUTPUT}
      MONGO_URL: mongodb://root:${MONGO_ROOT_PASSWORD:-root2025}@mongo:27017/commands-api?retryWrites=false&authSource=admin
      ACE_DB_GATEWAY_ENDPOINT: http://db-gateway:3031
      LOCAL_DEV: "true"
      RESOURCE_HEALTH_CHECKS_QUEUE_URL: http://localdash.ace.ezops.cloud:3066/000000000000/ResourceHealthChecks-local.fifo
      RESOURCE_HEALTH_CHECK_RESULTS_QUEUE_URL: http://localdash.ace.ezops.cloud:3066/000000000000/ResourceHealthCheckResults-local.fifo
      QUEUE_DOCS_SYNC_URL: http://localdash.ace.ezops.cloud:3066/000000000000/DocsSync-local.fifo
      LOCAL_AWS_ENDPOINT: http://localdash.ace.ezops.cloud:3066
    volumes:
      - ${ACE_ROOT}/ace-commands-api:/app
    ports:
      - "3035:3035"
    user: "${UID:-1000}:${GID:-1000}"
    working_dir: /app
    entrypoint: /app/entrypoint.sh

  localstack:
    container_name: localstack_main
    restart: always
    profiles: [infra, all]
    image: localstack/localstack
    ports:
      - "3066:4566"
    environment:
      - SERVICES=sqs
      - AWS_DEFAULT_REGION=us-east-1
    volumes:
      - ./init-scripts:/etc/localstack/init/ready.d

  ops-scheduler:
    image: node:20
    restart: always
    profiles: [apps, ops, scheduler, all]
    environment:
      PORT: 3036
      PERIOD: 60000
      LLM_API_URI: http://llm:8080
      SCHEDULLER_DB_GATEWAY_TOKEN: ${SCHEDULLER_DB_GATEWAY_TOKEN:-SCHEDULLER-DB_GATEWAY-TOKEN}
      AWS_ACCESS_KEY_ID: ${AWS_ACCESS_KEY_ID}
      AWS_SECRET_ACCESS_KEY: ${AWS_SECRET_ACCESS_KEY}
      AWS_REGION: us-east-1
      ACE_DB_GATEWAY_URI: http://db-gateway:3031
      ACE_DB_GATEWAY_ENDPOINT: http://db-gateway:3031
      RESOURCE_HEALTH_CHECKS_QUEUE_URL: http://localdash.ace.ezops.cloud:3066/000000000000/ResourceHealthChecks-local.fifo
      RESOURCE_HEALTH_CHECK_RESULTS_QUEUE_URL: http://localdash.ace.ezops.cloud:3066/000000000000/ResourceHealthCheckResults-local.fifo
      LOCAL_AWS_ENDPOINT: http://localdash.ace.ezops.cloud:3066
      REDIS_HOST: redis
      STACK_BACKEND_URL: http://dash-back:8080
      SELF_URL: http://ops-scheduler:3036
    volumes:
      - ${ACE_ROOT}/ace-ops-scheduler:/app
    ports:
      - "3036:3036"
    user: "${UID:-1000}:${GID:-1000}"
    working_dir: /app
    entrypoint: yarn sequelize:dev
    depends_on:
      - localstack

  ops-bot:
    image: node:20
    restart: always
    profiles: [apps, bot, bot-logs, all]
    environment:
      PORT: 3038
      SLACK_BOT_TOKEN: ${OPSBOT_SLACK_BOT_TOKEN}
      SLACK_SIGNING_SECRET: ${OPSBOT_SLACK_SIGNING_SECRET}
      AI_API_URL: http://llm:8080
      REDIS_HOST: redis
      AWS_ACCESS_KEY_ID: ${AWS_ACCESS_KEY_ID}
      AWS_SECRET_ACCESS_KEY: ${AWS_SECRET_ACCESS_KEY}
      ACE_DB_GATEWAY_ENDPOINT: http://db-gateway:3031
      SCHEDULLER_DB_GATEWAY_TOKEN: ${SCHEDULLER_DB_GATEWAY_TOKEN:-SCHEDULLER-DB_GATEWAY-TOKEN}
    volumes:
      - ${ACE_ROOT}/ace-ops-bot:/app
    ports:
      - "3038:3038"
    user: "${UID:-1000}:${GID:-1000}"
    working_dir: /app
    entrypoint: yarn start:dev

  llm:
    build:
      context: ${ACE_ROOT}/ezrael-bot-llm
      dockerfile: ${ACE_ROOT}/ace-infra/ace-llm/ace.llm.Dockerfile3
    restart: always
    profiles: [apps, llm, scheduler, bot, bot-logs, dashboard, all]
    ports:
      - "3040:8080"
    environment:
      DATABASE_URL: postgresql://root:${POSTGRES_PASSWORD:-root_password}@psql:5432/llmdatabase
      REDIS_URL: redis://redis:6379
      DB_GATEWAY_URL: http://db-gateway:3031
      ACE_DB_GATEWAY_ENDPOINT: http://db-gateway:3031
      SLACKBOT_URL: http://slackbot:3033
      SCHEDULE_AWS_URL: postgresql://root:${POSTGRES_PASSWORD:-root_password}@psql:5432/configuration
      OPS_SCHEDULER_URL: http://ops-scheduler:3036
      LITELLM_BASE_URL: http://litellm:4100/v1
      ANTHROPIC_API_KEY: ${ANTHROPIC_API_KEY}
      GEMINI_API_KEY: ${GEMINI_API_KEY}
      OPENROUTER_API_KEY: ${OPENROUTER_API_KEY}
      AWS_ACCESS_KEY_ID: ${AWS_ACCESS_KEY_ID}
      AWS_SECRET_ACCESS_KEY: ${AWS_SECRET_ACCESS_KEY}
      QUEUE_COMMANDSLIST: ${QUEUE_COMMANDSLIST}
      QUEUE_COMMANDSOUTPUT: ${QUEUE_COMMANDSOUTPUT}
      LITELLM_API_KEY: ${LITELLM_API_KEY}
      LITELLM_MASTER_KEY: ${LITELLM_MASTER_KEY}
      SCHEDULLER_DB_GATEWAY_TOKEN: ${SCHEDULLER_DB_GATEWAY_TOKEN:-SCHEDULLER-DB_GATEWAY-TOKEN}
    # Add other LLM env vars (DAYTONA_API_KEY, channel IDs, etc.) as needed

  dash-back:
    image: node:20
    restart: always
    profiles: [application, dashboard, all]
    environment:
      DB_HOST: psql
      DB_PORT: 5432
      DB_USER: root
      DB_NAME: dashboard
      DB_PASSWORD: ${POSTGRES_PASSWORD:-root_password}
      JWT_SECRET: ${JWT_SECRET}
      JWT_EXPIRES_IN: 24h
      PORT: 8080
      DB_SSL_ENABLED: "false"
      ACE_GATEWAY_URL: http://db-gateway:3031
      ACE_GATEWAY_TOKEN: ${BACKEND_DB_GATEWAY_TOKEN:-BACKEND-DB_GATEWAY-TOKEN}
      SERVICE_TOKEN: ${BACKEND_DB_GATEWAY_TOKEN:-BACKEND-DB_GATEWAY-TOKEN}
      REDIS_URL: redis://redis:6379
      FRONTEND_URL: http://localdash.ace.ezops.cloud:3042
      FRONTEND_HOST: http://localdash.ace.ezops.cloud:3042
      API_HOST: http://localdash.ace.ezops.cloud:3041
      QUEUE_DOCS_SYNC_URL: http://localdash.ace.ezops.cloud:3066/000000000000/DocsSync-local.fifo
      LLM_URL: http://llm:8080
      SELF_URL: http://dash-back:8080
      SLACK_BOT_TOKEN: ${OPSBOT_SLACK_BOT_TOKEN}
      SLACK_SIGNING_SECRET: ${OPSBOT_SLACK_SIGNING_SECRET}
      GOOGLE_CLIENT_ID: ${GOOGLE_CLIENT_ID}
      GOOGLE_CLIENT_SECRET: ${GOOGLE_CLIENT_SECRET}
      GITHUB_CLIENT_ID: ${GITHUB_CLIENT_ID}
      GITHUB_CLIENT_SECRET: ${GITHUB_CLIENT_SECRET}
      WIKI_JS_URL: ${WIKI_JS_URL}
      WIKI_JS_GRAPHQL_URL: ${WIKI_JS_GRAPHQL_URL}
      WIKI_JS_API_KEY: ${WIKI_JS_API_KEY}
    volumes:
      - ${ACE_ROOT}/ace-stack-backend:/app
    ports:
      - "3041:8080"
    user: "${UID:-1000}:${GID:-1000}"
    working_dir: /app
    entrypoint: ["sh", "-c", "yarn install && yarn start:dev"]

  dash-front:
    image: node:20
    restart: no
    profiles: [application, dashboard, all]
    environment:
      VITE_API_URL: http://localdash.ace.ezops.cloud:3041
      VITE_FRONTEND_URL: http://localdash.ace.ezops.cloud:3042
      VITE_ACE_GATEWAY_URL: http://localdash.ace.ezops.cloud:3031
      VITE_BACKEND_URL: http://localdash.ace.ezops.cloud:3041
      VITE_LLM_URL: http://localdash.ace.ezops.cloud:3040
    volumes:
      - ${ACE_ROOT}/ace-dashboard-frontend:/app
    ports:
      - "3042:4173"
    user: "${UID:-1000}:${GID:-1000}"
    working_dir: /app
    entrypoint: /app/entrypoint.sh

  litellm:
    image: ghcr.io/berriai/litellm:main-latest
    restart: always
    profiles: [apps, llm, scheduler, bot, bot-logs, dashboard, all]
    ports:
      - "3100:4100"
    environment:
      ANTHROPIC_API_KEY: ${ANTHROPIC_API_KEY}
      GEMINI_API_KEY: ${GEMINI_API_KEY}
      OPENROUTER_API_KEY: ${OPENROUTER_API_KEY}
      STORE_MODEL_IN_DB: "true"
    volumes:
      - ${ACE_ROOT}/litellm-config.yaml:/app/config.yaml
    command: ["--config", "/app/config.yaml", "--port", "4100", "--host", "0.0.0.0"]

  jira:
    image: node:20
    restart: always
    profiles: [jira, all]
    environment:
      PORT: 3000
      NODE_ENV: local
      ACE_DB_GATEWAY_ENDPOINT: http://db-gateway:3031
      ACE_DB_GATEWAY_TOKEN: ${ACE_DB_GATEWAY_TOKEN:-BACKEND-DB_GATEWAY-TOKEN}
      OPS_SCHEDULER_URL: http://ops-scheduler:3036
    volumes:
      - ./jira-app/meu-app-connect:/app
    ports:
      - "3055:3000"
    user: "${UID:-1000}:${GID:-1000}"
    working_dir: /app
    entrypoint: ["sh", "-c", "npm install && npm start"]

  nginx:
    image: nginx:alpine
    restart: always
    profiles: [jira, all]
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./volumes/nginx/certs:/etc/letsencrypt:ro
      - ./volumes/nginx/nginx.conf.template:/etc/nginx/templates/nginx.conf.template:ro
    environment:
      CERT_DOMAIN: "${CERT_DOMAIN:-localhost}"
    command: ["sh", "-c", "envsubst '$${CERT_DOMAIN}' < /etc/nginx/templates/nginx.conf.template > /etc/nginx/nginx.conf && exec nginx -g 'daemon off;'"]
    depends_on:
      - jira

  letsencrypt:
    build: ./letsencrypt
    restart: "no"
    profiles: [cert]
    environment:
      CERT_DOMAIN: "${CERT_DOMAIN:?CERT_DOMAIN is required for letsencrypt}"
      AWS_ACCESS_KEY_ID: "${AWS_ACCESS_KEY_ID:-}"
      AWS_SECRET_ACCESS_KEY: "${AWS_SECRET_ACCESS_KEY:-}"
      AWS_DEFAULT_REGION: "${AWS_DEFAULT_REGION:-us-east-1}"
    volumes:
      - ./volumes/nginx/certs:/etc/letsencrypt
    user: root

volumes:
  psql_data:
  redis_data:
```

The real `local-env/docker-compose.yaml` may contain additional services (e.g. n8n) and inline secrets; the above is the ACE subset (including ACE–Jira integration) with placeholders. Use it as reference or copy into a file and set `ACE_ROOT` and all `${VAR}` in `.env`.

---

## HTTPS for ACE–Jira (nginx + letsencrypt)

The **Jira integration** is exposed over HTTPS via nginx. To use it:

1. **Profile `cert`**: Run the letsencrypt service once with `CERT_DOMAIN` and `AWS_*` set (Route53 DNS challenge for the wildcard). Certificates are written to `local-env/volumes/nginx/certs`.
2. **Profile `jira` or `all`**: nginx uses `volumes/nginx/nginx.conf.template` and certs in `volumes/nginx/certs`; `CERT_DOMAIN` is substituted. Ensure the Jira hostname (e.g. `jira.localdash.ace.ezops.cloud`) resolves to your machine and that the cert covers it.

See `local-env/docs/jira-nginx-letsencrypt.md` for details.

---

## Links

- **local-env** — Docker Compose, .env, init-scripts, volumes, docs (e.g. localstack-sqs.md, llm-env-local-vs-dev.md, jira-nginx-letsencrypt.md).
- **Environment variables reference** — [env-vars-and-secrets.md](./env-vars-and-secrets.md).
- **Architecture** — [../architecture/deployment.md](../architecture/deployment.md).
- **Infrastructure rules** — [../rules/infrastructure-rules.md](../rules/infrastructure-rules.md).
- **Troubleshooting** — [troubleshooting.md](./troubleshooting.md).
