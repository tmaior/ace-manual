# Step 5: Startup Sequence

Start services in the order below so that dependencies (databases, Redis, LocalStack, configuration) are ready before the apps that use them. All commands assume you are in **local-env** (`ACE_ROOT/local-env`) and use Docker Compose V2 (`docker compose`).

---

## Option 1: Start the full stack in one go

If paths and env vars are already set (steps 01–04), you can start everything with the **all** profile:

```bash
cd local-env
docker compose --profile all up -d
```

Then wait for healthchecks and app startup (see [07-health-checks-and-validation.md](./07-health-checks-and-validation.md)). This is the fastest way to get an environment equal to local-env, but if something fails, use Option 2 to isolate the problem.

---

## Option 2: Step-by-step startup (recommended for first run or troubleshooting)

### 1. Start infrastructure (databases, Redis, Mongo)

```bash
cd local-env
docker compose --profile database up -d
```

**Wait** until PostgreSQL and Redis are healthy (about 10–30 seconds). Check:

```bash
docker compose ps
```

Look for `psql` and `redis` state "healthy" (or "running"). Optional: test Redis with `docker compose exec redis redis-cli ping` and PostgreSQL with `docker compose exec psql pg_isready -U root`.

### 2. Start LocalStack (for SQS queues)

Required if you run **ops-scheduler** or **commands-api** with LocalStack queue URLs.

```bash
docker compose --profile infra up -d localstack
```

**Wait** for LocalStack to be ready and for init scripts to run (about 15–30 seconds). The script in `init-scripts/sqs.sh` creates the FIFO queues. Check logs:

```bash
docker compose logs localstack
```

You should see "Creating SQS queues..." and "SQS queues created successfully" (or equivalent from the init script).

### 3. Start configuration service

This service runs DB migrations/setup for the configuration database.

```bash
docker compose --profile all up -d configuration
```

**Wait** for the configuration healthcheck to pass (e.g. 30–60 seconds). Check:

```bash
curl -f http://localhost:3030/health
```

(or use the host/port you mapped; use your chosen hostname — e.g. `ace-development.ace.ezops.cloud` for the agent — if configured; see [09-hostname-and-dns.md](./09-hostname-and-dns.md)).

### 4. Start core applications

Start db-gateway, backend, and frontend (and optionally other app services):

```bash
docker compose --profile dashboard up -d
```

Or start only the services you need:

```bash
docker compose --profile all up -d db-gateway dash-back dash-front
```

**Wait** for db-gateway and dash-back to be up (no formal healthcheck in all setups; check logs). Then verify:

- db-gateway: `curl -f http://localhost:3031/` or the health path from ace-db-gateway docs.
- dash-back: `curl -f http://localhost:3041/health` or equivalent.
- dash-front: open `http://localhost:3042` (or the URL set in `VITE_FRONTEND_URL`) in a browser.

### 5. Start bots, scheduler, LLM (optional)

To run the full stack as in local-env:

```bash
docker compose --profile all up -d
```

This brings up slackbot, ops-bot, commands-api, ops-scheduler, llm, litellm, and any other services in the **all** profile. **ops-scheduler** depends on **localstack**; ensure LocalStack and init scripts have run (step 2) before relying on scheduler/commands-api.

For only bots and LLM (no dashboard):

```bash
docker compose --profile bot --profile llm up -d
```

Adjust profiles as needed (see [06-profiles-and-services-reference.md](./06-profiles-and-services-reference.md)).

---

## Wait times (summary)

| Step | Suggested wait |
|------|-----------------|
| After `database` up | 10–30 s for psql/redis healthy |
| After localstack up | 15–30 s for init scripts to create queues |
| After configuration up | 30–60 s for healthcheck to pass |
| After db-gateway / dash-back up | 20–60 s for apps to listen |

Use `docker compose logs -f <service>` to confirm a service has started (e.g. "Listening on port 3031").

---

## Restarting or stopping

- **Stop all** (keep volumes): `docker compose --profile all down`
- **Stop and remove volumes** (fresh DB/Redis): `docker compose --profile all down -v`
- **Restart one service**: `docker compose restart db-gateway`
- **View logs**: `docker compose logs -f db-gateway` (or service name)

---

## Order summary (quick reference)

1. `docker compose --profile database up -d` → wait for healthy.
2. `docker compose --profile infra up -d localstack` → wait for init scripts.
3. `docker compose up -d configuration` → wait for health.
4. `docker compose --profile dashboard up -d` or `docker compose --profile all up -d`.
5. Validate with [07-health-checks-and-validation.md](./07-health-checks-and-validation.md).

Next: [06-profiles-and-services-reference.md](./06-profiles-and-services-reference.md) for profile details, then [07-health-checks-and-validation.md](./07-health-checks-and-validation.md) to confirm the environment works.
