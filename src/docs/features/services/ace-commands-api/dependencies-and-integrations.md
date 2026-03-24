# Dependencies and integrations

## External dependencies (runtime)

| Dependency | Purpose |
|------------|---------|
| **MongoDB (DocumentDB)** | Persist command logs, queue logs, message logs. Connection via `MONGO_URL`; TLS and auth for non-local. |
| **AWS SQS** | FIFO queues: CommandsList, CommandsOutputs, ResourceHealthChecks, ResourceHealthCheckResults, optional DocsSync. Region from `AWS_REGION` (default us-east-1). |
| **ace-db-gateway** | Secrets by project_id, debug mode by Slack channel, safety level by project, channel/project lookup. Token: `COMMANDS_API_DB_GATEWAY_TOKEN`. |
| **ace-slackbot** | Send message, thread message, error to logs channel; debug and unsafe-command notifications. URL: `SLACKBOT_URL`. |
| **Safety API or AWS Bedrock** | When safety is enabled: classify script as safe/unsafe. Safety API URL: `SAFETY_API_URL`; or Bedrock agent in SafetyService (agentId/alias per safety level). |

## Main libraries (package.json)

| Package | Version (approx) | Use |
|---------|------------------|-----|
| **express** | ^4.21 | HTTP server and routes. |
| **mongoose** | ^8.14 | MongoDB/DocumentDB connection and models (CommandsLogs, QueueLogs, MessagesLogs). |
| **dotenv** | ^16.4 | Load `.env`. |
| **@aws-sdk/client-sqs** | ^3.699 | SQS ReceiveMessage, SendMessage, DeleteMessage. |
| **@aws-sdk/client-bedrock-agent-runtime** | ^3.821 | InvokeAgent for safety check (SafetyService). |
| **p-limit** | ^3.1 | Concurrency limit (e.g. 5) when processing multiple SQS messages in a batch. |
| **uuid** | ^13 | UUIDs (e.g. resource health debug correlation_id). |
| **heapdump** | ^0.3 | Optional heap dumps for debugging. |
| **nodemon** | dev | Development auto-restart. |

## Who the app talks to

| Service / system | Direction | Purpose |
|------------------|-----------|---------|
| **ace-db-gateway** | Outbound | GET secrets by project_id; GET debug mode by channel; GET safety level by project; GET channel/project. |
| **ace-slackbot** | Outbound | POST send-message, send-thread-message; error and debug notifications. |
| **Safety API** | Outbound | POST script content for safety check (when not using Bedrock). |
| **AWS Bedrock** | Outbound | InvokeAgent for safety classification (when using Bedrock in SafetyService). |
| **AWS SQS** | Outbound / consumer | Receive from CommandsList, ResourceHealthChecks, DocsSync; send to CommandsOutputs, ResourceHealthCheckResults. |
| **MongoDB/DocumentDB** | Outbound | Persist logs. |
| **Slack** | Indirect (via ace-slackbot) | User-facing messages and errors. |

## Internal structure (no external call)

- **FileService**: Temp scripts under `/tmp`, last timestamp file.
- **resourceHealthLogger**: When DEBUG_MONITORED_RESOURCES is on, append to JSON file at `PATHS.RESOURCE_HEALTH_LOGS` (`/tmp/commands-workspace/resource-health-messages.json`).

## Requirements

- **Node.js**: 16+ (runtime); Dockerfile uses Node 20.
- **MongoDB**: Compatible with Mongoose (e.g. DocumentDB with MongoDB compatibility).
- **AWS**: Credentials (env or IAM role) with SQS receive/send/delete for the queues used; Bedrock InvokeAgent if safety uses Bedrock; S3 and Bedrock ingestion for docs-sync worker if used.
- **Network**: Reachability to ace-db-gateway, ace-slackbot, Safety API (if used), AWS APIs (SQS, Bedrock, and for docs-sync S3/Bedrock).
