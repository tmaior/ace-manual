# ace-slackbot

Slack bot service for ACE. Node.js, Slack API, Redis for sessions.

## Purpose

- Handles Slack events and commands.
- Integrates with ace-stack-backend and/or ace-commands-api with appropriate auth.
- Sessions stored in Redis; rate limiting applies.

## Documentation

- **This folder**: Central reference; detailed docs live in the service repo.
- **Repository docs**: In **ace-slackbot/docs/**.

## Related

- [Architecture service catalog](../../architecture/service-catalog.md)
- [Security rules](../../rules/security-rules.md) (rate limiting, auth)
