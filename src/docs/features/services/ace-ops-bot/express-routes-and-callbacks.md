# Express routes and callbacks

The Bolt app uses an **ExpressReceiver**. Custom routes are mounted on `slackAPI.receiver.router` in `routes/index.js` and use `express.json()` for JSON bodies. These endpoints are served on the same port as the Slack app (e.g. `PORT` or 3000).

## POST /send-message

Sends a message to a Slack channel (no thread).

**Request**

- Method: `POST`
- Headers: `Content-Type: application/json`
- Body (JSON):
  - **channel** (required) – Slack channel ID.
  - **message** (required) – Text to post.

**Response**

- `200`: `{ ok: true, message: "Message sent successfully.", ts, channel }`
- `400`: `{ ok: false, error: "Channel and message are required." }`
- `500`: `{ ok: false, error: "<error message>" }` (e.g. Slack API error)

**Usage**

- Any client with network access to the service can use this to post into a channel (e.g. alerts, internal tools). Not used by the bot’s own Slack event handlers.

## POST /send-thread-message

Sends a message into an existing Slack thread. This is the **callback** used by ace-stack-backend after processing an omni/LLM request: the backend calls this URL with the channel, thread `ts`, and the reply text so the bot can post it in the correct thread.

**Request**

- Method: `POST`
- Headers: `Content-Type: application/json`
- Body (JSON):
  - **channelID** (required) – Slack channel ID.
  - **ts** (required) – Thread root message timestamp (`thread_ts`).
  - **message** (required) – Text to post in the thread.

**Response**

- `200`: `{ ok: true, message: "Thread message sent successfully.", ts, channel }`
- `400`: `{ ok: false, error: "Channel, message, and ts are required." }`
- `500`: `{ ok: false, error: "<error message>" }`

**Callback flow**

1. User sends a message in a channel (mention) or thread or DM.
2. ops-bot sends payload to ace-stack-backend `POST /api/omni/ingest` including `callback_url: {SELF_URL}/send-thread-message`.
3. When the backend has the LLM reply, it POSTs to that URL with `channelID`, `ts`, and `message`.
4. ops-bot calls Slack `chat.postMessage` with `thread_ts: ts` and returns 200.

**Note**

- For DMs, the backend may still use this callback; the same endpoint works for any channel and thread. `SELF_URL` must be reachable from the stack-backend (e.g. in-cluster service URL or public URL).
