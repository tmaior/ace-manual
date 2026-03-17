# Features and capabilities

## Slack event handling

- **App mention** (`app_mention`): Only processed when the message is not in a thread (no `thread_ts`). User is authenticated (session or login via ace-db-gateway); project association is checked. Message is registered with DB Gateway (channel, ts, origin SEC). Then either a built-in command is run or the message is sent to the AI API and the reply is posted in the channel.
- **Message in thread** (`message`): Processed only if there is a `thread_ts` and the message is not from a bot. Same auth and project-association checks. Thread must be already registered (isRegisteredMessage); if not registered, the message is ignored. Then commands or sendToLLM and reply in thread.
- **Direct message** (`message` with `channel_type === 'im'`): No project association check; project is resolved from user’s projects (first project if any). Commands or sendToLLM; reply in DM.

## Built-in commands

- **reset session**: Calls ace-db-gateway slackLogin, then restarts Redis session with new token and TTL. Responds with "Session reset successfully!" or an error message.
- **get docs**: Fetches policy documentations for the channel’s project via DB Gateway (`identifier=policy`), writes each doc to a temp file, uploads to Slack with `files.uploadV2` (markdown), then responds. If no docs, responds "You don't have any docs."

## HTTP endpoints (routes)

Mounted on the Bolt ExpressReceiver router (same server/port as the Slack app):

- **POST /send-message**: Body `{ channel, message }`. Sends the message to the given channel. Returns 200 with `ok`, `ts`, `channel` or 400/500 with error.
- **POST /send-thread-message**: Body `{ channelID, ts, message }`. Sends the message in the thread identified by `ts`. Returns 200 with `ok`, `ts`, `channel` or 400/500 with error.

Used by other services to post messages or thread replies into Slack without going through Slack events.

## Session and authentication

- Sessions are stored in Redis under a configurable prefix (e.g. `session:bots:slack:`), keyed by Slack user ID. Value includes token and user info. TTL from login response (token expiry minus buffer).
- If session is missing when handling a message, bot attempts auto-login via DB Gateway; on success it starts a new session and continues. On failure it asks the user to type "reset session".
- Mock mode: if SLACK_BOT_TOKEN starts with `xoxb-mock-token`, auth and Slack API calls are stubbed (e.g. mock user, mock send success) for local/testing use.

## Channel-thread and conversa_id

- On mention, the bot registers the message with ace-db-gateway (registerMessage) so the thread is linked to the project with origin SEC.
- For sendToLLM, conversa_id is taken from isRegisteredMessage when the thread is registered and has a conversaId; otherwise it is synthesized as `{channelID}_{thread_ts or ts}`.
- Knowledge Base ID is resolved per project via DB Gateway and cached in Redis (key `{projectId}:KnowledgeBase`, TTL 86400 seconds).

## Limitations and notes

- The code references `debugEnabled(channelID)` in customerMessages.js (used for optional debug behavior in the request). This function is not defined in the scanned codebase; it may be missing or provided elsewhere. If missing, that path will throw at runtime when hit.
- App mentions in a thread are intentionally ignored (logged and return early).
- All DB Gateway and AI API calls depend on correct env (ACE_DB_GATEWAY_ENDPOINT, AI_API_URL) and valid tokens/session.
