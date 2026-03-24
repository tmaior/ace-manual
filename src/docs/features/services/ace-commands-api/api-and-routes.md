# API and routes

## Base URL and prefix

- **Server**: Listens on `PORT` (default 3000).
- **API prefix**: `/api` for all API routes below (e.g. `/api/health`, `/api/logs/...`).
- **Dashboard**: Static files under `/dashboard` (e.g. `/dashboard/index.html`).

## Public / unauthenticated endpoints

Authentication is not implemented in the app; these endpoints are reachable by anyone with network access. In production, consider Ingress/auth or network policies.

### Health

| Method | Path | Description |
|--------|------|-------------|
| GET | `/health` | Returns `200` and `{ ok: true, health: 'healthy' }`. Used by liveness probe. |
| GET | `/api/health` | Same body. Used by readiness probe. |

### Logs (MongoDB)

All require MongoDB; query params are optional.

| Method | Path | Query | Description |
|--------|------|--------|-------------|
| GET | `/api/logs/commands` | `startDate`, `endDate` (epoch ms) | List command logs in date range, sorted by date desc, limit 1000. |
| GET | `/api/logs/commands/:id` | - | Single command log by ID. 404 if not found. |
| GET | `/api/logs/queues` | `startDate`, `endDate` | List queue logs in date range, limit 1000. |
| GET | `/api/logs/queues/:id` | - | Single queue log by ID. 404 if not found. |
| GET | `/api/logs/messages` | `startDate`, `endDate` | List message logs; limit 1000. |
| GET | `/api/logs/messages/:id` | - | Single message log by ID. |
| GET | `/api/logs/related/:correlationId` | - | All logs (commands, queues, messages) with the given correlationId. |

### Resource health debug (conditional)

When **DEBUG_MONITORED_RESOURCES** is not `true`, these return **403** with a message that debug is not enabled.

| Method | Path | Body / Query | Description |
|--------|------|----------------|-------------|
| POST | `/api/resource-health-debug/send-message` | JSON: `command`, `resource_id`, `project_id`, `client_id`, optional `correlation_id` | Sends a test message to ResourceHealthChecks queue. |
| GET | `/api/resource-health-debug/messages` | `limit`, `offset` (default 100, 0) | Returns paginated messages from file-based resource health log. |
| DELETE | `/api/resource-health-debug/messages` | - | Clears all entries in the resource health log file. |

## Static dashboard

| Path | Description |
|------|-------------|
| `/dashboard` | Served from `public/` (e.g. index.html). |
| `/dashboard/index.html` | Dashboard index. |
| `/dashboard/resource-health-logs.html` | Resource health logs viewer (when debug enabled). |
| `/dashboard/resource-health-debug.html` | Resource health debug: send test message, view/clear logs. |

## Not exposed

- **HTTP command execution**: `CommandsController.runHTTPCommands` exists (safety check + script execution) but is **not** mounted on any route. Enabling it would require adding e.g. `POST /api/run` (or similar) and securing it (auth, API_ENABLED, etc.).
