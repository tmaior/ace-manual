# ace-infra – Repository structure

This document describes the layout of the **ace-infra** repository as reflected in the codebase. For detailed procedures (adding a service, Terraform workflow, scripts usage), see the other docs in this folder and [../../infrastructure/ace-infra-repository.md](../../infrastructure/ace-infra-repository.md).

---

## Top-level directories

| Path | Purpose |
|------|---------|
| **cluster/** | Terraform root for the EKS cluster and related AWS resources. |
| **cluster/terraform/** | main.tf, variables.tf, outputs.tf, backend and provider config; uses modules from terraform-library. |
| **cluster/environments/** | Per-environment tfvars (e.g. development/terraform.tfvars, production/terraform.tfvars). |
| **terraform-library/** | Reusable Terraform modules: vpc, eks, eks-general-configs, rds, documentDB, efs, redis, docs-sync-queue, codepipeline. |
| **ace-&lt;service&gt;/** | One folder per deployed ACE service: Dockerfile(s), \*.deployment.yaml, \*.service.yaml, \*.ingress.yaml (and optional README, scripts). |
| **helm-charts/** | Helm charts (e.g. ace-litellm, ace-daytona-proxy, ace-slackbot). |
| **scripts/** | Automation scripts: SQS, Route53, DB setup, validation, cleanup, tests. |
| **iam/** | IAM policy JSON files (e.g. route53 for Jira integration). |
| **docs/** | Repo-level docs (e.g. eks-pods-inventory.md, demo-environment). |
| **demo-env/** | Optional Terraform/config for a dedicated demo environment. |
| **local-env/** | Local development references (main local setup may live at repo root local-env). |
| **monitoring/** | Monitoring and analysis (Prometheus, Grafana, etc.). |
| **bkp-terraform-library/** | Backup/legacy Terraform modules; use **terraform-library/** for new work. |

---

## terraform-library modules

Used by **cluster/terraform/main.tf**:

| Module | Purpose |
|--------|---------|
| **vpc** | VPC, public/private subnets, Internet Gateway, NAT Gateway, route tables; subnet tags for ELB (kubernetes.io/role/elb, internal-elb). |
| **eks** | EKS cluster, node group, IAM roles (cluster + node), launch template. |
| **eks-general-configs** | Helm: metrics-server, EBS CSI driver, EFS CSI driver; K8s namespaces from variable; optional New Relic; ALB controller when enabled. |
| **rds** | RDS PostgreSQL (single or multi-AZ). Main app DB per environment. |
| **documentDB** | DocumentDB (MongoDB-compatible) cluster. Used by commands-api and others. |
| **efs** | EFS file system for shared persistent storage. |
| **redis** | ElastiCache Redis (e.g. cache.t3.micro). Cache and session storage. |
| **docs-sync-queue** | SQS FIFO queue for docs sync (Knowledge Base). Often created via script; module available for optional Terraform management. |
| **codepipeline** | AWS CodePipeline (GitHub Actions is the primary CI/CD). |

---

## Service folders (ace-*)

Naming: **ace-&lt;service-name&gt;** (e.g. ace-db-gateway, ace-jira-integration). Some app names differ in infra (e.g. ace-stack-backend → ace-web-backend, ace-dashboard-frontend → ace-web-frontend).

Typical contents:

| File / pattern | Purpose |
|----------------|---------|
| **Dockerfile** or **ace.&lt;service&gt;.Dockerfile** | Image build; CI/CD uses this or the one in the app repo as documented. |
| **ace.&lt;service&gt;.deployment.yaml** | Kubernetes Deployment: replicas, image, envFrom secretRef, ports. |
| **ace.&lt;service&gt;.service.yaml** | Kubernetes Service (ClusterIP or LoadBalancer). |
| **ace.&lt;service&gt;.ingress.yaml** | Ingress for ALB (host, path, TLS); uses alb.ingress.kubernetes.io annotations. |
| **README.md** | Service-specific infra notes (optional). |
| **scripts/** | One-off or DB/queue setup scripts for that service. |

Current service folders (from repo): ace-alb-ingress, ace-commands-api, ace-configuration, ace-daytona-proxy, ace-db-gateway, ace-docs-api (deprecated), ace-jira-integration, ace-jira-mcp, ace-litellm, ace-llm, ace-mem0, ace-ops-bot, ace-ops-scheduler, ace-passwordbot (deprecated), ace-presidio, ace-prometheus, ace-sandbox, ace-sec-api, ace-sec-bot, ace-slackbot, ace-web-backend, ace-web-frontend, plus supporting (redisinsights, mongo-express, wikijs, etc.).

---

## Links

- [Terraform and environments](./terraform-and-environments.md) – cluster Terraform, variables, workspaces, outputs.
- [Kubernetes manifests and services](./kubernetes-manifests-and-services.md) – manifest pattern and example.
- [Scripts](./scripts.md) – scripts list and usage.
- [../../infrastructure/ace-infra-repository.md](../../infrastructure/ace-infra-repository.md) – full layout and “adding a new service” steps.
