# Build, deploy and CI/CD

## Build

### Prerequisites

- Node.js (version per repo; e.g. 18+)
- Yarn
- PostgreSQL (for TypeORM and migrations when running locally)
- Redis (for WebSocket adapter and OAuth state)

### Local build and run

```bash
# Install dependencies
yarn

# Development (watch mode; loads .env.development)
yarn start:dev

# Production (loads env from process; use AWS secrets or env in non-dev)
yarn start

# Staging
yarn start:stg

# Tests
yarn test
```

### TypeORM migrations

Migrations use `src/shared/database/datasource.ts` and load env from `.env.<NODE_ENV>` when run via CLI.

```bash
# Generate migration (development DB)
yarn migration:generate

# Run migrations
yarn migration:run           # development
yarn migration:run:prod      # production
yarn migration:run:staging   # staging
yarn migration:run:test      # test

# Revert
yarn migration:revert
yarn migration:revert:prod
# etc.
```

### Lint and format

```bash
yarn lint
yarn format
```

## Docker and Kubernetes

- **Dockerfile**: Typically in **ace-infra** (e.g. under a backend or web-backend path); build context is the application repo root. Image is pushed to ECR.
- **Deployment**: Backend runs as a long-running Deployment (unlike ace-configuration, which runs as a Job). Manifests (Deployment, Service, Ingress, optional ConfigMap/Secret) are in ace-infra; namespace and cluster depend on environment (dev, demo, prod).
- **Secrets**: In non-dev, `AWS_SECRET_NAME` and `AWS_REGION` are set so the app fetches secrets from AWS Secrets Manager at startup; alternatively, env can be injected via Kubernetes Secret or deployment env.

## CI/CD (GitHub Actions)

- **Workflows**: In the **ace-stack-backend** repository (e.g. `.github/workflows/`) and/or in **ace-infra** for image build and EKS apply. Branch flow follows ACE rules: feature → development → main; PRs to production from development.
- **Typical steps**: Checkout app (and infra if Dockerfile is there); install dependencies; lint/test; build Nest app; build Docker image; push to ECR; assume IAM role for EKS; apply Kubernetes manifests with correct namespace and image tag; secrets from AWS Secrets Manager or GitHub Environments.
- **Environments**: dev, demo, staging, production; each may have its own workflow or matrix. See repository workflows and ace-infra docs for exact job names and triggers.

## Notes

- No global API prefix is set in code; all routes are defined per controller (e.g. `auth`, `api/admin`, `api/users`).
- WebSocket adapter requires Redis at startup; if Redis is unavailable, the app may fail to start or WebSockets will not work across instances.
- For local development, ensure `.env.development` exists with at least PORT, JWT_SECRET, ACE_GATEWAY_URL, REDIS_URL, DB_*, and FRONTEND_HOST/API_HOST if testing OAuth.
