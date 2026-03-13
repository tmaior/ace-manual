# ace-stack-backend

NestJS backend for the ACE system: auth (JWT), business logic, and orchestration.

## Purpose

- Issues and validates JWT; protects endpoints.
- Orchestrates calls to **ace-db-gateway** and optionally **ace-configuration**, **ace-commands-api**.
- Tech stack: NestJS, Node.js, TypeScript. Typical port: 3000.

## Documentation

- **This folder**: Central reference; detailed docs live in the service repo.
- **Repository docs**: In **ace-stack-backend/docs/**.

## Related

- [Architecture service catalog](../../architecture/service-catalog.md)
- [Data flow](../../architecture/data-flow.md), [Security](../../architecture/security.md)
