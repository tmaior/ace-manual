# Features and capabilities

## 1. Slack command execution (queue-driven)

- **Flow**: ace-slackbot (or backend) sends a message to **CommandsList.fifo** with `conversa_id`, `commands` (array of lines), `ts`, `channel_id`, `client_id`, `project_id`. commands-api polls the queue, parses the message, runs safety check, creates a temp script, loads project secrets from DB Gateway, executes the script with timeout, sends output to **CommandsOutputs.fifo**, and logs to MongoDB.
- **Safety**: If `SAFETY_ENABLED` is on, script is checked via Safety API or Bedrock agent; unsafe scripts are blocked and Slack is notified (thread + optional logs channel).
- **Debug mode**: If the project/channel has debug enabled (via DB Gateway), debug messages (script preview, safety result) are sent to the Slack thread; execution errors could be sent to thread and logs channel.
- **Secrets**: Env vars for execution come from DB Gateway `GET /api/secrets/:projectId` (only when `project_id` is present).

## 2. Resource health check execution

- **Flow**: ace-ops-scheduler (or another producer) sends messages to **ResourceHealthChecks.fifo** with `command`, `resource_id`, `project_id`, `client_id`, `correlation_id`. commands-api runs the command with project secrets, captures exit code and output, and sends a result message to **ResourceHealthCheckResults.fifo** (`resource_id`, `exit_code`, `output`, `error`, `correlation_id`, `execution_time`, `project_id`, `client_id`).
- **Isolation**: Fully separate from Slack command flow (different controller and queue); same timeout and env handling as command execution.
- **Debug**: When `DEBUG_MONITORED_RESOURCES=true`, incoming/outgoing messages and results are appended to a JSON file and exposed via `/api/resource-health-debug/*` and dashboard pages.

## 3. Docs-sync (Knowledge Base) worker

- **Flow**: Backend (or admin) sends a message to the **DocsSync** FIFO queue with `commands` (array of script lines) and `project_id`. The docs-sync worker (Worker Thread inside the main process when `QUEUE_DOCS_SYNC_URL` is set) consumes the queue, optionally runs safety check, then runs the script with AWS env vars (or project secrets if not skipped). No Slack output and no CommandsOutput queue.
- **Purpose**: Run KB sync scripts (clone repo, `aws s3 sync`, Bedrock ingestion) without blocking the API or posting to Slack.
- **Optional**: Can be run as a separate process (`npm run start:docs-sync-worker`) instead of the in-process Worker Thread.

## 4. Logging and observability

- **MongoDB**: CommandsLogs (command, stdout, stderr, duration, correlationId), QueueLogs (read/delete/send, queue URL, duration), MessagesLogs (correlationId, deduplicationID, message).
- **API**: `/api/logs/commands`, `/api/logs/queues`, `/api/logs/messages` with optional `startDate`/`endDate`; `/api/logs/commands/:id`, `/api/logs/queues/:id`, `/api/logs/messages/:id`; `/api/logs/related/:correlationId` to get all logs for a correlation ID.
- **Resource health debug**: When `DEBUG_MONITORED_RESOURCES=true`, file-based log and endpoints to get/clear messages and send test messages.

## 5. HTTP API (read-only and debug)

- **Health**: `GET /health`, `GET /api/health` return 200 and `{ ok: true, health: 'healthy' }`.
- **Logs**: See above; require MongoDB.
- **Resource health debug**: `POST /api/resource-health-debug/send-message`, `GET /api/resource-health-debug/messages`, `DELETE /api/resource-health-debug/messages` (only when `DEBUG_MONITORED_RESOURCES=true`; otherwise 403).
- **Dashboard**: Static files under `/dashboard` (e.g. index.html, resource-health-logs.html, resource-health-debug.html).
- **Direct command execution**: `CommandsController.runHTTPCommands` exists (safety check + execution) but is **not** mounted on any route in the current codebase; `API_ENABLED` in constants is unused for routing. Enabling it would require adding a route and protecting it (e.g. auth, network).

## 6. Feature flags and behaviour

- **SAFETY_ENABLED**: When on, every script (Slack and docs-sync) is checked via Safety API or Bedrock; unsafe scripts are blocked.
- **DEBUG_MODE / debug per project**: DB Gateway exposes debug per channel; when true, debug and error details can be sent to Slack thread and logs channel.
- **DEBUG_MONITORED_RESOURCES**: Enables resource health debug logging (file + API + dashboard).
- **API_ENABLED**: Defined in constants but no route uses it in the current app; reserved for future HTTP command endpoint.

## 7. Graceful shutdown

- On SIGTERM/SIGINT: stop docs-sync worker thread (if any), stop both queue listeners (CommandsList and ResourceHealthChecks), close HTTP server, then exit. Ensures in-flight messages are not left in an inconsistent state where possible.
