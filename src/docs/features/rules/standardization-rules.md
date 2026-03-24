# Standardization Rules

Rules that define the **standardizations** (naming, format, structure, and patterns) used across the ACE system. These must be respected so that code, docs, and infra stay consistent. When adding something that already exists elsewhere (e.g. pagination, error handling), **reuse the existing pattern** instead of inventing a new one.

See also: [main rules](./main-rules.md) (respect existing standardizations), [development rules](./development-rules.md) (follow existing patterns when implementing).

---

## 1. Language and content in code

- **Language**: All code, comments, and technical identifiers are in **English**. User-facing text (UI labels, messages to end users) may be in another language (e.g. Portuguese) when required by the product.
- **No emojis in code**: Do not use emojis in source code, comments, or commit messages. Reserve emojis for user-facing content only when the product allows.
- **TypeScript**: Use **TypeScript** where the project already uses it (e.g. frontend, NestJS backend). Do not introduce plain JavaScript in TypeScript codebases without a documented reason.

**Agents**: Generate code and comments in English. Do not add emojis to code or commits. Use TypeScript in TypeScript projects.

---

## 2. Naming conventions

### Code (variables, functions, types, constants)

- **Variables and functions**: **camelCase** (e.g. `userData`, `fetchUserData`, `isLoading`).
- **Classes, types, interfaces**: **PascalCase** (e.g. `UserService`, `UserDto`, `ApiResponse`).
- **Constants** (config, env-derived): **UPPER_SNAKE_CASE** when they are true constants (e.g. `API_BASE_URL`, `MAX_RETRIES`). In many codebases, config keys or env names use this style.
- **Files**: Follow the convention of the repository (e.g. `user.service.ts`, `user.controller.ts` in NestJS; kebab-case or PascalCase for React components as per the project).

### Branches and documentation

- **Branches**: **kebab-case** only (e.g. `feature/user-management`, `bugfix/login-validation`). See [gitflow rules](./gitflow-rules.md).
- **Documentation files**: **kebab-case** (e.g. `api-endpoints.md`, `debugging-guide.md`). See [documentation rules](./documentation-rules.md).

**Agents**: Use camelCase for variables and functions, PascalCase for types and classes. Use kebab-case for branch names and doc filenames. Match existing file-naming in the repo.

---

## 3. APIs and URLs

- **No global /api prefix**: The project does **not** mandate a single global `/api` prefix for all routes. Route paths are defined per controller or module; do not add a project-wide `/api` prefix unless the project explicitly uses it.
- **URLs from environment**: Base URLs and service endpoints must come from **environment variables** (e.g. `VITE_API_URL`, `ACE_DB_GATEWAY_URL`). **Never hardcode** full URLs or hosts in code or config that could differ per environment.
- **REST**: APIs between ACE services are **REST**. Use consistent HTTP methods and status codes; document endpoints in the service `docs/`.

**Agents**: When adding or calling an API, use env vars for base URLs. Do not hardcode URLs. Follow the existing route and controller structure in the service (e.g. NestJS controller paths, backend route patterns).

---

## 4. Commits: Conventional Commits

