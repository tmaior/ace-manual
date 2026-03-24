# Application Requirements

This document defines **all rules, standardizations, behaviors, configurations, and procedures** that every ACE application must follow. It applies to **existing applications** and to **any new application** added to the system. Everything that is **common to all apps** and **necessary for an environment to run without problems** is stated or referenced here.

Use this document when:
- **Including a new application** in the ACE system (new microservice or new deployable component).
- **Auditing** whether an application complies with project-wide rules.
- **Defining** what is mandatory for all apps (code, docs, infra, security, deployment).

Details for specific topics (e.g. PR process, infrastructure tags, security) are in the referenced rule files; this document is the **single entry point** for "what every app must do."

---

## 1. Scope and applicability

- **Applies to**: Every application (service) that is part of the ACE system and is deployed or run in any environment (local, dev, staging, demo, production). This includes frontends, backends, gateways, bots, schedulers, and any other runnable component.
- **Does not apply to**: Pure infrastructure (Terraform, K8s cluster config), local-env docker-compose wiring only, or one-off scripts that are not deployed as a service. Deprecated services are excluded from new work but should still meet these requirements if they remain in use.
- **Relationship to other rules**: This file **aggregates and references** the project rules. The [main rules](./main-rules.md), [standardization](./standardization-rules.md), [security](./security-rules.md), [documentation](./documentation-rules.md), [development](./development-rules.md), [infrastructure](./infrastructure-rules.md), [gitflow](./gitflow-rules.md), and [testing](./testing-rules.md) rules remain the source of detail; applications must comply with all of them as summarized below.

---

## 2. Repository and documentation (mandatory for every app)

### 2.1 Service docs/ directory

- Every application **must** have a **`docs/`** directory at the repository root (or at the service root if the repo contains multiple services).
- **Contents**: Description of the application, setup (local and deployed), behavior, API documentation (endpoints, request/response, auth), and any repo-specific conventions. Use subfolders (e.g. `api/`, `development/`, `architecture/`) and **kebab-case** file names.
- **Entry points**: Each `docs/` must have an **index**, **README**, and **START_HERE** (or equivalent) kept in sync when docs are added, removed, or reorganized. See [main-rules](./main-rules.md) and [documentation-rules](./documentation-rules.md).

### 2.2 Document every change

- **Every change** that affects behavior, APIs, or usage **must** be reflected in documentation in the **same** PR or change set. No exceptions for "small" or "internal" changes.
- **Types of docs**: Setup guides, architecture, API docs, debugging/troubleshooting, development guides. Keep examples **functional and tested**. See [documentation-rules](./documentation-rules.md).

### 2.3 Central documentation updates

- When an application is **new** or when its **contracts or behavior** affect other services or the system, update **central docs**: [service-catalog](../architecture/service-catalog.md), [data-flow](../architecture/data-flow.md), [integrations](../architecture/integrations.md), [deployment](../architecture/deployment.md), and [env-vars-and-secrets](../environments/env-vars-and-secrets.md) as applicable. Update architecture index/README/START_HERE when adding or changing services.

---

## 3. Code and content rules

### 3.1 Language

- **Code and comments**: **English** only. Naming (variables, functions, types, constants, file names when descriptive) in English.
- **User-facing text**: May be in another language (e.g. Portuguese) when required by the product.
- **No emojis** in code, comments, or commit messages. See [main-rules](./main-rules.md) and [standardization-rules](./standardization-rules.md).

### 3.2 Technology

- **TypeScript**: Use TypeScript where the project already uses it (e.g. NestJS backend, React/Vite frontend). Do not introduce plain JavaScript in TypeScript codebases without a documented reason.

### 3.3 Naming conventions

- **Variables and functions**: **camelCase** (e.g. `userData`, `fetchUserData`).
- **Classes, types, interfaces**: **PascalCase** (e.g. `UserService`, `ApiResponse`).
- **Constants** (config, env-derived): **UPPER_SNAKE_CASE** (e.g. `API_BASE_URL`, `MAX_RETRIES`).
- **Branches**: **kebab-case** only (e.g. `feature/user-management`, `bugfix/login-validation`). See [gitflow-rules](./gitflow-rules.md).
- **Documentation files**: **kebab-case** (e.g. `api-endpoints.md`, `debugging-guide.md`).

### 3.4 Commits

