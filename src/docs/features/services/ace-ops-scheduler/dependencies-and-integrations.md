# Dependencies and integrations

ace-ops-scheduler depends on **ace-db-gateway**, **ace-stack-backend** (omnichannel), an **LLM service** (scripts, exploration, health checks), and **Redis** (optional but used for caching and timing). All URLs and credentials are configured via environment variables.

## ace-db-gateway

- **Role**: Single source for scheduler events, monitored resources, projects, channels, configurations, secrets, and knowledge base IDs. All DB access from ace-ops-scheduler goes through this service.
- **Base URL**: `ACE_DB_GATEWAY_ENDPOINT` or `ACE_DB_GATEWAY_URI` (default `http://ace-db-gateway-service`).
- **Auth**: `SCHEDULLER_DB_GATEWAY_TOKEN` sent as `Authorization` header on requests (DBGatewayClient). For contextResolver, `ACE_DB_GATEWAY_AUTH_TOKEN` is used as `Bearer` token (different env name).
- **Endpoints used** (from code):
  - Schedulers: GET/POST/PUT/DELETE `/api/schedulers` (with optional `excludeCheckPrefix`, `limit`, etc.).
  - Monitored resources: GET `/api/monitored-resources`, PUT `/api/monitored-resources/:id`, PUT `/api/monitored-resources/batch`, PUT `/api/monitored-resources/increment-misses`, DELETE `/api/monitored-resources/:id`.
  - Channels: GET `/api/channels?slackId=`, GET `/api/channels/:id`, GET `/api/channels/project?slackId=`, POST `/api/channels/project` (body `slackChannelId` for contextResolver).
  - Projects: GET `/api/projects/:id`, GET `/api/projects/by-name?name=`, GET `/api/projects/:id` for channel (project response includes channels).
  - Knowledge base: GET `/api/projects/:id/knowledge-base`.
  - Configurations: GET `/api/configurations?projectId=`.
  - Secrets: GET `/api/secrets/:projectId`.
  - GitHub token: GET `/api/github-tokens/user/:userId`.

## ace-stack-backend (omnichannel)

- **Role**: Receives payloads and scheduler-originated messages via omnichannel ingest; processes them and can call back ace-ops-scheduler with results.
- **URL**: `STACK_BACKEND_URL` (default `http://web-backend-service`).
- **Endpoint**: POST `/api/omni/ingest` with body containing `message`, `channel_type`, `channel_id`, `user_id`, `origin`, `callback_url`, `client_id`, `project_id`, optional `knowledge_base_id`, optional `token`.
- **Callback**: ace-ops-scheduler registers `callback_url: SELF_URL/scheduler-callback` so the backend can POST execution summary back; the service parses `conversa_id` (e.g. `scheduler-event-123`) and updates the corresponding event execution.

## LLM service

- **Role**: Generate health check scripts, run agentic resource exploration, and execute health check scripts in a sandbox (Daytona).
- **Base URL**: `LLM_API_URI` (default `http://llm-service`).
- **Endpoints used** (from llmAIClient):
  - POST `/api/v3/ai/generate-scripts`: body `{ resources }`; returns `scripts`.
  - POST `/api/v3/ai/explore-resources`: body `{ seeds, project_id, repository, infra_files }`; returns `resources`.
  - POST `/api/v3/ai/execute-health-checks`: body `{ project_id, resources }`; returns `results` (and optional `sandbox_id`).
- **Timeouts**: `LLM_TIMEOUT_MS` (default 120000), `LLM_EXPLORATION_TIMEOUT_MS` (default 300000), `LLM_HEALTH_CHECK_TIMEOUT_MS` (default 300000).

## Redis

- **Role**: (1) Cache for knowledge base ID per project in DBGatewayClient (TTL 24 h). (2) Resource discovery timing: start time stored when discovery starts; completed when discovery finishes (then optionally written to a JSON file).
- **Config**: `REDIS_HOST` (default `redis-service`), `REDIS_PORT` (default 6379). Connection is lazy; errors are logged but do not fail the app.

## GitHub / docs repo

- **Role**: Resource discovery fetches infra files from the project's docs repo (repository and branch from project configurations). Used by `githubDocsFetcher` and discoveryOrchestrator. GitHub token comes from project secrets or user token via DB Gateway.

## Environment variables summary

| Variable | Purpose |
|----------|---------|
| `PORT` | Express server port (default 3000). |
| `ACE_DB_GATEWAY_ENDPOINT` / `ACE_DB_GATEWAY_URI` | DB Gateway base URL. |
| `SCHEDULLER_DB_GATEWAY_TOKEN` | Auth for DB Gateway (schedulers, resources, etc.). |
| `ACE_DB_GATEWAY_AUTH_TOKEN` | Auth for contextResolver (channel/project resolution). |
| `STACK_BACKEND_URL` | Backend omnichannel ingest URL. |
| `SELF_URL` | Base URL of this service (for callback_url). |
| `LLM_API_URI` | LLM service base URL. |
| `LLM_TIMEOUT_MS`, `LLM_EXPLORATION_TIMEOUT_MS`, `LLM_HEALTH_CHECK_TIMEOUT_MS` | Timeouts for LLM calls. |
| `REDIS_HOST`, `REDIS_PORT` | Redis connection. |
| `CHANNEL_ID`, `FALLBACK_CHANNEL_ID` | Default Slack channel for payloads. |
| `AGENTIC_EXPLORATION_ENABLED` | If `'false'`, discovery skips LLM exploration. |
| `DEFAULT_DOCS_REPOSITORY` | Fallback docs repo if project has no docsRepo config. |

## Related

- [Overview](./overview.md)
- [API and routes](./api-and-routes.md)
- [Resource health](./resource-health.md)
- [Architecture service catalog](../../architecture/service-catalog.md)
