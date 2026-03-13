# Queues and workers

## SQS queues used

All queues are **FIFO**. URLs come from environment variables (with fallbacks in code for dev; production should set them explicitly).

| Queue | Env var | Producer | Consumer (in commands-api) | Purpose |
|-------|---------|----------|-----------------------------|---------|
| **CommandsList** | QUEUE_COMMANDSLIST | ace-slackbot / backend | Main process (Queue listener) | Incoming Slack command requests (script + metadata). |
| **CommandsOutputs** | QUEUE_COMMANDSOUTPUT | commands-api | ace-slackbot | Command output and metadata to post in Slack thread. |
| **ResourceHealthChecks** | RESOURCE_HEALTH_CHECKS_QUEUE_URL | ace-ops-scheduler | Main process (separate Queue listener) | Health check commands (command, resource_id, project_id, client_id, correlation_id). |
| **ResourceHealthCheckResults** | RESOURCE_HEALTH_CHECK_RESULTS_QUEUE_URL | commands-api | ace-ops-scheduler | Health check result (exit_code, output, error, correlation_id, etc.). |
| **DocsSync** | QUEUE_DOCS_SYNC_URL | ace-stack-backend | Docs-sync worker (Worker Thread or separate process) | KB sync jobs (commands array + project_id). |

## Main process listeners

- **CommandsList**: Long-poll (WaitTimeSeconds 10), VisibilityTimeout 170 s, concurrency limit 5 (p-limit). Handler: `CommandsController.runQueueCommands`. On success, message is deleted; output is sent to CommandsOutputs with same correlation/deduplication IDs when possible.
- **ResourceHealthChecks**: Same pattern; VisibilityTimeout 170 s; handler: `ResourceHealthController.processHealthCheckCommand`. Completely independent of the CommandsList listener. Results sent to ResourceHealthCheckResults.

Both listeners use **adaptive backoff**: when no messages are received for 10 minutes, backoff increases (up to 30 s); when messages are received, backoff resets to 1 s.

## Docs-sync worker

- **When**: Started only if `QUEUE_DOCS_SYNC_URL` is set.
- **How (default)**: A **Worker Thread** is spawned from `index.js` (`workers/docsSyncWorkerThread.js`). It connects to MongoDB, creates a `Queue` instance for the DocsSync URL (WaitTimeSeconds 1, VisibilityTimeout 300), and calls `DocsSyncController.runDocsSyncJob` for each message.
- **Alternative**: Standalone process: `node workers/docsSyncWorker.js` (or `yarn start:docs-sync-worker`). Same logic but in a separate process; requires its own MongoDB and env (including QUEUE_DOCS_SYNC_URL and AWS credentials).
- **Shutdown**: Main process sends `shutdown` to the worker thread; thread stops the queue listener and posts `ready`; main process then terminates the thread. Standalone process handles SIGTERM/SIGINT and stops the listener.
- **Message format**: Body JSON with `commands` (array of script lines) and `project_id`. Script is joined with newlines and executed; no Slack or CommandsOutput.

## Queue class (controllers/Queue.js)

- **Constructor**: queueUrl, queueReadTime (default 10), visibilityTimeout (default 30), blockLocalStack (default true for main queues).
- **Methods**: `getMessages`, `listenForMessages(handlerFunction)`, `stopListening`, `sendMessage(message, options)`, `deleteMessage(message)`.
- **SQS**: Uses `@aws-sdk/client-sqs` (ReceiveMessage, SendMessage, DeleteMessage). FIFO messages use `MessageGroupId: "MessageForAce"` (hardcoded in sendMessage); deduplication ID from options or timestamp.
- **Logging**: Each read/delete/send is logged to MongoDB (QueueLogs, MessagesLogs) with correlation and deduplication IDs.

## Relation to ace-infra and Knowledge Base

- DocsSync queue is created by script **ace-infra/scripts/create-docs-sync-queue.sh** or Terraform module **terraform-library/docs-sync-queue**. See [Knowledge Base and resources](../../infrastructure/knowledge-base-and-resources.md).
- Backend enqueues sync jobs with script that uses GitHub token, repo, branch, S3 bucket, Bedrock KB/DataSource; worker runs that script in the container (with AWS CLI and gh available in the image).
