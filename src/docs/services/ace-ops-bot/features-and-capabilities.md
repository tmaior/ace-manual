# Features and capabilities

## Slack event handling

- **App mention (channel, no thread)** – User @mentions the bot in a channel. Bot authenticates the user (session or DB Gateway login), checks project association for that channel, registers the message as a new thread with origin `OPS`, then either runs a built-in command or sends the message to the stack-backend omni ingest. LLM reply is posted in a new thread via callback.
- **Message in thread** – Only processed if the thread is a registered ACE thread and origin is one of `OPS`, `PAYLOADS`, or `CRON`. Same auth and project check; then command or send to LLM; reply is posted in the same thread via callback.
- **DM** – Messages in direct messages (`channel_type === 'im'`). No channel-based project; project is derived from the user’s first project. Commands or send to LLM; reply is sent back in the DM.

## Built-in commands

Handled in `customerMessages.checkCommands` (before sending to LLM):

| Command | Behavior |
|---------|----------|
| **reset session** | Calls DB Gateway Slack login again and restarts Redis session for the user; replies “Session reset successfully!” or an error message. |
| **get docs** | Fetches monitoring documentations for the channel from ace-db-gateway and uploads each as a file to the Slack channel with `files.uploadV2`. Replies “You don’t have any docs.” if none. |

## LLM integration (omni ingest)

- User message (after removing bot mention) is sent to **ace-stack-backend** `POST /api/omni/ingest` with:
  - Message text, channel and user IDs, thread `ts`, origin `ace_ops`, JWT, `client_id`, `project_id`, optional `knowledge_base_id` (from DB Gateway, cached in Redis per project).
- Payload includes `callback_url: {SELF_URL}/send-thread-message` so the backend can POST the LLM reply back; ops-bot then posts it into the same thread with `messageToThread`.

## Thread registration and origin filtering

- On **app_mention** (non-thread), the bot registers the message with ace-db-gateway `POST /api/channel-thread/register` with `origin: 'OPS'`.
- On **message** (thread), the bot calls `isRegisteredMessage` and only continues if the thread’s origin is `OPS`, `PAYLOADS`, or `CRON`. This avoids handling arbitrary channel threads that are not ACE-managed.

## Mock / invalid token behavior

- If `SLACK_BOT_TOKEN` starts with `xoxb-mock-token`, the app runs in a mock mode: Bolt authorize returns a fixed mock; `authUser` returns a mock user and token; `messageToChannel` and `messageToThread` log and return success without calling Slack API. Useful for local or CI runs without a real Slack app.

## HTTP endpoints (summary)

- **POST /send-message** – Body: `channel`, `message`. Posts a message to a Slack channel. Used by external callers or integrations.
- **POST /send-thread-message** – Body: `channelID`, `ts`, `message`. Posts a message into an existing thread. Used as the callback by ace-stack-backend for omni/LLM replies.

Details: [Express routes and callbacks](./express-routes-and-callbacks.md).

## Debug mode

- `customerMessages.debugEnabled(channelSlackID)` calls ace-db-gateway `GET /check/debug-project/{channelSlackID}`. The result is passed in the ingest payload; behavior of “debug” is implemented on the stack-backend side.
