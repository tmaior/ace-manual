# Build, deploy and CI/CD

## Build

### Prerequisites

- Node.js (version per repo and Dockerfile; e.g. 16+)
- Yarn
- PostgreSQL (for local migrations/seed)
- Docker (for image build)

### Local build and run

```bash
# Install dependencies
yarn

# Create DB, run migrations, run seeders, then start server (dev)
yarn dev

# Or only schema/seed (no server) – same as EKS Job
yarn start:configuration

# Local with nodemon
yarn local

# Production-style run
yarn start
# or
yarn prod
```

### Docker image (ace-infra)

- **Dockerfile**: In **ace-infra** at `ace-configuration/ace.configuration.Dockerfile` (workflows use `infra-repo/ace-${{ env.APP_NAME }}/ace.${{ env.APP_NAME }}.Dockerfile`).
- **Build context**: Application repo root; Dockerfile path from infra checkout.
- **Image**: Pushed to ECR, e.g. `975635808270.dkr.ecr.us-east-1.amazonaws.com/ace/configuration:<tag>`.

Build example (with infra repo checked out as `infra-repo`):

```bash
docker build -f infra-repo/ace-configuration/ace.configuration.Dockerfile -t ace/configuration:local .
```

## Deploy (Kubernetes)

- **Manifests**: In **ace-infra** under `ace-configuration/`: Job (and any Service/Ingress if the server is deployed).
- **Namespace**: **dev** (development), **demo** (demo), **prod** (production); set by CI.
- **Job**: The deployment applies a **Job** (`ace-configuration-job`) that runs the configuration image to execute `db:create`, `db:migrate`, `db:seed` (e.g. `yarn start:configuration`). There is no `ace.configuration.deployment.yaml` in the apply step (workflow removes it). So in EKS, ace-configuration typically runs as a one-off Job, not a long-running Deployment.
- **Secrets**: From AWS Secrets Manager **ace/&lt;env&gt;/configuration-secrets**; workflow generates a Kubernetes Secret and applies it in the same namespace.

## CI/CD (GitHub Actions)

### Development

- **Workflow**: `.github/workflows/eks-deploy.dev.yaml`
- **Trigger**: Push to `development`, `feature/*`, `hotfix/*`
- **Steps**: Checkout app and infra; assume IAM role (ace-dev-eks-role); login to ECR; build Docker image with infra Dockerfile; tag `dev-<timestamp>`; push to ECR; fetch secrets from `ace/dev/configuration-secrets`; generate and apply K8s Secret; update image in Job YAML and namespace to **dev**; update Ingress host to `dev-*`; delete existing `ace-configuration-job` if present; apply manifests (Job and related resources).
- **Cluster**: **development-ace-eks**

### Production

- **Workflow**: `.github/workflows/eks-deploy.prod.yaml`
- **Trigger**: Push to `main`
- **Steps**: Same pattern; IAM role **ace-prod-eks-role**; secrets from `ace/prod/configuration-secrets`; namespace **prod**; no Ingress host rewrite; apply to **production-ace-eks**.
- **Image tag**: `prod-<timestamp>`

### Demo

- **Workflow**: `.github/workflows/eks-deploy.demo.yaml`
- **Trigger**: Push to `demo`, `demo/*`
- **Steps**: Same pattern; cluster **development-ace-eks**; namespace **demo**; secrets from `ace/demo/configuration-secrets`; Ingress host set to `demo-*`; optional Route 53 update for demo DNS.
- **Image tag**: `demo-<timestamp>`

### Notes

- Dockerfile lives in **ace-infra**; workflows depend on `INFRA_REPO_TOKEN` and infra checkout.
- Required GitHub secrets: `INFRA_REPO_TOKEN`, `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY` (for assume-role and ECR).