- **Conventional Commits**: Format `<type>(<scope>): <description>`. Types: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`. Description in English, no emojis. See [standardization-rules](./standardization-rules.md) and [gitflow-rules](./gitflow-rules.md).

---

## 4. APIs and configuration

### 4.1 URLs and base URLs

- **All base URLs and service endpoints** must come from **environment variables**. **Never hardcode** full URLs or hosts in code or config that can differ per environment (e.g. `VITE_API_URL`, `DB_GATEWAY_URL`, `REDIS_HOST`). See [standardization-rules](./standardization-rules.md) and [integrations](../architecture/integrations.md).
- **No global /api prefix**: The project does not mandate a single `/api` prefix for all routes. Paths are defined per controller or module in each service.
- **REST**: Inter-service APIs are REST. Use consistent HTTP methods and status codes. Document endpoints in the service `docs/`.

### 4.2 API contracts and integration

- **Request/response**: Document and keep request/response shapes (body, query, headers) consistent between caller and callee. When changing either side, update both and the docs.
- **JWT propagation**: When a service calls another ACE service on behalf of a user, forward or derive the JWT as defined (e.g. `Authorization: Bearer <token>`). Do not drop user context unless the design explicitly requires a service-to-service token. See [integrations](../architecture/integrations.md) and [security-rules](./security-rules.md).
- **Idempotency and retries**: For operations that must not be applied twice, design for idempotency (e.g. idempotency key, PUT with same key). If callers retry, respect backoff and do not retry indefinitely. Document in the service that owns the API.

---

## 5. Security (mandatory for every app)

### 5.1 Input validation

- **All inputs** at API boundaries (query, body, path, headers) must be **validated and sanitized**. Do not trust client input. Use DTOs, schemas, or validation middleware (e.g. NestJS validation pipes, class-validator). Reject invalid input with a **safe, clear** error message. See [security-rules](./security-rules.md).

### 5.2 Authentication and authorization

- **Protected endpoints** must require **JWT authentication** (or the auth mechanism defined for that service). Use existing guards/middleware; do not bypass auth for convenience. **Public routes** (e.g. health, login, webhooks with other auth) must be **explicitly defined and documented**; everything else is protected by default.
- **Permissions**: Enforce roles/permissions on the **server**. Do not rely on the frontend to hide or allow actions; the backend must reject unauthorized requests. Use existing guards (e.g. `SuperUserGuard`) where applicable. See [security-rules](./security-rules.md).

### 5.3 Secrets and credentials

- **Never commit** secrets, passwords, API keys, or tokens. Do not hardcode them in code, config files, or manifests.
- **Runtime secrets**: Use **environment variables** or a secrets manager. In EKS, path pattern **`ace/<env>/<service>-secrets`** in AWS Secrets Manager; these secrets are **only for environment variables** the application uses at runtime. See [infrastructure-rules](./infrastructure-rules.md) and [env-vars-and-secrets](../environments/env-vars-and-secrets.md).
- **CI/CD secrets**: Stored in **GitHub Environments**, not in repository variables or workflow file content. See [infrastructure-rules](./infrastructure-rules.md).

### 5.4 Error messages and sensitive data

- **Do not expose** sensitive data in error messages, logs, or API responses (no stack traces in production, no DB details, internal hostnames, or tokens). Return generic or safe messages to the client; log details server-side only when needed for debugging, without secrets. See [security-rules](./security-rules.md).

### 5.5 Rate limiting (bots and external calls)

- **Bot services** (and any app that calls external APIs or processes user-triggered input at scale) must implement **rate limiting** to avoid abuse and respect third-party limits. Document limits and configuration in the service docs. See [security-rules](./security-rules.md).

---

## 6. Error handling and logging

### 6.1 HTTP and client responses

- **Check response before reading body**: Before reading the response body (e.g. `.json()` or `.text()`), check `response.ok` (or equivalent) and handle non-2xx statuses. Do not assume success.
- **Do not read the body twice**: Do not call both `.json()` and `.text()` on the same response. Read once and use the result.
- **Propagate meaningful errors**: Surface clear, safe error messages to callers or logs. Use try/catch and consistent error shapes so clients and logs can handle them uniformly. See [standardization-rules](./standardization-rules.md).

### 6.2 Logging and observability

- **Structured logs**: Use structured logging (e.g. JSON or key-value fields) for log aggregation and search.
- **Correlation ID**: Include a **correlation ID** (or request ID) in logs when available so requests can be traced across services.
- **Log levels**: Use appropriate levels (debug, info, warn, error). **Do not log** secrets or sensitive data. See [standardization-rules](./standardization-rules.md).

---

## 7. Infrastructure and deployment (every deployable app)

### 7.1 Region and tags

- **AWS region**: All ACE infrastructure is in **us-east-1**. Applications run in EKS in this region.
- **Resource tags**: Every resource that belongs to ACE must have **`Project`** = **`ACE`** and **`Environment`** = **`<ENV>-ACE`** (e.g. `dev-ACE`, `prod-ACE`). See [infrastructure-rules](./infrastructure-rules.md).

### 7.2 ace-infra: folder and manifests

- **One folder per deployed service** under **ace-infra** (e.g. `ace-stack-backend/`, `ace-db-gateway/`). Each folder contains:
  - **Dockerfile(s)** for building the application image.
  - **Kubernetes manifests**: Deployment, Service, Ingress, ConfigMap (as needed). Use naming such as `*-deployment.yaml`, `*-service.yaml`, `*-ingress.yaml`.
  - **Namespace**: Set the correct `namespace` in metadata per environment (e.g. `dev`, `stg`, `prod`). Do not deploy to `default` or ad-hoc namespaces unless documented.
- **Ingress/ALB**: Use the existing ALB Ingress Controller and group names for consistent routing and TLS. Hostnames follow the convention per environment (e.g. `dev-*`, `stg-*`, or as in [creating-a-new-environment](../environments/creating-a-new-environment.md)).
- **Scripts**: DB setup, queue creation, or other one-off scripts may live in **scripts/** under ace-infra or under the service folder. Document in ace-infra `docs/`. See [infrastructure-rules](./infrastructure-rules.md) and [deployment](../architecture/deployment.md).

### 7.3 Secrets and environment variables

- **Path pattern**: **`ace/<env>/<service>-secrets`** in AWS Secrets Manager (e.g. `ace/dev/ace-stack-backend-secrets`). Contents are **only** environment variables for the application at runtime.
- **Document required keys**: In the service `docs/` (and in [env-vars-and-secrets](../environments/env-vars-and-secrets.md) for shared or critical vars), document every required env var and secret key. When adding a new service, add a row to the env-vars table and define the Secrets Manager path in the service README or ace-infra. See [env-vars-and-secrets](../environments/env-vars-and-secrets.md).

### 7.4 Health and readiness

- **Health endpoint**: Every application that is deployed (and that other services or the platform depend on) **should** expose a **health** (or readiness) endpoint (e.g. `/health`) that returns a success status when the app is running and, if applicable, when dependencies (DB, Redis) are reachable. This is used by K8s probes, load balancers, and runbooks. Document the path and expected response in the service docs.
- **Docker**: Application must be buildable and runnable via the Dockerfile in ace-infra. Do not rely on host-only tools or unstated dependencies.

### 7.5 CI/CD

- **Pipeline**: GitHub Actions. Typical flow: build on push/tag, run **lint and type check**, push images to **ECR** (us-east-1), deploy to EKS using manifests from ace-infra. See [infrastructure-rules](./infrastructure-rules.md) and [deployment](../architecture/deployment.md).
- **Environments**: Use **GitHub Environments** (e.g. `development`, `staging`, `production`) for deploy targets and secrets. **Production** (and optionally staging) deploys **must** use **approvals** (Environment protection rules). Do not bypass approval for production.
- **Branch strategy**: Follow [gitflow-rules](./gitflow-rules.md) and repo-specific flow (e.g. deploy to dev from `development`, to prod from the branch defined for production). Document in the service or in [development](../environments/development.md), [staging](../environments/staging.md), [production](../environments/production.md).

---

## 8. Development and testing procedures

### 8.1 Branching and local testing

- **Feature/bugfix branches**: Create from the **latest** `development` branch (or the branch defined by the repo). Use **kebab-case** branch names. Do not implement on long-lived branches directly. See [development-rules](./development-rules.md) and [gitflow-rules](./gitflow-rules.md).
- **Local testing before commit/push**: After development, **test locally** (e.g. docker-compose or per-service runbooks) using **feature-branch code for changed services** and **development-branch code for unchanged services**. Ensure services **communicate correctly**. Only after integration passes, commit and push. See [development-rules](./development-rules.md).

### 8.2 Testing (behavior-based, no committed test suites)

- **No unit tests or test code** are required or committed. **Lint and type check** are required where the project uses them.
- **Simple tests** before PR: Call URLs (including health), open UIs, submit forms, check DB or API results where relevant. For cross-service flows, run end-to-end (e.g. frontend → backend → db-gateway). Document in the **PR body (Testing section)** the steps to reproduce, what was checked, and expected results. See [testing-rules](./testing-rules.md) and [pr-rules](./pr-rules.md).

### 8.3 Reuse existing patterns

- **Do not invent** new patterns for something that already exists in the codebase (pagination, error handling, API shape, validation). **Read the existing implementation** and **reuse the same format, rules, and logic**. If the pattern is in another service, copy the approach. See [standardization-rules](./standardization-rules.md) and [development-rules](./development-rules.md).
- **Nothing must be invented**: If something is unclear, read the docs and code first; only then ask the user. Do not guess behavior or conventions. See [main-rules](./main-rules.md).

---

## 9. Cross-service and environment behavior

### 9.1 Dependencies and env vars

- **Document dependencies**: In the service `docs/` and in [service-catalog](../architecture/service-catalog.md), document which other ACE services (or external systems) the application calls. When changing an API or contract, update the caller, the callee, and the docs.
- **New integration**: When adding a new integration, add the env var name and purpose to the service docs and, if shared, to [env-vars-and-secrets](../environments/env-vars-and-secrets.md) and [integrations](../architecture/integrations.md).

### 9.2 Local and EKS parity

- **Local**: Application must be runnable locally (e.g. via docker-compose in local-env or via dev server) with configuration from env vars (e.g. `.env`). Document required env vars and startup order in the service docs and in [local](../environments/local.md) if it is part of the standard stack.
- **EKS**: Same application runs in EKS with env vars from Secrets Manager (`ace/<env>/<service>-secrets`). No hardcoded URLs or env-specific logic that cannot be configured via env.

---

## 10. Summary: what every application must have

| Area | Requirement |
|------|-------------|
| **Docs** | `docs/` directory with description, setup, API, and entry points (index/README/START_HERE); document every change; update central docs when contracts or behavior affect other services. |
| **Language** | Code and comments in English; TypeScript where the project uses it; no emojis in code/commits. |
| **Naming** | camelCase/PascalCase/UPPER_SNAKE as above; kebab-case for branches and doc files; Conventional Commits. |
| **URLs/config** | All base URLs and endpoints from environment variables; never hardcode; REST; document endpoints. |
| **Security** | Validate all inputs; JWT on protected endpoints; no secrets in repo; safe error messages; enforce permissions on server; rate limiting for bots/external calls. |
| **Errors/logging** | Check response before reading body; do not read body twice; structured logs; correlation ID; no secrets in logs. |
| **Infra** | ace-infra folder (Dockerfile + K8s manifests); correct namespace; Secrets Manager path `ace/<env>/<service>-secrets`; document required keys; health endpoint; tags Project=ACE, Environment=<ENV>-ACE. |
| **CI/CD** | GitHub Actions; ECR us-east-1; deploy from ace-infra; GitHub Environments for secrets; approvals for production. |
| **Development** | Feature branches from development; local testing before push; simple behavior tests; PR Testing section; reuse existing patterns; do not invent. |
| **Cross-service** | Document dependencies and env vars; update service-catalog, integrations, env-vars-and-secrets when adding or changing integrations. |

---

## Links

- **Main rules** — [main-rules.md](./main-rules.md)
- **Standardization** — [standardization-rules.md](./standardization-rules.md)
- **Security** — [security-rules.md](./security-rules.md)
- **Documentation** — [documentation-rules.md](./documentation-rules.md)
- **Development** — [development-rules.md](./development-rules.md)
- **Infrastructure** — [infrastructure-rules.md](./infrastructure-rules.md)
- **Gitflow** — [gitflow-rules.md](./gitflow-rules.md)
- **Testing** — [testing-rules.md](./testing-rules.md)
- **PR** — [pr-rules.md](./pr-rules.md)
- **Architecture** — [../architecture/](../architecture/) (service-catalog, deployment, integrations, data-flow)
- **Environments** — [../environments/](../environments/) (local, dev, stg, prod, env-vars-and-secrets, creating-a-new-environment)
