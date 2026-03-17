# Build, deploy and CI/CD

## Build

### Prerequisites

- Node.js (version per repo and Dockerfile in ace-infra; e.g. Node 18+)
- Yarn
- Docker (for image build in CI or local)

### Local build and run

```bash
# Install dependencies
yarn

# Development server (port 8080)
yarn dev

# Production build
yarn build

# Build in development mode (e.g. for dev deploy)
yarn build:dev

# Preview production build locally (port 4173)
yarn preview
```

### Lint and tests

```bash
yarn lint
yarn test          # Playwright E2E
yarn test:ui       # Playwright with UI
yarn test:headed   # Playwright headed browser
yarn test:user-management  # User management E2E spec
```

### Docker image (ace-infra)

- **Dockerfile**: In **ace-infra** at `ace-web-frontend/ace.web-frontend.Dockerfile` (workflows use `infra-repo/ace-web-frontend/ace.web-frontend.Dockerfile`).
- **Build context**: Application repo root; Dockerfile path from infra checkout.
- **Image**: Pushed to ECR, e.g. `975635808270.dkr.ecr.us-east-1.amazonaws.com/ace/web-frontend:<tag>`.

Build example (with infra repo checked out as `infra-repo`):

```bash
# After creating .env or .env.development with VITE_* vars (or from secrets)
docker build -t ace/web-frontend:local -f infra-repo/ace-web-frontend/ace.web-frontend.Dockerfile .
```

## Deploy (Kubernetes)

- **Manifests**: In **ace-infra** under `ace-web-frontend/`: Deployment, Service, Ingress, and optional ConfigMap/Secret for env.
- **Namespace**: **dev** (development), **demo** (demo), **stg** (staging), **prod** (production); set by CI.
- **Secrets**: From AWS Secrets Manager **ace/&lt;env&gt;/web-frontend-secrets**; workflow generates Kubernetes Secret and applies it; build-time VITE_* are baked into the image, not runtime env in the pod (static SPA).

## CI/CD (GitHub Actions)

### Development

- **Workflow**: `.github/workflows/eks-deploy.dev.yaml`
- **Trigger**: Push to `development`, `feature/*`, `hotfix/*`, `bugfix/*`
- **Steps**: Checkout app and infra; assume IAM role (ace-dev-eks-role); login to ECR; fetch secrets from `ace/dev/web-frontend-secrets`; create `.env.development` from secrets; build Docker image with infra Dockerfile; tag `dev-<timestamp>`; push to ECR; create K8s Secret from secrets; update image in Deployment YAML and namespace to **dev**; set Ingress host to `dev-dashboard.ace.ezops.cloud`; apply manifests.
- **Cluster**: **development-ace-eks**
- **APP_NAME**: `web-frontend`

### Staging

- **Workflow**: `.github/workflows/eks-deploy.stg.yaml`
- **Trigger**: Push to staging branch (see workflow for exact branches).
- **Pattern**: Same as dev; namespace **stg**; Ingress host for staging dashboard.

### Production

- **Workflow**: `.github/workflows/eks-deploy.prod.yaml`
- **Trigger**: Push to `main` (or branch used for production).
- **Pattern**: IAM role **ace-prod-eks-role**; secrets from `ace/prod/web-frontend-secrets`; namespace **prod**; Ingress host for production dashboard; apply to production cluster.

### Demo

- **Workflow**: `.github/workflows/eks-deploy.demo.yaml`
- **Trigger**: Push to `demo`, `demo/*` (see workflow).
- **Pattern**: Namespace **demo**; secrets from `ace/demo/web-frontend-secrets`; Ingress host `demo-dashboard.ace.ezops.cloud` (or as configured).

### Notes

- Dockerfile lives in **ace-infra**; workflows depend on `INFRA_REPO_TOKEN` and infra checkout.
- Required GitHub secrets: `INFRA_REPO_TOKEN`, `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY` (for assume-role and ECR).
- Frontend is a static SPA; env vars are embedded at build time. Changing backend URLs requires a new build and deploy.
