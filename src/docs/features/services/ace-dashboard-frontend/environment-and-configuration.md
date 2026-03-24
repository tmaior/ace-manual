# Environment and configuration

## Required / common environment variables

| Variable | Purpose |
|----------|---------|
| **VITE_API_URL** | Base URL for admin and user APIs (useApi / axios). Must point to ace-stack-backend. Required for login, admin panel, user/client/project management, and other useApi-based features. |
| **VITE_BACKEND_URL** | Base URL for DashboardBackend and backend-only endpoints (secrets, documentations, chat, integrations). Must point to ace-stack-backend. Required for those features. |
| **VITE_FRONTEND_URL** | Public URL of the frontend (e.g. for redirects after login, provider callback, reset password). Required for login success, provider callback, and profile settings redirects. |

These are **build-time** variables: Vite embeds them at `yarn build`. For local dev, set them in `.env` or `.env.local` (e.g. `VITE_API_URL=http://localhost:8080`, `VITE_BACKEND_URL=http://localhost:341`, `VITE_FRONTEND_URL=http://localhost:8080`). For deployed builds, inject them in the CI/CD pipeline (e.g. from AWS Secrets Manager) into the build step so the resulting assets contain the correct URLs.

## Optional environment variables

| Variable | Purpose | Default / behaviour |
|----------|---------|---------------------|
| **VITE_LLM_URL** | Base URL for LLM-related API calls. | Unset; required only if LLM features are used; app throws if used and unset. |
| **VITE_AWS_REGION** | AWS region for ACE AWS Knowledge Base. | Fallback default in code (e.g. `us-east-1`) when unset. |

## Environment-specific behavior

- **Local**: Typically `VITE_API_URL` and `VITE_BACKEND_URL` point to local backend (e.g. different ports). `VITE_FRONTEND_URL` is the local dev URL (e.g. `http://localhost:8080`). Dev server runs on port 8080; see `vite.config.ts`.
- **Dev / Demo / Staging / Production**: URLs point to the corresponding backend and frontend hosts (e.g. dev-dashboard.ace.ezops.cloud, backend on same or different host). Values are usually taken from AWS Secrets Manager (e.g. `ace/dev/web-frontend-secrets`) and written into `.env.development` (or equivalent) before `docker build` in CI/CD.

## Paths and files

- **Vite config**: `vite.config.ts` — dev server port 8080, `allowedHosts` for local and ACE domains, path alias `@/` → `src/`. No API URL is hardcoded here; only `process.env.VITE_FRONTEND_URL` is used for allowedHosts during build if set.
- **Feature flags**: `src/lib/features.ts` — e.g. `FEATURE_MONITORED_RESOURCES_ENABLED`; change and rebuild to enable/disable features.

## Secrets (EKS / CI)

- Secrets (including VITE_* for build) are stored in AWS Secrets Manager at **ace/&lt;env&gt;/web-frontend-secrets** and injected into the build (e.g. via `.env.development` or `.env`) in the GitHub Actions workflow before `docker build`. Never commit secrets; use env-specific secret names per environment.
