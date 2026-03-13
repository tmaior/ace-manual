# Security Rules

Rules for security across the ACE system: authentication, validation, secrets, error handling, and bot protection. These rules apply to all services and must be followed when adding or changing code, APIs, or configuration.

See also: [standardization-rules](./standardization-rules.md) (validation, JWT, secrets summary), [infrastructure-rules](./infrastructure-rules.md) (secrets storage, GitHub Environments).

---

## 1. Input validation

- **All inputs** at API boundaries must be **validated and sanitized**. Do not trust client input (query params, body, headers, path params).
- Use DTOs, schemas, or validation middleware as appropriate for the stack (e.g. NestJS validation pipes, class-validator). Reuse the existing validation pattern in the service.
- Validate type, format, length, and allowed values. Reject invalid input with a clear, safe error message (see section 4).

**Agents**: When adding or changing endpoints, add or update validation for all inputs. Follow the same validation approach used elsewhere in the same service (e.g. DTOs with decorators). Do not skip validation for "internal" or "admin" endpoints without an explicit, documented exception.

---

## 2. JWT authentication

- **Protected endpoints** must require **JWT authentication**. Authentication is centralized via ace-stack-backend; other services that expose APIs (e.g. ace-db-gateway) must validate JWT on every request unless the route is explicitly public.
- Use the existing guards and middleware (e.g. `JwtAuthGuard`, token verification). Do not bypass auth for convenience or for "temporary" endpoints.
- Public routes (e.g. health checks, login, webhooks that use other auth mechanisms) must be explicitly defined and documented; everything else is protected by default.

**Agents**: When adding a new endpoint, determine if it should be public or protected. If protected, use the existing JWT guard or equivalent. Do not add routes that skip authentication unless they are documented as public.

---

## 3. Secrets and credentials

- **Never commit** secrets, passwords, API keys, or tokens to the repository. Do not hardcode them in code, config files, or manifests.
- **Runtime secrets** for applications: use **environment variables** or a secrets manager (e.g. AWS Secrets Manager). Path pattern for app env vars: `ace/<env>/<service>-secrets`. See [infrastructure-rules](./infrastructure-rules.md).
- **CI/CD secrets** (e.g. deploy credentials, GitHub tokens): use **GitHub Environments** and reference them in workflows. Do not store production secrets in repository variables or in workflow file content.

**Agents**: When a feature needs a new secret or credential, document the required key and where it is stored (Secrets Manager path or GitHub Environment). Do not add code that reads secrets from the repo or from hardcoded strings.

---

## 4. Error messages and sensitive data

- **Do not expose sensitive data** in error messages, logs, or API responses. This includes stack traces in production, database details, internal hostnames, or tokens.
- Return **generic or safe messages** to the client (e.g. "Invalid request", "Authentication failed") while logging detailed information server-side only when needed for debugging (and without secrets).
- When handling errors, avoid leaking whether a user exists, whether a password is wrong, or other information that could help an attacker.

**Agents**: When adding or changing error handling, ensure that responses to the client do not contain secrets, stack traces, or internal details. Use the existing error-handling pattern in the service.

---

## 5. Rate limiting (bots)

- **Bot services** (e.g. ace-slackbot, ace-sec-bot, ace-ops-bot) must implement **rate limiting** where they call external APIs or process user input, to avoid abuse and to respect third-party limits.
- Follow the existing rate-limiting approach in the bot codebase. Document any new limits or configuration in the service docs.

**Agents**: When adding or changing bot behavior that calls external APIs or processes user commands, check if rate limiting is already in place and align with it. Do not add high-frequency or unbounded external calls without rate limiting.

---

## 6. Permissions and authorization

- Where the system has roles or permissions (e.g. super user vs common user), **enforce them** on the server. Do not rely on the frontend or client to hide or allow actions; the backend must reject unauthorized requests.
- Use existing guards or middleware (e.g. `SuperUserGuard`) where applicable. Document any new permission rules in the service docs.

**Agents**: When adding functionality that depends on role or permission, enforce it in the backend (or in the service that owns the resource). Do not add endpoints that perform privileged actions without checking authorization.

---

## Summary for AI agents

| Topic | Rule |
|-------|------|
| Input | Validate and sanitize all inputs at API boundaries; use existing DTOs/validation. |
| Auth | Protected endpoints require JWT; use existing guards; document public routes. |
| Secrets | Never commit or hardcode; use env vars or Secrets Manager; CI secrets in GitHub Environments. |
| Errors | Do not expose sensitive data or internals in client-facing messages or logs. |
| Bots | Apply rate limiting on external calls and user-triggered actions. |
| Permissions | Enforce roles/permissions on the server; use existing guards. |
