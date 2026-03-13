# Environment and configuration

## Required environment variables

| Variable | Purpose |
|----------|---------|
| **MONGO_URL** | MongoDB/DocumentDB connection string. Required for startup; app exits if connection fails. |
| **PORT** | HTTP server port (default 3000). |
| **ACE_DB_GATEWAY_ENDPOINT** | Base URL for ace-db-gateway (e.g. `http://ace-db-gateway-service`). |
| **COMMANDS_API_DB_GATEWAY_TOKEN** | Token sent as `Authorization` (or `Bearer`) to ace-db-gateway for secrets and debug/safety endpoints. |
| **QUEUE_COMMANDSLIST** | SQS FIFO URL for incoming Slack commands (CommandsList.fifo). |
| **QUEUE_COMMANDSOUTPUT** | SQS FIFO URL for command output (CommandsOutputs.fifo). |
| **RESOURCE_HEALTH_CHECKS_QUEUE_URL** | SQS FIFO URL for resource health check commands. |
| **RESOURCE_HEALTH_CHECK_RESULTS_QUEUE_URL** | SQS FIFO URL for health check results. |
| **AWS_REGION** | AWS region for SQS (and Bedrock if used). Default us-east-1 in Queue and SafetyService. |

For **docs-sync worker** (when used):

- **QUEUE_DOCS_SYNC_URL** – SQS FIFO URL for docs-sync jobs. If set, the main process starts a Worker Thread that consumes this queue.
- AWS credentials (or IAM role) with S3 and Bedrock ingestion permissions for the sync script.

## Optional environment variables

| Variable | Purpose | Default / behaviour |
|----------|---------|---------------------|
| **SLACKBOT_URL** | ace-slackbot base URL (send message, thread, errors). | `http://ace-slackbot-service` |
| **SAFETY_API_URL** | External safety API URL (POST script for classification). | `http://dev-commands-sec.ace.ezops.cloud/generate` (SafetyService may use Bedrock instead) |
| **SAFETY_ENABLED** | Enable safety check (on/true = enabled). | Off/false |
| **API_ENABLED** | Reserved for future HTTP command execution route. | false (no route uses it currently) |
| **DEBUG_MODE** | General debug flag. | off |
| **DEBUG_MONITORED_RESOURCES** | Enable resource health debug (file log + API + dashboard). | false |
| **LOGS_CHANNEL_ID** | Slack channel ID for error/log alerts. | Unset (SlackService may skip or use default) |
| **COMMAND_EXECUTION_TIMEOUT** | Script execution timeout in milliseconds. 0 = no timeout. | 120000 (2 minutes) |
| **LOCAL_DEV** | If "true", MongoDB connection skips TLS and authMechanism (local dev). | false |
| **LOCAL_AWS_ENDPOINT** | Override SQS endpoint (e.g. LocalStack). Used only when queue URL does not look like AWS SQS and `blockLocalStack` is false. | Unset |
| **DEBUG_QUICK_FIX** | If "true", SlackService uses alternate debug check. | false |
| **PASSWORDBOT_URL** | Legacy; not used for secrets (secrets come from DB Gateway). | `http://ace-passwordbot-service` |

## Paths (constants)

- **TEMP_WORKSPACE**: `/tmp/commands-workspace` (temp script dir; FileService uses `temp_commands_script.<ts>.sh` under project root).
- **LAST_TIMESTAMP**: `/tmp/last-timestamp.txt` (updated on each command run for monitoring).
- **RESOURCE_HEALTH_LOGS**: `/tmp/commands-workspace/resource-health-messages.json` (when DEBUG_MONITORED_RESOURCES is on).

## Execution environment (script runs)

- **PATH**: `/usr/local/bin:/usr/local/sbin:/usr/bin:/usr/sbin:/bin:/sbin:/usr/local/bin/aws`
- **HOME**: `/root`
- **Timeout**: From `COMMAND_EXECUTION_TIMEOUT` (default 120000 ms). Applied to Slack commands and resource health; docs-sync uses the same executor.

## MongoDB connection

- **TLS**: Enabled unless `LOCAL_DEV=true`. For DocumentDB, `tlsCAFile` is set to `config/global-bundle.pem` when not local.
- **Auth**: `authMechanism: 'SCRAM-SHA-1'` when not local.

## Secrets (AWS Secrets Manager)

In EKS, secrets are typically stored in AWS Secrets Manager at **ace/&lt;env&gt;/commands-api-secrets** and injected as env (e.g. via `get_secrets.sh` in the container or external-secrets). All of the variables above that are sensitive (MONGO_URL, tokens, AWS keys, etc.) should be provided via that secret or a secure mechanism, never committed.
