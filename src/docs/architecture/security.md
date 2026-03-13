# ACE Security Architecture

This document describes the **security model** that agents and developers must respect: authentication (JWT), roles/permissions, where secrets live, and input validation. For detailed rules (e.g. how to validate, what not to commit), see [../rules/security-rules.md](../rules/security-rules.md).

---

## Authentication: JWT

- **Issuance**: JWT is issued by **ace-stack-backend** after successful login (credentials validated). The frontend (or client) stores and sends the token on subsequent requests.
- **Validation**: Protected endpoints in **ace-stack-backend** and in **ace-db-gateway** (and any other service that exposes protected APIs) must validate the JWT on every request. Use existing guards and middleware (e.g. `JwtAuthGuard`, token verification); do not bypass auth.
- **Public routes**: Health checks, login, and webhooks or callbacks that use another auth mechanism must be explicitly defined and documented. Everything else is protected by default.

**Agents**: When adding an endpoint, decide if it is public or protected. If protected, use the existing JWT guard or equivalent. Do not add routes that skip authentication unless they are documented as public. See [../rules/security-rules.md](../rules/security-rules.md).

---

## Roles and Permissions

- Where the system has roles or permissions (e.g. super user vs normal user), **enforcement is on the server**. The backend (or the service that owns the resource) must reject unauthorized requests. Do not rely on the frontend to hide or allow actions.
- Use existing guards or middleware (e.g. `SuperUserGuard`) where applicable. Document any new permission rules in the service docs.

**Agents**: When adding functionality that depends on role or permission, enforce it in the backend (or owning service). Do not add endpoints that perform privileged actions without checking authorization.

---

## Secrets: Where They Live and How They Are Injected

- **Never commit** secrets, passwords, API keys, or tokens to the repository. Do not hardcode them in code, config files, or manifests.
- **Runtime secrets for applications**: Use **environment variables** or a secrets manager (e.g. **AWS Secrets Manager**). Path pattern for app env vars: **`ace/<env>/<service>-secrets`**. These are **only for values that become environment variables** for the app at runtime; not for arbitrary config. See [deployment.md](./deployment.md) and [../rules/infrastructure-rules.md](../rules/infrastructure-rules.md).
- **CI/CD secrets** (e.g. deploy credentials, GitHub tokens): **GitHub Environments**. Do not store production secrets in repository variables or in workflow file content.
- **Injection**: Secrets are injected into the app (e.g. via K8s or deployment pipeline) as environment variables. Document required keys and the secret path when adding a new service.

**Agents**: When a feature needs a new secret, document the required key and where it is stored (Secrets Manager path or GitHub Environment). Do not add code that reads secrets from the repo or from hardcoded strings.

---

## Input Validation and Safe Defaults

- **All inputs** at API boundaries must be **validated and sanitized**. Do not trust client input (query params, body, headers, path params). Use DTOs, schemas, or validation middleware as appropriate (e.g. NestJS validation pipes). See [../rules/security-rules.md](../rules/security-rules.md).
- **Safe defaults**: Prefer safe defaults for security-sensitive options (e.g. HTTPS, secure headers). Do not expose internal details in error messages or logs (see below).

---

## What Not to Do

- **No secrets in code or logs**: Do not log tokens, passwords, or API keys. Do not expose stack traces or internal hostnames to the client in production. Return generic or safe messages to the client (e.g. “Invalid request”, “Authentication failed”) and log detailed information server-side only when needed for debugging, without secrets.
- **No bypassing auth**: Do not disable or bypass JWT validation for “temporary” or “internal” endpoints without an explicit, documented exception.
- **No sensitive data in errors**: Avoid leaking whether a user exists, whether a password is wrong, or other information that could help an attacker. See [../rules/security-rules.md](../rules/security-rules.md).

---

## Bots: Rate Limiting

- Bot services (ace-slackbot, ace-sec-bot, ace-ops-bot) must implement **rate limiting** where they call external APIs or process user input, to avoid abuse and to respect third-party limits. Follow the existing rate-limiting approach in the bot codebase; document any new limits in the service docs. See [../rules/security-rules.md](../rules/security-rules.md).

---

## Summary for Agents

| Topic | Rule |
|-------|------|
| Auth | JWT issued by backend; all protected endpoints validate JWT. Use existing guards; document public routes. |
| Permissions | Enforce roles/permissions on the server; use existing guards. |
| Secrets | Never commit or hardcode; use env vars or Secrets Manager (`ace/<env>/<service>-secrets`) for app runtime; CI secrets in GitHub Environments. |
| Input | Validate and sanitize all inputs at API boundaries; use existing validation patterns. |
| Errors/logs | Do not expose sensitive data or internals to the client; no secrets in logs. |
| Bots | Apply rate limiting on external and user-triggered actions. |
