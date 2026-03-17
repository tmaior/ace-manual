# Step 8: Troubleshooting

Common failures when bringing up the local ACE environment and how to fix them. Use this after [05-startup-sequence.md](./05-startup-sequence.md) and [07-health-checks-and-validation.md](./07-health-checks-and-validation.md) when something does not start or respond.

---

## Path and volume errors

### Symptom

- Error like "no such file or directory" or "path does not exist" when starting a service.
- Container exits immediately with volume mount errors.

### Cause

The paths in `docker-compose.yaml` point to a directory that does not exist on your machine (e.g. `/home/admin/ace/ace-configuration` but your repos are under `/Users/me/ace`).

### Fix

1. Confirm your [directory layout](./02-repositories-and-directory-layout.md): all repos under `ACE_ROOT` and names match (ace-configuration, ace-db-gateway, etc.).
2. Edit `local-env/docker-compose.yaml` and replace every absolute path (e.g. `/home/admin/ace`) with your actual `ACE_ROOT`. Also fix **build** contexts (commands-api, llm) that use absolute paths.
3. Or introduce `ACE_ROOT` in the Compose file and set it in `local-env/.env` (see [03-docker-compose-and-configuration.md](./03-docker-compose-and-configuration.md)).

---

## Database or Redis not ready

### Symptom

- configuration, db-gateway, or dash-back fail to start; logs show "connection refused" or "ECONNREFUSED" to psql or redis.

### Cause

Apps started before PostgreSQL or Redis were healthy, or healthchecks have not passed yet.

### Fix

1. Start with `docker compose --profile database up -d` and wait 10–30 seconds.
2. Verify: `docker compose exec psql pg_isready -U root`, `docker compose exec redis redis-cli ping`.
3. Then start configuration, then db-gateway and dash-back. See [05-startup-sequence.md](./05-startup-sequence.md).

---

## LocalStack / SQS queues missing

### Symptom

- ops-scheduler or commands-api logs show errors about SQS (queue does not exist, connection error to LocalStack).

### Cause

LocalStack was not started, or init scripts did not run (queues not created).

### Fix

1. Start LocalStack: `docker compose --profile infra up -d localstack`.
2. Wait 15–30 seconds for init scripts in `local-env/init-scripts/` (e.g. `sqs.sh`) to run. Check `docker compose logs localstack`.
3. Ensure `init-scripts/sqs.sh` exists and is executable (`chmod +x init-scripts/sqs.sh`).
4. In Compose, queue URLs and `LOCAL_AWS_ENDPOINT` must point to LocalStack (from host use your chosen hostname and port 3066, e.g. `http://ace-development.ace.ezops.cloud:3066` for the agent, or `http://localhost:3066`; from another container use `http://localstack:4566`). See [04-environment-variables.md](./04-environment-variables.md) and [09-hostname-and-dns.md](./09-hostname-and-dns.md).

---

## Configuration service fails or never healthy

### Symptom

- configuration container exits or healthcheck never passes (curl to 3030 fails).

### Cause

- PostgreSQL not ready or wrong credentials (e.g. wrong `POSTGRES_PASSWORD`).
- Missing or wrong `DATABASE_URL` / `DB_*` in Compose.

### Fix

1. Check psql is healthy and that the `configuration` database exists (configuration service may create it on first run; ensure user/password match).
2. Verify env in Compose for configuration: `DATABASE_URL=postgres://root:<password>@psql:5432/configuration`, and that `POSTGRES_PASSWORD` matches psql.
3. Check logs: `docker compose logs configuration`.

---

## dash-back or db-gateway connection errors

### Symptom

- dash-back cannot connect to db-gateway; db-gateway cannot connect to configuration DB or Redis.

### Cause

- Wrong URLs or tokens: `ACE_GATEWAY_URL`, `ACE_GATEWAY_TOKEN` / `BACKEND_DB_GATEWAY_TOKEN`, or DB URLs.
- db-gateway not running or not listening yet.

### Fix

