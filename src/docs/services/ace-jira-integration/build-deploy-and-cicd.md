# Build, deploy and CI/CD

## Build

### Prerequisites

- Node.js (version per repo and Dockerfile; ES modules)
- npm (or yarn if used in repo)
- For JSX views: Parcel (dev dependency); run `npm run build` to compile views for browser and node.

### Local build and run

```bash
# Install dependencies
npm install

# Build JSX views (if using hello-world.jsx or other .jsx)
npm run build
# or watch: npm run watch-jsx

# Start server (development: config.json development, port 3000)
npm start
```

Development mode: addon auto-registers with a host when `app.get('env') === 'development'`. Use a tunnel (e.g. ngrok) pointing to the app URL and set APP_URL (or AC_LOCAL_BASE_URL) so Jira can reach webhooks.

```bash
# Watch server only (nodemon)
npm run watch-server

# Full watch (server + JSX browser + JSX node)
npm run watch
```

Lint: `npm run lint` (ESLint on app.js and routes).

### Docker image (ace-infra)

- **Dockerfile**: In **ace-infra** at `ace-jira-integration/` (workflows reference infra repo and app name `jira-integration`).
- **Build context**: Application repo root; Dockerfile path from infra checkout.
- **Image**: Pushed to ECR, e.g. `975635808270.dkr.ecr.us-east-1.amazonaws.com/ace/jira-integration:latest`.

Build example (with infra repo checked out):

```bash
docker build -f infra-repo/ace-jira-integration/<Dockerfile> -t ace/jira-integration:local .
```

(Exact Dockerfile name may be in ace-infra; e.g. `ace.jira-integration.Dockerfile` or similar.)

## Deploy (Kubernetes)

- **Manifests**: In **ace-infra** under `ace-jira-integration/`:
  - Deployment: `ace.jira-integration.deployment.yaml`
  - Service: `ace.jira-integration.service.yaml`
  - Ingress: `ace.jira-integration.ingress.yaml`
- **Namespace**: **dev** (development), **stg** (staging), **prod** (production); set by CI.
- **Secrets**: From AWS Secrets Manager **ace/<env>/jira-integration-secrets**; workflow generates Kubernetes Secret `jira-integration-secrets` and deployment uses `envFrom: secretRef: jira-integration-secrets`.
- **Database**: RDS/Postgres for production store; `DATABASE_URL` must be set in the secret. Script `ace-infra/scripts/jira-integration-db-setup.sh` can create/update DB and write DATABASE_URL into the secret.

Container port: 3000.

## CI/CD (GitHub Actions)

Workflows live in the **application repo** (`ace-jira-integration/.github/workflows/`).

### Development

- **Workflow**: `eks-deploy.dev.yaml`
- **Trigger**: Push to branches (e.g. development or feature branches; see workflow for exact triggers).
- **Steps**: Checkout app and infra; assume IAM role; ECR login; build Docker image; push image; fetch secrets from `ace/dev/jira-integration-secrets`; apply K8s Secret and manifests; update DNS (Route53) for dev-jira hostname to ALB.
- **Cluster**: development-ace-eks (or per workflow)
- **Namespace**: dev

### Staging

- **Workflow**: `eks-deploy.stg.yaml`
- **Trigger**: As defined in workflow (e.g. push to staging branch).
- **Namespace**: stg
- **DNS**: stg-jira.ace.ezops.cloud

### Production

- **Workflow**: `eks-deploy.prod.yaml`
- **Trigger**: Push to main (or as defined).
- **Namespace**: prod
- **DNS**: jira.ace.ezops.cloud

### Other workflows

- **build-new-tag.yaml**: Build and tag image.
- **version-selection-deploy.yaml**: Deploy with version selection; uses `ace/dev/jira-integration-secrets`.

## URLs (per environment)

| Env   | App URL |
|-------|---------|
| Local | https://jira.localdash.ace.ezops.cloud |
| Dev   | https://dev-jira.ace.ezops.cloud |
| Stg   | https://stg-jira.ace.ezops.cloud |
| Prod  | https://jira.ace.ezops.cloud |

Ensure APP_URL (or equivalent) is set in secrets for the descriptor baseUrl so Jira uses the correct webhook and descriptor URLs.

## Route53 / DNS

ace-infra script `scripts/apply-route53-jira-policy.sh` applies IAM policy for jira-integration to upsert Route53 records (dev-jira, stg-jira, jira.ace.ezops.cloud). Deploy workflows get the Ingress ALB hostname and call AWS Route53 to point the hostname to the ALB.