- **Format**: `<type>(<scope>): <description>`.
- **Types**: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`, etc. Use lowercase.
- **Scope** (optional): Module or area (e.g. `users`, `auth`, `api`).
- **Description**: Short, clear, in English. No emojis.
- **Examples**: `feat(users): add pagination to user list`, `fix(auth): correct JWT validation for expired tokens`.

See [gitflow rules](./gitflow-rules.md) for details.

**Agents**: Use Conventional Commits for every commit. Prefer a clear type and scope; keep the description concise and in English.

---

## 5. Error handling and HTTP responses

- **Check response before reading body**: Before reading the response body (e.g. `.json()` or `.text()`), check `response.ok` (or equivalent) and handle non-2xx statuses. Do not assume success.
- **Do not read the body twice**: Do not call both `.json()` and `.text()` on the same response. Read once and use the result.
- **Propagate meaningful errors**: Surface clear, safe error messages to callers or logs. Do not expose sensitive data in error messages.
- **Structured error handling**: Use try/catch where appropriate; return or throw consistent error shapes so that clients and logs can handle them uniformly.

**Agents**: When calling APIs or reading responses, check `response.ok`, read the body once, and handle errors with clear messages. Follow existing error-handling patterns in the file or service.

---

## 6. Logging and observability

- **Structured logs**: Use structured logging (e.g. JSON or key-value fields) so that log aggregation and search work correctly.
- **Correlation ID**: Include a **correlation ID** (or request ID) in logs when available, so that requests can be traced across services.
- **Log levels**: Use appropriate levels (e.g. debug, info, warn, error). Do not log secrets or sensitive data.

**Agents**: When adding or changing logs, use the existing logger and structure. Include correlation/request ID when the context provides it. Do not log credentials or PII.

---

## 7. Security

- **Input validation**: Validate and sanitize **all** inputs at API boundaries (e.g. DTOs, query params, body). Do not trust client input.
- **JWT authentication**: Protected endpoints must use **JWT authentication** (e.g. via ace-stack-backend and guards). Do not bypass auth on routes that should be protected.
- **Secrets**: Never commit secrets. Use environment variables or a secrets manager (e.g. AWS Secrets Manager for app env vars). See [infrastructure rules](./infrastructure-rules.md).

**Agents**: Add validation to new endpoints; use existing auth guards. Do not hardcode secrets or skip validation.

---

## 8. Documentation structure and placement

- **Central docs**: Under the central docs root (in ace-manual: `src/docs/`) in folders such as `rules/`, `architecture/`, `environments/`, `services/`. Place each doc in the **correct** folder.
- **Service docs**: Each service has a **`docs/`** directory. Use subfolders (e.g. `api/`, `development/`, `architecture/`) as in the project; **kebab-case** for file names.
- **Entry points**: When adding or moving docs, update **index**, **README**, and **START_HERE** at the relevant level. See [main rules](./main-rules.md) and [documentation rules](./documentation-rules.md).

**Agents**: Use kebab-case for doc filenames. Put new docs in the correct folder (central or service). Update index/README/START_HERE when the structure changes.

---

## 9. Infrastructure and resources

- **AWS region**: **us-east-1** for ACE resources.
- **Tags**: Every resource must have **`Project`** = **`ACE`** and **`Environment`** = **`<ENV>-ACE`** (e.g. `dev-ACE`, `prod-ACE`).
- **Naming**: Resource names follow the pattern **`<environment>-ace-*`** or **`<environment>-*`** (e.g. cluster name, queue name). Keep this pattern for new resources.

See [infrastructure rules](./infrastructure-rules.md) for full details.

**Agents**: When defining or documenting infra, use us-east-1, the mandatory tags, and the existing naming pattern. Do not invent new tag keys or naming schemes.

---

## 10. Reuse existing patterns (no invention)

- When you add something that **already exists** in the codebase (pagination, filtering, error handling, API response shape, validation, etc.), **read the existing implementation** and **reuse the same format, rules, and logic**.
- Do not introduce a **new** pattern for the same concept. Consistency reduces bugs and keeps the codebase maintainable.
- If the pattern is in another file or service, copy the approach (naming, structure, status codes, DTOs) rather than inventing a different one.

**Agents**: Before implementing pagination, a new endpoint, error handling, or similar feature, search the repo (and related services if needed) for existing implementations and replicate the same pattern. Do not invent a new pattern when one already exists.

---

## 11. Deprecation and breaking changes

- **Deprecating an API or service**: When an endpoint, feature, or service is deprecated, **document it** in the relevant `docs/` (service docs and, if system-wide, central docs). State what is deprecated, from when, and what to use instead (or when it will be removed). Do not remove without a deprecation period or explicit decision.
- **Breaking changes**: When a change breaks existing behavior or contracts (e.g. API response shape, required headers, removed endpoint), use **`BREAKING CHANGE:`** in the commit body (Conventional Commits) and describe the break and migration in the **PR body** and in **documentation**. Update the affected service docs and any clients or callers documented in the project.
- **Communication**: For deprecations and breaking changes, ensure the PR description and the docs make it clear what changed and how to adapt. If there are known consumers (other services, frontend), mention them in the PR.

**Agents**: When deprecating or introducing a breaking change, add or update the relevant docs, use `BREAKING CHANGE:` in the commit, and describe the impact in the PR. Do not remove or change contracts without documenting the deprecation or break.

---

## Summary for AI agents

| Area | Standardization |
|------|-----------------|
| Language | English for code/comments/identifiers; no emojis in code; TypeScript where used. |
| Naming | camelCase (vars/functions), PascalCase (types/classes), kebab-case (branches, doc files). |
| APIs/URLs | No global /api requirement; URLs from env vars; REST; no hardcoded hosts. |
| Commits | Conventional Commits: `type(scope): description` in English. |
| Error handling | Check response.ok; read body once; meaningful errors; structured handling. |
| Logging | Structured logs; correlation ID; no secrets in logs. |
| Security | Input validation; JWT on protected routes; no secrets in code. |
| Docs | Correct folder; kebab-case filenames; update index/README/START_HERE. |
| Infra | us-east-1; tags Project=ACE, Environment=<ENV>-ACE; naming pattern. |
| Patterns | Reuse existing patterns (pagination, errors, APIs); do not invent new ones. |
| Deprecation / breaking | Document deprecations in docs; use BREAKING CHANGE: in commit and describe in PR and docs. |