1. Ensure db-gateway is up and healthy: `curl -f http://localhost:3031/` (or health endpoint).
2. In dash-back env: `ACE_GATEWAY_URL=http://db-gateway:3031` (container name) and `ACE_GATEWAY_TOKEN` (or `BACKEND_DB_GATEWAY_TOKEN`) must match what db-gateway expects.
3. Check db-gateway logs for auth or DB connection errors.

---

## Frontend (dash-front) cannot reach API or shows wrong URL

### Symptom

- Dashboard loads but API calls fail (CORS, 404, or "cannot connect").
- Browser points to wrong host/port for API.

### Cause

- `VITE_*` URLs are set at **build** time. If they point to a different hostname than the one you use in the browser (e.g. ace-development.ace.ezops.cloud vs localhost), or the port is wrong, the browser will call the wrong URL.
- CORS on dash-back may not allow the origin you use (e.g. localhost vs ace-development.ace.ezops.cloud).

### Fix

1. Rebuild dash-front with correct env: set `VITE_API_URL`, `VITE_ACE_GATEWAY_URL`, `VITE_LLM_URL` (and related) to your actual hostname and ports — e.g. `http://ace-development.ace.ezops.cloud:3041` for the agent, or `http://localhost:3041`. Restart dash-front container after rebuilding.
2. For a developer on one machine: add the hostname to `/etc/hosts` (e.g. `127.0.0.1 localdash.ace.ezops.cloud`) and use the same URLs as in the Compose file. For the agent, use ace-development.ace.ezops.cloud (see [09-hostname-and-dns.md](./09-hostname-and-dns.md)).
3. Ensure dash-back CORS allows your frontend origin (host and port).

---

## Bots (slackbot, ops-bot) not connecting to Slack

### Symptom

- Containers run but logs show Slack API errors or "invalid token".

### Cause

- `SLACK_BOT_TOKEN` or `SLACK_SIGNING_SECRET` missing, wrong, or from a different Slack app.

### Fix

1. Obtain valid tokens from your Slack app (or use the team's dev app). Set them in `.env` or in the Compose env for slackbot/ops-bot.
2. For local-only runs without real Slack, you can leave bots running; they will not connect but the rest of the stack can be validated.

---

## commands-api or llm build fails

### Symptom

- `docker compose build commands-api` or `build llm` fails (Dockerfile not found, context error, npm install failure).

### Cause

- Build context or dockerfile path wrong (absolute path points to another machine's path).
- ace-infra or ezrael-bot-llm not at the path used in Compose.

### Fix

1. Ensure **ace-infra** and **ezrael-bot-llm** are cloned under `ACE_ROOT` (see [02-repositories-and-directory-layout.md](./02-repositories-and-directory-layout.md)).
2. Update build context and dockerfile in `docker-compose.yaml` to use your `ACE_ROOT` (e.g. `context: ${ACE_ROOT}/ace-infra`, `dockerfile: ${ACE_ROOT}/ace-infra/ace-commands-api/local-ace.commands-api.Dockerfile`). Set `ACE_ROOT` in `.env`.
3. If the Dockerfile expects files from the **app** repo (e.g. ace-commands-api), ensure the Compose build context and Dockerfile match what the Dockerfile expects (some builds copy from a different context).

---

## Port already in use

### Symptom

- "port is already allocated" or "bind: address already in use" when starting a service.

### Cause

Another process (or an old container) is using the same port (e.g. 3432, 3379, 3030, 3041).

### Fix

1. Stop other containers: `docker compose --profile all down`.
2. Find process using the port (e.g. `lsof -i :3030` or `ss -tlnp | grep 3030`). Stop that process or change the host port in the Compose file (e.g. "3043:4173" for dash-front if 3042 is in use).

---

## Where to look for more help

- **Service-specific docs**: [../../services/](../../services/) for each ACE service (env vars, APIs, health endpoints).
- **Local summary**: [../local.md](../local.md).
- **Env vars and secrets**: [../env-vars-and-secrets.md](../env-vars-and-secrets.md).
- **General troubleshooting**: [../troubleshooting.md](../troubleshooting.md) (CORS, URLs, per-environment differences).
