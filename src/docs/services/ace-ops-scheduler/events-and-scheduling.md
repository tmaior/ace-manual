# Events and scheduling

ace-ops-scheduler runs two independent in-process loops (every 60 seconds). Events are stored in the configuration database and accessed via **ace-db-gateway**. There are no external job queues (e.g. Agenda/SQS) used for the main scheduler logic in this service.

## Event types and separation

- **Regular events**: Any scheduler event whose `check` field does **not** start with `RESOURCE_DISCOVERY:` or `HEALTH_CHECK:`. These are processed by the first loop: payload is sent to omnichannel ingest; execution result is updated via DB Gateway.
- **Resource health events**: Events whose `check` starts with `RESOURCE_DISCOVERY:{projectId}` or `HEALTH_CHECK:{projectId}`. Processed by the second loop: discovery runs the discovery pipeline (and optionally creates a HEALTH_CHECK event); health check runs health checks via LLM and updates monitored resource status.

## SchedulerBLL (controller)

| Method | Purpose |
|--------|---------|
| `getAllEvents()` | Fetch all scheduler events from DB Gateway (no filter). |
| `getRegularEvents()` | Fetch events with `excludeCheckPrefix: 'RESOURCE_DISCOVERY:,HEALTH_CHECK:'` and limit; used by the regular-events loop. |
| `getResourceHealthEvents()` | Fetch all events (limit), then filter in memory for `check.startsWith('RESOURCE_DISCOVERY:')` or `check.startsWith('HEALTH_CHECK:')`; used by the resource-health loop. |
| `saveEvent(channel_id, period, check)` | Validate inputs; resolve `clientId` and `projectId` via `contextResolver.resolveClientAndProject(channel_id)`; call DB Gateway to create event. |
| `updateEventExecution(eventId, newLastRunTime, lastRunResult)` | Update event's `lastRunTime` and `lastRunResult` via DB Gateway. |

## Regular events loop (index.sequelize.js)

1. Every 60 s: `SchedulerBLL.getRegularEvents()`.
2. For each event: compute `periodMs = period * 60 * 1000`; if `!lastRunTime || (now - lastRunTime) > periodMs`, the event is due.
3. For due events: call `sendToLLM({ id, channel_id, period, check, last_run_time, last_run_result, client_id, project_id })`. `sendToLLM` builds an ingest payload and POSTs to `STACK_BACKEND_URL/api/omni/ingest` with `channel_type: 'scheduler'`, `callback_url: SELF_URL/scheduler-callback`, and optional `knowledge_base_id` from DB Gateway (cached in Redis).
4. On success or failure: `SchedulerBLL.updateEventExecution(event.id, now, result)`.

## Resource health loop (index.sequelize.js)

1. Every 60 s: `SchedulerBLL.getResourceHealthEvents()`.
2. For each due event (same period/lastRunTime logic):
   - **RESOURCE_DISCOVERY:** Update execution with "RUNNING"; call `executeResourceDiscovery(event)` in a fire-and-forget manner (non-blocking). On discovery completion, `resourceDiscoveryHandler` updates the event (lastRunTime, period, lastRunResult) and may create a HEALTH_CHECK event. On error before/during discovery, `applyRetryPeriodForEvent(event)` applies exponential backoff for the period.
   - **HEALTH_CHECK:** Await `executeHealthChecks(event)`; then `updateEventExecution` with the result message.
3. Errors for RESOURCE_DISCOVERY trigger period backoff and lastRunResult update; for HEALTH_CHECK only lastRunResult is updated.

## Creating events via API

- **POST /api/events** with `channel_id`, `period`, `check`. The service resolves `client_id` and `project_id` from `channel_id` via DB Gateway (contextResolver). Resource health events (e.g. `check: "HEALTH_CHECK:123"`) are typically created by the discovery handler or by other tooling; regular events can be created by clients with any `check` string that does not use the resource health prefixes.

## Due-time logic

- An event is **due** when it has never run (`!lastRunTime`) or when `(now - lastRunTime) > period * 60 * 1000` (period in minutes).
- Both loops run on a fixed 60 s interval; actual execution time may be up to ~60 s after the theoretical due time.

## Related

- [Resource health (discovery and health checks)](./resource-health.md) for RESOURCE_DISCOVERY and HEALTH_CHECK behavior.
- [API and routes](./api-and-routes.md) for POST /api/events and /api/resource-health/trigger.
