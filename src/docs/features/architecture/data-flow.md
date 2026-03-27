# ACE Data Flow

This document describes **how data and requests move** through the ACE system: authentication flow, main request flows, and error/timeout handling. Use it to understand where to add or change logic and how components depend on each other.

---

## Authentication Flow

1. **Login**: User submits credentials to the **frontend**, which sends them to **ace-stack-backend** (e.g. `/auth/login` or equivalent).
2. **Backend** validates credentials (delegating to **ace-db-gateway** for local/email login and identity checks as implemented) and issues a **JWT** to the client.
3. **Frontend** stores the JWT (e.g. in memory or secure storage) and sends it on subsequent requests (e.g. `Authorization: Bearer <token>`).
4. **Backend** and other protected services (e.g. **ace-db-gateway**) validate the JWT on each request. Invalid or expired tokens result in 401; missing token on a protected route also results in 401.

**Agents**: When adding or changing login or auth, use the existing auth module and guards. Do not bypass JWT validation on protected endpoints. See [security.md](./security.md) and [../rules/security-rules.md](../rules/security-rules.md).

---

## Main Request Flows

### User → Frontend → Backend → DB Gateway → Database

1. User acts in the **dashboard** (ace-dashboard-frontend).
2. Frontend calls **ace-stack-backend** with JWT (base URL from env, e.g. `VITE_API_URL`).
3. Backend validates JWT and, if it needs data, calls **ace-db-gateway** with the same (or a propagated) JWT.
4. DB Gateway validates JWT and executes the query against the appropriate database.
5. Response flows back: DB → DB Gateway → Backend → Frontend → User.

All service-to-service URLs must come from **environment variables**; never hardcode hosts or base URLs. See [integrations.md](./integrations.md).

### Backend → Other Internal Services

- Backend uses **ace-db-gateway** for persisted entities (users, projects, configurations, knowledge base metadata, etc.). It does **not** call **ace-configuration** over HTTP for that data—**ace-configuration** owns migrations/models used across services.
- Backend may call **ace-commands-api** or other internal HTTP APIs when needed. Same rules: env-based URLs, JWT or service auth as defined per service.
- Document any new internal call in the service’s docs and in [integrations.md](./integrations.md).

### Jira → ACE (issue and comment events)

1. **Jira** invokes **ace-jira-integration** webhook routes (Atlassian Connect JWT).
2. The app resolves **ACE project ↔ Jira project** via **ace-db-gateway** (`GET /api/project-jira-links`).
3. When linked, it **POST**s the payload to **ace-ops-scheduler** (`/api/payloads`) for downstream processing (omnichannel / LLM pipeline).

### Bots (Slack)

- **ace-slackbot**, **ace-sec-bot**, **ace-ops-bot** receive events from Slack and may call **ace-stack-backend** or **ace-commands-api** with appropriate auth (e.g. JWT or service token as per design).
- Sessions are typically stored in **Redis**. Rate limiting applies to external and user-triggered actions; see [../rules/security-rules.md](../rules/security-rules.md).

---

## Sequence Overview (High-Level)

See [diagrams.md](./diagrams.md) for Mermaid sequence diagrams. In text:

- **Login**: User → Frontend → Backend → (validate) → JWT → Frontend.
- **Authenticated request**: User → Frontend (JWT) → Backend (validate JWT) → DB Gateway (validate JWT) → DB → response back along the chain.
- **Bot command**: Slack → Bot → Backend or Commands API (auth) → response → Bot → Slack.
- **Jira event (linked project)**: Jira → ace-jira-integration → DB Gateway (link lookup) → ace-ops-scheduler → (pipeline).

---

## Error and Timeout Handling

- **4xx/5xx**: Services return appropriate HTTP status codes. Clients (frontend, other services) must handle errors and not assume 2xx. Do not expose internal details or stack traces to the client; see [security.md](./security.md).
- **Timeouts**: Service-to-service calls should use sensible timeouts (e.g. HTTP client timeout). Document timeout behavior in the calling service’s docs if it affects UX or retries.
- **Structured logs**: Use correlation IDs or request IDs across the chain so that errors can be traced from frontend to backend to db-gateway. See project standardization and logging rules.

**Agents**: When adding or changing a call between services, ensure the caller handles non-2xx responses and timeouts. Follow existing error-handling patterns in the service. Update this doc or [integrations.md](./integrations.md) if you introduce a new flow.
