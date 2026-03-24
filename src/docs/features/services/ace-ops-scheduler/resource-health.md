# Resource health (discovery and health checks)

ace-ops-scheduler runs two kinds of resource health work: **resource discovery** (discover resources from project docs/CI-CD and upsert them) and **health checks** (execute health check scripts for monitored resources and update status). Both are driven by scheduler events with `check` values `RESOURCE_DISCOVERY:{projectId}` and `HEALTH_CHECK:{projectId}`.

## Resource discovery

### Flow

1. **Trigger**: The resource health loop (or manual trigger) runs when the RESOURCE_DISCOVERY event is due. A per-project **concurrency lock** prevents overlapping discovery for the same project (lock TTL 10 minutes).
2. **Pipeline** (`discoveryOrchestrator.executeDiscovery`):
   - **Project config**: Fetch docs repo, branch, GitHub token from DB Gateway (configurations, optional user token, project secrets).
   - **CI/CD seeds**: Fetch infra files from the docs repo (e.g. GitHub Actions, Terraform, Dockerfile) via `githubDocsFetcher`; parse with `cicdSeedExtractor.extractSeeds()` (regex-based, no LLM).
   - **Agentic exploration** (if `AGENTIC_EXPLORATION_ENABLED !== 'false'`): Call LLM `explore-resources` with seeds and infra files; get back discovered resources.
   - **Reconcile**: Compare discovered resources with existing monitored resources in DB Gateway; split into "toGenerateScript" (new or missing script) and "toKeep" (reuse existing checker).
   - **Script generation**: For resources that need scripts, call LLM `generate-scripts`; build final resource list with `resourceIdentifier`, `checker` (script), etc.
   - **Batch upsert**: PUT monitored resources to DB Gateway batch endpoint.
   - **Stale reconciliation**: Resources that exist in DB but were not in discovery have `consecutiveMisses` incremented; after 3 consecutive misses the resource is soft-deleted.
3. **Event update**: On success (and `totalDiscovered > 0`), event's `lastRunTime` and `period` are set (success period 1440 min). On failure or zero discovered, only `lastRunResult` and `period` are updated (exponential backoff: 15, 30, 60, 120, 360 min); `lastRunTime` is not advanced so the event will retry.
4. **Health check event**: After a successful discovery with at least one resource, `ensureHealthCheckEventExists(projectId, channelId)` creates a scheduler event `HEALTH_CHECK:{projectId}` (period 60 min) if it does not exist.

### Timing

- `ResourceDiscoveryTiming.saveStartTime(projectId)` is called at start (Redis); `completeTiming(projectId, endObject)` is called on completion (reads Redis, writes to optional JSON file). GET `/api/resource-discovery/timing` returns data from that file.

## Health checks

### Flow

1. **Trigger**: The resource health loop (or manual trigger) runs when the HEALTH_CHECK event is due.
2. **Fetch resources**: `healthCheckHandler` calls DB Gateway `getMonitoredResources(projectId)`; filters to resources that have `checker` or `healthCheckCommand`.
3. **Execute**: `llmAIClient.executeHealthChecks(resources, projectId)` POSTs to LLM service `/api/v3/ai/execute-health-checks` with `project_id` and list of `{ resource_id, resource_name, checker }`. The LLM runs scripts in a Daytona sandbox and returns `results` (e.g. `resource_id`, `exit_code`, `output`, `error`).
4. **Status mapping**: Exit code → status: `0` → UP (true), `1–99` → DOWN (false), `-1` or `>= 100` → UNKNOWN (null).
5. **Update DB**: For each result, `dbGatewayClient.updateMonitoredResourceStatus(resource_id, { status, lastMonitoredAt })` is called.

Health checks are performed **synchronously** via the LLM service in this service; there is no SQS queue used by ace-ops-scheduler for health check execution (the LLM/commands stack may use other mechanisms internally).

## Handlers and services

| Handler / Service | Role |
|-------------------|------|
| **resourceDiscoveryHandler** | executeResourceDiscovery (lock, call orchestrator, update event, ensure HEALTH_CHECK event); updateResourceDiscoveryEvent (backoff/success); applyRetryPeriodForEvent. |
| **discoveryOrchestrator** | executeDiscovery (config → seeds → explore → reconcile → generate-scripts → batch upsert → stale reconcile). |
| **cicdSeedExtractor** | extractSeeds(infraFiles): regex-based extraction of resource identifiers from YAML/Terraform etc. |
| **healthCheckHandler** | executeHealthChecks: get resources → LLM execute-health-checks → map exit codes → updateMonitoredResourceStatus. |
| **llmAIClient** | generateScripts, exploreResources, executeHealthChecks (HTTP to LLM_API_URI). |
| **DBGatewayClient** | getMonitoredResources, updateMonitoredResourceStatus, incrementMisses, deleteMonitoredResource, createEvent (for HEALTH_CHECK), getChannelByProjectId, getChannelIdBySlackId, etc. |

## Environment and feature flags

- **AGENTIC_EXPLORATION_ENABLED**: If not `'false'`, discovery runs agentic exploration (LLM explore-resources); otherwise only CI/CD seeds are used (and if no seeds, discovery returns "No resources discovered").
- **LLM_API_URI**, **LLM_TIMEOUT_MS**, **LLM_EXPLORATION_TIMEOUT_MS**, **LLM_HEALTH_CHECK_TIMEOUT_MS**: Used by llmAIClient for LLM service calls.

## Related

- [Events and scheduling](./events-and-scheduling.md) for how RESOURCE_DISCOVERY and HEALTH_CHECK events are scheduled.
- [Dependencies and integrations](./dependencies-and-integrations.md) for DB Gateway and LLM service.
- [API and routes](./api-and-routes.md) for manual trigger and timing endpoint.
