# Demo Environment (EKS)

This document describes the **demo** environment used for demonstrations and showcases. Demo runs in the same EKS cluster as development and staging, in a dedicated namespace, and includes **frontend (dashboard)**, **backend (stack backend)**, **DB Gateway**, **Redis**, and optional **RedisInsight**.

---

## Purpose

- **Demos and showcases** — run a full ACE stack (dashboard, API, DB gateway, Redis) for live demos without affecting dev or stg.
- **Isolation** — namespace **demo**, database **ace_configuration_demo**, Redis key prefix **demo:**, and secrets path **ace/demo/**.

---

## Cluster and namespace

| Item | Value |
|------|--------|
| **Cluster** | **development-ace-eks** |
| **Region** | us-east-1 |
| **Namespace** | **demo** |

**kubectl context** (example): `arn:aws:eks:us-east-1:<account>:cluster/development-ace-eks`

```bash
kubectl config set-context --current --namespace=demo
kubectl get pods
```

---

## Workloads and URLs

Demo deploys a **subset** of ACE services—only those that have an `eks-deploy.demo.yaml` workflow. The following are deployed to namespace **demo** with demo-specific hostnames:

| Application | URL | Notes |
|-------------|-----|--------|
| **Dashboard Frontend** (ace-dashboard-frontend) | `demo-dashboard.ace.ezops.cloud` | Port 4173 (container); ingress on 80/443. |
| **Stack Backend** (ace-stack-backend) | `demo-api-ace.ace.ezops.cloud` | Port 8080 (container). Uses IAM role `ace-web-backend-demo-role`. |
| **DB Gateway** (ace-db-gateway) | `demo-db-gateway.ace.ezops.cloud` | Port 80 (container). |
| **ace-configuration** | `demo-configuration.ace.ezops.cloud` | Schema/migrations; may run as Job or Deployment. Workflow: `ace-configuration/.github/workflows/eks-deploy.demo.yaml` (branch demo, demo/*). |
| **Redis** | (internal K8s service) | Redis key prefix **demo:** for isolation. |
| **RedisInsight** | (optional) | Redis UI for demos. |

**Database**: Demo uses database **ace_configuration_demo** on **production** RDS (URLs without `dev-` prefix). See ace-infra docs (e.g. demo-environment) for details.

---

## Services not in demo

The following ACE services **do not** have an `eks-deploy.demo.yaml` workflow and are **not** deployed in the demo namespace. They run only in **dev**, **stg**, and/or **prod**:

| Service | Deploy environments (from workflows) | Notes |
|---------|-------------------------------------|--------|
| **ace-ops-scheduler** | dev, stg, prod | No demo workflow. Demo does not run scheduled jobs. |
| **ace-commands-api** | dev, prod | No demo (or stg) workflow. |
| **ace-slackbot** | dev, stg, prod | No demo workflow. Demo has no Slack bot. |
| **ace-sec-bot** | dev, stg, prod | No demo workflow. |
| **ace-ops-bot** | dev, stg, prod | No demo workflow. |

So in demo: **no ops-scheduler, no commands-api, no bots**. The demo stack is frontend + backend + db-gateway + configuration (and Redis/RedisInsight). Features that depend on ops-scheduler, commands-api, or bots are not available in demo unless they are called from demo backend to another environment (not recommended).

---

## How to deploy

- **CI/CD**: **GitHub Actions** in each application repo. Demo deploy is triggered by:
  - **ace-dashboard-frontend**: push to branch **demo**, **demo/***, or **feature/demo-*** (workflow `eks-deploy.demo.yaml`). Ingress host is set to `demo-dashboard.ace.ezops.cloud`; Route53 is updated via `ace-infra/scripts/update-route53-demo.sh`.
  - **ace-stack-backend**: push to branch **demo** or **demo/*** (workflow `eks-deploy.demo.yaml`). Ingress host is set to `demo-api-ace.ace.ezops.cloud`; service account uses IAM role `ace-web-backend-demo-role`; Route53 updated via same script.
  - **ace-db-gateway**: push to **demo** or **demo/*** (workflow `eks-deploy.demo.yaml`). Ingress host `demo-db-gateway.ace.ezops.cloud`; Route53 via same script.
  - **ace-configuration**: push to **demo** or **demo/*** (workflow `eks-deploy.demo.yaml`). Deploys to namespace demo; URL `demo-configuration.ace.ezops.cloud`. May run as Job (migrations) or Deployment; see ace-infra docs/demo-environment.
- **Manifests**: Manifests live in **ace-infra** (e.g. ace-web-frontend, ace-web-backend, ace-db-gateway). The workflow checks out ace-infra and replaces `namespace:` with **demo** and (for demo) replaces the ingress host with the demo hostname, then applies to the cluster.
- **Image tags**: Demo often uses tags such as **demo-** or **demo-<timestamp>** (e.g. in ECR: `ace/web-frontend:demo-*`, `ace/web-backend:demo-*`, `ace/db-gateway:demo-*`). Use tags suitable for demo; do not point demo at production images.

---

## Access

- **Dashboard**: https://demo-dashboard.ace.ezops.cloud (or HTTP as configured).
- **API**: https://demo-api-ace.ace.ezops.cloud (or HTTP as configured).
- **DB Gateway**: https://demo-db-gateway.ace.ezops.cloud (or HTTP as configured).
- **Configuration** (if deployed): https://demo-configuration.ace.ezops.cloud (or HTTP as configured).

Use these URLs only for demo; do not reuse production or staging hostnames.

---

## AWS Secrets Manager (demo apps)

Secrets for demo are stored in **AWS Secrets Manager** under the path **`ace/demo/`**. The CI/CD workflow fetches the secret for each app and injects it into the Kubernetes Secret (e.g. `web-backend-secrets`) in namespace **demo**, which the deployment mounts as env vars.

| Secret ID | Used by | Purpose |
|-----------|---------|---------|
| **ace/demo/web-frontend-secrets** | web-frontend (dashboard) | Build-time or runtime env (e.g. API URL). Fetched in workflow and applied as K8s Secret. |
| **ace/demo/web-backend-secrets** | web-backend (stack backend) | Runtime env: ACE_GATEWAY_URL, REDIS_URL, AWS_REGION, AWS_SECRET_NAME (no LLM_URL; see “How demo uses resources…”). Injected into pods. |
| **ace/demo/db-gateway-secrets** | ace-db-gateway | DB connections, JWT validation. Injected into pods. |
| **ace/demo/configuration-secrets** | ace-configuration (if deployed) | Configuration service env. Injected into pods/Job. |

**Backend (web-backend)** — Current keys in **ace/demo/web-backend-secrets**: `ACE_GATEWAY_URL` (points to shared **db-gateway.ace.ezops.cloud**), `REDIS_URL` (points to **production** Redis), `AWS_REGION`, `AWS_SECRET_NAME`. There is no `LLM_URL` in this secret. Other keys (JWT, OAuth, etc.) may exist in other envs; manage all keys in AWS Secrets Manager and the service’s env docs.

**CI/CD**: Workflows use GitHub Environment **demo** and assume a role (e.g. `ace-dev-eks-role`); they call `aws secretsmanager get-secret-value --secret-id ace/demo/<app>-secrets` and apply the result as a K8s Secret in namespace **demo**. See each repo’s `eks-deploy.demo.yaml`.

---

## Shared infrastructure: demo uses production resources

Demo does **not** run its own RDS or Redis cluster. The resources it uses for services not deployed in demo are **production** ones. Convention: **dev** URLs use the prefix **`dev-`**; **prod** URLs do not (e.g. `db-gateway.ace.ezops.cloud`, `production-ace-redis-cluster`).

| Resource | Who uses it in demo | Actually points to |
|----------|---------------------|--------------------|
| **DB Gateway (HTTP)** | Backend **ACE_GATEWAY_URL** | **Production** — `db-gateway.ace.ezops.cloud` (no `dev-` prefix). |
| **Redis (backend)** | Backend **REDIS_URL** (web-backend-secrets) | **Production** ElastiCache Redis (`production-ace-redis-cluster`). Isolation by key prefix **demo:** if configured in app. |
| **RDS / Redis / SQS (db-gateway)** | db-gateway (db-gateway-secrets) | **Production** — RDS, Redis, and queues used by demo db-gateway are prod (no `dev-` in hostnames). |
| **VPC / EKS cluster** | All demo workloads | **development-ace-eks**, namespace **demo** only (only compute runs in dev cluster; data and shared services are prod). |

---

## How demo uses resources not deployed in demo (from Secrets Manager)

The following was verified by reading the **ace/demo/** secrets in AWS Secrets Manager. Demo uses **production** resources for shared services (RDS, Redis, DB Gateway, SQS). Convention: **dev** URLs have the prefix **`dev-`**; **prod** URLs do not — so URLs without `dev-` (e.g. `db-gateway.ace.ezops.cloud`, `production-ace-redis-cluster`) are production.

### ace/demo/web-backend-secrets

| Key | Value (summary) | Meaning |
|-----|------------------|---------|
| **ACE_GATEWAY_URL** | `https://db-gateway.ace.ezops.cloud` | **Production** DB Gateway (no `dev-` prefix). Backend calls prod gateway, not demo-db-gateway or dev. |
| **REDIS_URL** | `redis://production-ace-redis-cluster...` | **Production** ElastiCache Redis. Backend uses prod Redis (omnichannel, cache, etc.). Isolation by key prefix **demo:** if configured. |
| **AWS_REGION**, **AWS_SECRET_NAME** | (set) | Used by the app for AWS/Secrets. |

There is **no LLM_URL** (and no COMMANDS_API_URL, etc.) in this secret; the backend falls back to defaults. LLM/chat in demo either uses a default that resolves to prod or is disabled.

### ace/demo/db-gateway-secrets

The **demo db-gateway** is configured to use **production** infrastructure. Hostnames in these URLs do **not** use the `dev-` prefix, so they point to **prod** RDS, **prod** Redis, and **prod** SQS:

| Key | Target | Meaning |
|-----|--------|---------|
| **CONFIGURATION_DB_URL** | Prod RDS (no `dev-` in host) | Configuration DB on **production** RDS. Demo may use DB **ace_configuration_demo** or the prod config DB. |
| **SCHEDULE_DB_URL** | Prod RDS | Schedule/ops-scheduler DB on **production** RDS. Demo db-gateway reads/writes scheduler data in prod; **ops-scheduler** (not in demo) runs in prod and uses the same DB. |
| **LLM_DB_URL** | Prod RDS | LLM DB on **production** RDS. |
| **DOCS_DB_URL** | Prod RDS | Docs DB on **production** RDS. |
| **REDIS_HOST** | Prod Redis (no `dev-` in host) | **Production** ElastiCache Redis. Demo db-gateway uses **prod** Redis. |
| **QUEUE_DOCS_SYNC_URL** | Prod SQS queue URL | Docs-sync queue in **production**; **commands-api** (runs in prod) consumes it. Demo can enqueue jobs that prod commands-api processes. |
| **BACKEND_DB_GATEWAY_TOKEN**, **SCHEDULLER_DB_GATEWAY_TOKEN** | (tokens) | Auth for backend and scheduler when calling the gateway. |

So: **demo db-gateway** uses **prod** RDS, **prod** Redis, and **prod** SQS. Scheduler and LLM **data** live in prod; **ops-scheduler** and **commands-api** run only in **prod** (not in demo) and process that data or queue.

### ace/demo/configuration-secrets

Configuration service in demo: **DATABASE_URL**, **DB_***, **SLACK_BOT_TOKEN**, **SLACK_SIGNING_SECRET**, **OPENAI_API_KEY**, etc. **DATABASE_URL** typically points to **production** RDS (no `dev-` prefix), e.g. DB **ace_configuration_demo** or the prod config DB.

### Summary: how demo uses non-demo resources (all production)

| Resource | Used by demo via | Actually points to |
|----------|-------------------|---------------------|
| **DB Gateway (HTTP)** | Backend **ACE_GATEWAY_URL** | **Production** — `db-gateway.ace.ezops.cloud` (no `dev-`). |
| **Redis (backend)** | Backend **REDIS_URL** | **Production** ElastiCache Redis. |
| **RDS (config, schedule, LLM, docs)** | Demo **db-gateway-secrets** | **Production** RDS (URLs without `dev-`). |
| **Redis (db-gateway)** | Demo **db-gateway-secrets** REDIS_HOST | **Production** ElastiCache Redis. |
| **SQS (docs-sync)** | Demo **db-gateway-secrets** QUEUE_DOCS_SYNC_URL | **Production** queue; **prod** commands-api consumes it. |
| **LLM (service)** | Not set in demo backend | Default or disabled; if used, points to prod. |
| **ops-scheduler** | Not in demo | Runs in **prod**; data in **prod** RDS. |
| **commands-api** | Not in demo | Runs in **prod**; consumes **prod** SQS. |

Demo uses **production** resources (gateway, RDS, Redis, SQS) for everything not deployed in the demo namespace; **dev** is identified by the **`dev-`** URL prefix and is not used by demo for these shared services.

---

## Links

- **ace-infra** — Manifests (ace-web-frontend, ace-web-backend, ace-db-gateway, ace-configuration if present), scripts (`scripts/update-route53-demo.sh`), and docs (`docs/demo-environment/`: maintenance-guide.md, troubleshooting-guide.md, training-slides.md, route53-implementation-summary.md, deploy-guide.md).
- **Repos with demo workflow** — ace-dashboard-frontend, ace-stack-backend, ace-db-gateway, ace-configuration (each has `.github/workflows/eks-deploy.demo.yaml`). No demo workflow: ace-ops-scheduler, ace-commands-api, ace-slackbot, ace-sec-bot, ace-ops-bot.
- **Frontend deploy** — ace-dashboard-frontend `.github/workflows/eks-deploy.demo.yaml` (triggers: demo, demo/*, feature/demo-*).
- **Backend deploy** — ace-stack-backend `.github/workflows/eks-deploy.demo.yaml` (triggers: demo, demo/*).
- **DB Gateway deploy** — ace-db-gateway `.github/workflows/eks-deploy.demo.yaml` (triggers: demo, demo/*).
- **Configuration deploy** — ace-configuration `.github/workflows/eks-deploy.demo.yaml` (triggers: demo, demo/*).
- **development** — [development.md](./development.md) (same cluster).
- **env-vars-and-secrets** — [env-vars-and-secrets.md](./env-vars-and-secrets.md)
- **troubleshooting** — [troubleshooting.md](./troubleshooting.md)
