# ace-commands-api – Overview

## What it is

**ace-commands-api** is a Node.js microservice in the ACE (Automated Cloud Engineer) platform. It securely executes shell commands in controlled environments, either triggered by Slack (via SQS) or by internal systems (resource health checks, docs-sync). It does **not** expose a public HTTP API for arbitrary command execution by default; execution is driven by queues and optional HTTP when explicitly enabled.

## Purpose

- **Execute commands from Slack**: The Slack bot (ace-slackbot) sends command requests to an SQS queue; commands-api consumes the queue, runs the script (with safety checks and project secrets), and sends the output back to another queue for the bot to post in the thread.
- **Resource health checks**: A separate SQS flow sends health-check commands (e.g. AWS CLI, scripts) per monitored resource; commands-api runs them and sends results to an output queue for ace-ops-scheduler to process and update resource status.
- **Docs-sync (Knowledge Base)**: When configured, a worker (in-process or separate) consumes a dedicated SQS queue and runs KB sync scripts (clone repo, S3 sync, Bedrock ingestion) without Slack output.
- **Logging and traceability**: All command runs, queue reads/writes, and messages are logged to MongoDB for debugging and auditing (correlation IDs, duration, stdout/stderr).

## High-level architecture

- **HTTP server**: Express on `PORT` (default 3000). Serves `/health`, `/api/health`, `/api/logs/*`, `/api/resource-health-debug/*`, and static dashboard under `/dashboard`. Does **not** mount a route for direct command execution unless explicitly added and `API_ENABLED=true`.
- **SQS consumers** (in main process):
  - **CommandsList.fifo**: Commands from Slack; each message contains `conversa_id`, `commands` (array of script lines), `ts`, `channel_id`, `client_id`, `project_id`. After safety check and execution, output is sent to **CommandsOutputs.fifo**.
  - **ResourceHealthChecks.fifo**: Health check jobs with `command`, `resource_id`, `project_id`, `client_id`, `correlation_id`. Results are sent to **ResourceHealthCheckResults.fifo**.
- **Docs-sync**: When `QUEUE_DOCS_SYNC_URL` is set, a **Worker Thread** (or separate process) listens to that FIFO queue and runs docs-sync jobs (no Slack, no CommandsOutput).
- **External calls**: ace-db-gateway (secrets, debug mode, safety level, channel/project), ace-slackbot (send message, thread message, errors), optional Safety API or AWS Bedrock for command safety.

## Main components

| Component | Role |
|-----------|------|
| **CommandsController** | Processes queue messages for Slack commands: parse message, safety check, temp script, execute, send response to CommandsOutput queue, log to MongoDB. |
| **ResourceHealthController** | Processes ResourceHealthChecks queue: parse message, get project env vars, run command, send result to ResourceHealthCheckResults queue, file-based debug log when DEBUG_MONITORED_RESOURCES is on. |
| **DocsSyncController** | Processes docs-sync queue: run KB sync script (optional safety check), no Slack/output queue. |
| **Queue** | SQS wrapper: poll (with backoff), process message, delete on success, send to another queue; uses correlation/deduplication IDs. |
| **CommandsExecutor** | Runs bash script with timeout; env vars from DB Gateway (by project_id); stdout/stderr captured; timeout kills process group. |
| **SafetyService** | Optional: call external Safety API or AWS Bedrock agent to classify script as safe/unsafe; block execution and notify Slack if unsafe. |
| **DBGateway** | HTTP client to ace-db-gateway: get secrets by project_id, debug mode by channel, safety level by project, channel/project resolution. |
| **SlackService** | HTTP client to ace-slackbot: send message, thread message, error to logs channel, debug/unsafe notifications. |
| **FileService** | Create temp script under `/tmp`, cleanup; update last timestamp file for monitoring. |
| **Logger / resourceHealthLogger** | MongoDB models (CommandsLogs, QueueLogs, MessagesLogs) and file-based JSON log for resource health debug. |

## What you can do with this app

- Run shell commands triggered by Slack (with safety check and project secrets).
- Run resource health check commands (triggered by ace-ops-scheduler) and return exit code and output.
- Run Knowledge Base sync scripts (clone, S3 sync, Bedrock ingestion) via dedicated queue.
- Query command/queue/message logs by date or correlation ID via `/api/logs/*`.
- When `DEBUG_MONITORED_RESOURCES=true`, use debug UI and API to send test health-check messages and view/clear resource health logs.

## Related documentation

- [Dependencies and integrations](./dependencies-and-integrations.md)
- [Features and capabilities](./features-and-capabilities.md)
- [Environment and configuration](./environment-and-configuration.md)
- [Build, deploy and CI/CD](./build-deploy-and-cicd.md)
- [API and routes](./api-and-routes.md)
- [Queues and workers](./queues-and-workers.md)
- Repository docs: `ace-commands-api/docs/` (e.g. docs-sync-worker.md, resource-health-testing-guide.md)
