# ACE Service Integrations

This document describes **how ACE services integrate**: base URLs, authentication, API contracts, and patterns for reliability (idempotency, retries). Use it when implementing or changing calls between services.

---

## Base URLs and Environment Variables

- **All service base URLs** must come from **environment variables**. Never hardcode hosts or full URLs in code or config that can differ per environment.
- **Frontend** → Backend: e.g. `VITE_API_URL` or equivalent (see ace-dashboard-frontend).
- **Backend** → DB Gateway: e.g. `ACE_DB_GATEWAY_URL` or `DB_GATEWAY_URL` (see ace-stack-backend).
- **Backend** → other internal services: use env vars per service (e.g. `ACE_COMMANDS_API_URL`, `ACE_CONFIGURATION_URL`).
- **Bots** → Backend or Commands API: same principle; URLs from env.

See [../rules/standardization-rules.md](../rules/standardization-rules.md): “URLs from environment; never hardcode.”

---

## How Backend Calls DB Gateway

- Backend sends HTTP requests to the DB Gateway base URL (from env). Paths and methods are defined by the DB Gateway API (see ace-db-gateway docs).
- **JWT**: Backend must send a valid JWT (e.g. in `Authorization: Bearer <token>`) so that DB Gateway can authorize the request. DB Gateway validates JWT on every request; no JWT or invalid JWT → 401.
- **Contracts**: Request/response shape (body, query params, headers) must match what DB Gateway expects. When changing either side, update both and document in the relevant service docs.

---

## JWT Propagation

- **User context**: When the backend calls DB Gateway on behalf of a user, it typically forwards the same JWT received from the frontend (or a token derived from it, if the design specifies so). Do not drop or replace the user context unless the design explicitly requires a service-to-service token.
- **Service-to-service**: If a service (e.g. a bot or job) calls the backend or another service without a user JWT, the auth mechanism (e.g. service account, API key, or internal JWT) must be documented and implemented consistently. See [security.md](./security.md).

---

## API Naming and Structure

- **No global /api prefix**: The project does not mandate a single `/api` prefix for all routes. Paths are defined per controller or module in each service. See [../rules/standardization-rules.md](../rules/standardization-rules.md).
- **REST**: Inter-service APIs are REST. Use consistent HTTP methods and status codes; document endpoints in the service’s `docs/`.
- **Versioning**: If a service uses versioned paths (e.g. `/v1/...`), callers must use the same version contract. Document in the service that owns the API.

---

## Idempotency and Retries

- **Idempotency**: For operations that must not be applied twice (e.g. creating a resource, sending a notification), design the API to be idempotent (e.g. idempotency key, or PUT with same key). Document in the service that owns the API.
- **Retries**: If a caller retries on failure (e.g. transient network error), use idempotent operations where possible and respect `Retry-After` or backoff. Do not retry indefinitely without limit. Document retry policy in the caller’s docs if it affects behavior.

**Agents**: When adding a new integration, add the env var name and purpose to the service catalog or service docs. Update this file if you introduce a new integration pattern or change how JWT or URLs are used.
