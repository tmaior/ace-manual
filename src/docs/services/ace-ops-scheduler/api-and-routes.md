# API and routes

This document lists the HTTP endpoints implemented by ace-ops-scheduler (Express routes). Base path is the service root; no global `/api` prefix is applied to the app except where noted below.

## Health and test

| Method | Path | Description |
|--------|------|-------------|
| GET | `/health` | Returns `{ health: true }`. Use for liveness/readiness. |
| GET | `/api/resource-discovery/timing/test` | Test endpoint; returns `{ message, timestamp }`. |

## Scheduler events

| Method | Path | Description |
|--------|------|-------------|
| POST | `/api/events` | Create a scheduler event. **Body**: `channel_id` (string), `period` (number, minutes), `check` (string). Service resolves `client_id` and `project_id` via contextResolver (DB Gateway). Returns `201` with created event or `400`/`500` on error. |

## Payloads (webhook ingest)

| Method | Path | Description |
|--------|------|-------------|
| POST | `/api/payloads` | Webhook ingest. **Body**: arbitrary JSON (e.g. alarm payload). **Query**: `project_id`, or `project_name`, or `channel_id`. If only `channel_id` is provided, project is resolved via DB Gateway `getChannelProject`. Channel is resolved from query, or default channel for project, or `FALLBACK_CHANNEL_ID` / default `CHANNEL_ID`. Sends payload to `STACK_BACKEND_URL/api/omni/ingest` with `origin: "payloads"`, `callback_url: SELF_URL/scheduler-callback`, and optional `knowledge_base_id` for the project. Returns `200` with `{ ok, projectId, projectName, channel }` or `400`/`502`/`503`. |
| GET | `/api/payloads/payloadobject` | Returns the last received payload object (in-memory; for debugging). |

## Resource discovery and health

| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/resource-discovery/timing` | Returns timing data for resource discovery runs. Reads from `ResourceDiscoveryTiming.readTimingFile()` (JSON file). Returns `200` with array of timing records or `500` on error. |
| POST | `/api/resource-health/trigger` | Manual trigger for resource health or discovery. **Body**: `projectId` (required), `type` (optional, one of `HEALTH_CHECK`, `RESOURCE_DISCOVERY`; default `HEALTH_CHECK`). Looks up the corresponding scheduler event by check prefix `{type}:{projectId}`; runs `executeResourceDiscovery` or `executeHealthChecks`; updates event execution. Returns `200` with `{ ok, message, eventId }` or `400`/`404`/`500`. |

## Scheduler callback (omnichannel)

| Method | Path | Description |
|--------|------|-------------|
| POST | `/scheduler-callback` | Callback called by omnichannel when a scheduler-originated conversation completes. **Body**: `conversa_id`, `message`, `message_type`, `ts`, `channelID`, `bot_name`. If `conversa_id` matches `scheduler-event-(\d+)`, the event ID is extracted and `SchedulerBLL.updateEventExecution(eventId, Date.now(), resultSummary)` is called. Returns `200` with `{ ok, received }` or `500`. |

## Static and mount summary

- **Dashboard**: Static files are served under `/dashboard` (e.g. `public/`).
- **Routes**: `/api/events` and `/api/payloads` are mounted from `routes/events.js` and `routes/payloads.js`; all other endpoints above are defined in `routes/index.js`.

## Environment variables used by routes

- `STACK_BACKEND_URL`, `SELF_URL`: omnichannel ingest and callback URL.
- `PORT`: server port (default 3000).
- `CHANNEL_ID`, `FALLBACK_CHANNEL_ID`: default Slack channel when project has no channel.
- DB Gateway and Redis are used by services; see [dependencies-and-integrations.md](./dependencies-and-integrations.md).
