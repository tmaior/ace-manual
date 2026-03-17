# Step 7: Health Checks and Validation

After starting the local environment ([05-startup-sequence.md](./05-startup-sequence.md)), use these checks to confirm that the stack is running correctly. Success means the local setup behaves like **local-env** and is ready for development or testing.

---

## Infrastructure

### PostgreSQL (psql)

- **Port**: 3432 (host) → 5432 (container).
- **Check**: From host:
  ```bash
  docker compose exec psql pg_isready -U root -d postgres
  ```
  Expected: `postgres:5432 - accepting connections`.
- **Optional**: Connect with psql or pgAdmin (port 3080). Create databases `configuration`, `dashboard`, `llmdatabase` if not created by migrations.

### Redis

- **Port**: 3379 (host) → 6379 (container).
- **Check**:
  ```bash
  docker compose exec redis redis-cli ping
  ```
  Expected: `PONG`.

### Mongo

- **Port**: 3017 (host) → 27017 (container).
- **Check**: Container running; optional: connect with mongo shell or Mongo Express if enabled. commands-api expects `mongodb://root:<MONGO_ROOT_PASSWORD>@mongo:27017/commands-api?retryWrites=false&authSource=admin`.

### LocalStack (SQS)

- **Port**: 3066 (host) → 4566 (container).
- **Check**: List queues (from host, if awslocal or AWS CLI pointed at LocalStack):
  ```bash
  docker compose exec localstack awslocal sqs list-queues --region us-east-1
  ```
  Expected: Queues created by `init-scripts/sqs.sh` (e.g. `ResourceHealthChecks-local.fifo`, `DocsSync-local.fifo`, etc.).
- **Alternative**: Check LocalStack logs for "SQS queues created successfully" (or equivalent from init script).

---

## ACE services

### configuration (3030)

- **Check**:
  ```bash
  curl -f http://localhost:3030/health
  ```
  Expected: HTTP 200 and a healthy response body. If using a custom hostname (e.g. `ace-development.ace.ezops.cloud` for the agent), use that host instead of localhost.

### db-gateway (3031)

- **Check**: Health or root endpoint as defined in ace-db-gateway docs, e.g.:
  ```bash
  curl -f http://localhost:3031/
  ```
  or `curl -f http://localhost:3031/health`. Expected: HTTP 200.

### dash-back (3041)

- **Check**:
  ```bash
  curl -f http://localhost:3041/health
  ```
  (or the path documented in ace-stack-backend). Expected: HTTP 200.

### dash-front (3042)

- **Check**: Open in browser: `http://localhost:3042` (or the URL in `VITE_FRONTEND_URL`). Expected: Dashboard UI loads. If API URL is wrong, login or API calls may fail; verify `VITE_API_URL` points to dash-back (e.g. port 3041).

### slackbot (3033), ops-bot (3038)

- **Check**: Containers running; no simple HTTP health in all setups. Check logs: `docker compose logs slackbot`. If Slack tokens are invalid, bots will log connection errors but containers stay up.

### commands-api (3035)

- **Check**: Container running; optional: hit a known endpoint if the service exposes one. Check logs for startup and SQS/DB connection.

### ops-scheduler (3036)

- **Check**: Container running; check logs for successful start and LocalStack/SQS connectivity. No HTTP health required for validation.

### llm (3040), litellm (3100)

- **Check**: Containers running; optional: `curl http://localhost:3040/health` or similar if the LLM service exposes health. litellm often listens on 4100 inside container (mapped to 3100 on host).

---

## URLs (browser and API)

Use your configured hostname and these ports. **For the agent**: use **ace-development.ace.ezops.cloud** (or the Daytona proxy URL) — see [09-hostname-and-dns.md](./09-hostname-and-dns.md). **For a developer**: localdash.ace.ezops.cloud or localhost.

Example with **ace-development.ace.ezops.cloud** (agent):

- Dashboard: `http://ace-development.ace.ezops.cloud:3042`
- Backend API: `http://ace-development.ace.ezops.cloud:3041`
- DB Gateway: `http://ace-development.ace.ezops.cloud:3031`
- LLM: `http://ace-development.ace.ezops.cloud:3040`
- LocalStack: `http://ace-development.ace.ezops.cloud:3066`

If you use **localhost**, replace the host with `localhost` and use the same ports (3042, 3041, 3031, 3040, 3066).

---

## Success criteria (summary)

The local environment is **working like local-env** when:

1. **psql** and **redis** are healthy (pg_isready, redis-cli ping).
2. **configuration** returns 200 on `/health`.
3. **db-gateway** responds on its health or root endpoint.
4. **dash-back** returns 200 on `/health`.
5. **dash-front** loads in the browser and can call the backend (if login is configured).
6. If you started **LocalStack** and **ops-scheduler** / **commands-api**: LocalStack has queues and scheduler/commands-api logs show no SQS connection errors.
7. (Optional) **llm** and **litellm** are up if you need LLM features.

If any check fails, see [08-troubleshooting.md](./08-troubleshooting.md).
