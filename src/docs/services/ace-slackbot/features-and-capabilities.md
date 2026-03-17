# Features and capabilities

## Slack events

- **app_mention**: Triggered when the bot is @mentioned in a channel. Only top-level mentions are processed (events with `thread_ts` are ignored). User is authenticated, project association checked, thread registered via DB Gateway, then message is handled (commands or sendToLLM).
- **message** (channel): Processed only when the message is inside a thread (`thread_ts` present). Same auth and project check; then `isRegisteredMessage` ensures the thread was started by ACE. Bot-originated messages are allowed only if the assumed user is authorized for the project.
- **message** (DM): When `channel_type === 'im'`, handled by `dmMessage`. No thread registration; project is resolved by user’s projects (first project used).

## Built-in commands (text in message)

Handled in `customerMessages.checkCommands` (and thus in both mention and DM flows):

- **reset session**: Re-login via DB Gateway, restart Redis session, reply “Session reset successfully!…”
- **get docs**: Load session, call DB Gateway for discovery docs by Slack channel, upload last 10 docs as files to the channel (Slack `files.uploadV2`).
- **set model**: Parse “set model &lt;model_id&gt;”, store in Redis `model:channel:{channelID}` and optionally in DB Gateway `POST /api/channels/:channelID/model`. Reply with selected model.
- **get model**: Read from Redis `model:channel:{channelID}`; if missing, fallback to DB Gateway `GET /api/channels/:channelID/model`. Reply with current model or “No model selected”.

If no command matches, the message is sent to the LLM via `sendToLLM` (stack-backend ingest).

## HTTP endpoints (Express, mounted on Bolt receiver)

- **GET /health**: Returns `{ ok: true, status: 'healthy' }`.
- **POST /send-message**: Body `{ channel, message }`. Sends `message` to Slack `channel` via `chat.postMessage`. Messages over ~11k chars are sent as file upload. Special-case for a test channel ID can skip certain default messages.
- **POST /send-thread-message**: Body `{ channelID, ts, message }`. Sends `message` in thread `ts` in `channelID`. If project debug is not enabled, messages that look like debug-only (e.g. starting with “[ENHANCED_SUMMARY_NODE]”, “REASONING:”, “ANOTATION LIST”, “EXECUÇÃO SQS”, “TO-DO LIST”) are not sent. Used by backend as callback for LLM/command output.

## Authentication and session

- **Login**: DB Gateway `POST /login/external` with `{ providerType: 'slack', providerId: slackID }` returns user and JWT. Token and user are stored in Redis under `SESSION_PREFIX + userId` with TTL derived from token expiry.
- **Project association**: For channel messages, `DBGateway.checkProjectUsersSlackIds(channelId, userId)` is used; if false, user is told they are not associated with the project. Same check is used to allow or block bot-originated messages (by the user ID the bot is assuming).

## Model and knowledge base

- **Model**: Per-channel preference in Redis (`model:channel:{channelID}`) and optionally in DB Gateway. Sent in ingest payload as `model_id`; backend uses it for LLM selection.
- **Knowledge base**: Fetched per project from DB Gateway `GET /api/projects/:projectId/knowledge-base`, cached in Redis with key `{projectId}:KnowledgeBase` and TTL 24h. Sent as `knowledge_base_id` in ingest payload.

## Debug mode

- **Debug project**: Routes and customerMessages call DB Gateway `GET /check/debug-project/:channelSlackID` to know if the project has debug enabled. If not, `/send-thread-message` filters out messages that match debug-only patterns. `DEBUG_TRUE_FIXED` env can force debug on for all channels.

## Logging and errors

- **Logger** (`controller/Logger.js`): Structured console logs with level (INFO, ERROR, DEBUG, WARNING), context string, and message. Uses chalk for colors.
- **Unhandled rejections**: Ignored when error indicates `invalid_auth` (e.g. mock token); otherwise logged. Process is not exited on mock auth to keep container up.
- **Uncaught exceptions**: Logged; no explicit exit in the snippet.

## Mock mode

When `SLACK_BOT_TOKEN` starts with `xoxb-mock-token`, the app uses a mock authorize and skips real Slack API calls for `messageToChannel` and `messageToThread` (returns success and a fake ts). Used for local/testing without a real Slack app.
