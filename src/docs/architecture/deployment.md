# ACE Deployment Architecture

This document describes **where and how ACE runs**: local development, staging, and production, plus Kubernetes/EKS and AWS. Use it when deploying, changing infra, or adding a new service.

---

## Environments

| Environment | Purpose | Typical location |
|-------------|---------|------------------|
| **Local** | Development on developer machines | docker-compose (local-env), per-service dev servers |
| **Development (dev)** | Integration and daily dev | EKS **development-ace-eks**, namespace **dev**, us-east-1 |
| **Staging (stg)** | Pre-production validation | EKS **development-ace-eks**, namespace **stg**, us-east-1 |
| **Demo** | Demos and showcases | EKS **development-ace-eks**, namespace **demo**, us-east-1 |
| **Production (prod)** | Live system | EKS **production-ace-eks**, namespaces **prod**, **apis**, **ace-system**, us-east-1 |

All ACE infrastructure is in **AWS region us-east-1**. Do not create ACE resources in other regions unless an approved design explicitly requires it. See [../rules/infrastructure-rules.md](../rules/infrastructure-rules.md).

---

## Local (docker-compose)

- **local-env/** contains (or references) docker-compose and configs for running ACE services locally.
- **Development** requires docker-compose for local runs; see project rules. Each service may have its own dev server (e.g. Vite for frontend, NestJS for backend); docs are in each repo or in [../environments/](../environments/).

---

## Kubernetes (EKS)

- ACE workloads run on **EKS**. Clusters: **development-ace-eks** (dev, stg, demo namespaces) and **production-ace-eks** (prod, apis, ace-system).
- **Namespaces**: Use the correct namespace per environment and service (e.g. `dev`, `stg`, `ace-system`, `apis` as defined in the cluster). Do not deploy ACE application pods to `default` or ad-hoc namespaces unless documented.
- **Manifests**: Kubernetes manifests (Deployment, Service, Ingress, ConfigMap, etc.) live under **ace-infra** in per-service folders (e.g. `ace-stack-backend/`, `ace-db-gateway/`). Each service typically has a Dockerfile and YAML manifests; new services follow the same structure.
- **Ingress/ALB**: Use the existing ALB Ingress Controller and group names (e.g. `alb.ingress.kubernetes.io/group.name`) for consistent routing and TLS.

**Agents**: When adding or changing K8s manifests in ace-infra, set the appropriate `namespace` and follow existing naming (e.g. `*-deployment.yaml`, `*-service.yaml`, `*-ingress.yaml`). See [../rules/infrastructure-rules.md](../rules/infrastructure-rules.md).

---

## AWS (High-Level)

- **Region**: us-east-1 for all ACE resources.
- **ECR**: Docker images for ACE services are built (e.g. via GitHub Actions) and pushed to **ECR** in us-east-1. Image naming and tagging follow the project convention (e.g. by service name and tag).
- **Secrets**: Runtime secrets for applications use **AWS Secrets Manager**. Path pattern for app env vars: **`ace/<env>/<service>-secrets`** (e.g. `ace/dev/ace-stack-backend-secrets`, `ace/prod/ace-db-gateway-secrets`). These secrets are for **environment variables that the application uses** at runtime only; see [../rules/infrastructure-rules.md](../rules/infrastructure-rules.md).
- **CI/CD secrets** (e.g. deploy credentials, GitHub tokens) are in **GitHub Environments**, not in repo variables or workflow file content.

---

## CI/CD

- **Pipeline**: GitHub Actions. Typical flow: build on push/tag, push images to ECR, deploy to EKS (dev/staging/prod) using manifests from ace-infra.
- **Production (and optionally staging)** deploys must use **approvals** (e.g. GitHub Environment protection rules). Do not bypass approval for production.
- **Lint and type check** are required before merge; tests and Docker build/push as defined per repo.

**Agents**: When adding or changing deployment workflows, ensure production (and optionally staging) require approvals. Use GitHub Environments for secrets; build and push to ECR us-east-1; apply manifests from ace-infra. See [../rules/infrastructure-rules.md](../rules/infrastructure-rules.md).

---

## Resource Tags

Every resource that belongs to ACE must have:

- **Project** = **ACE**
- **Environment** = **&lt;ENV&gt;-ACE** (e.g. `dev-ACE`, `staging-ACE`, `prod-ACE`)

Used for cost allocation, filtering, and governance. See [../rules/infrastructure-rules.md](../rules/infrastructure-rules.md).

---

## Links

- **Infrastructure rules**: [../rules/infrastructure-rules.md](../rules/infrastructure-rules.md)
- **Environments docs**: [../environments/](../environments/) for local setup and deployment targets
- **ace-infra**: Repository that holds Terraform, K8s manifests, and infra docs; add new services there following the existing folder structure.
