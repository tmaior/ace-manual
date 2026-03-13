# ace-db-gateway

Centralized database gateway for ACE. Validates JWT and executes queries against PostgreSQL and other configured databases.

## Purpose

- Single point of DB access for the backend and authorized callers.
- Validates JWT on every request; no direct DB access from apps.
- Tech stack: Node.js, TypeScript.

## Documentation

- **This folder**: Central reference; detailed docs live in the service repo.
- **Repository docs**: In **ace-db-gateway/docs/**.

## Related

- [Architecture service catalog](../../architecture/service-catalog.md)
- [Data flow](../../architecture/data-flow.md), [Integrations](../../architecture/integrations.md)
