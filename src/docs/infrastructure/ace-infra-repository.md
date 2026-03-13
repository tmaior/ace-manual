# ace-infra Repository

This document describes the **structure and layout** of the **ace-infra** repository. ace-infra holds infrastructure as code (Terraform, Kubernetes manifests), Dockerfiles for ACE services, Helm charts, scripts, and documentation. All ACE infrastructure is defined or referenced here.

---

## Repository location and purpose

- **Repository**: ace-infra (sibling to other ACE repos under the same parent directory, e.g. `ace/`).
- **Purpose**: Single place for Terraform (VPC, EKS, RDS, DocumentDB, EFS, Redis, etc.), Kubernetes manifests (Deployment, Service, Ingress) per service, Dockerfiles used by CI/CD to build images, Helm charts for some components, and scripts (SQS, Route53, DB setup, validation).

Do not create ACE AWS or EKS resources outside the patterns and modules defined in ace-infra unless an approved design explicitly requires it.

---

## Top-level layout

| Path | Purpose |
|------|---------|
| **cluster/** | Terraform root for EKS cluster and related resources (VPC, EKS, RDS, DocumentDB, EFS, Redis). |
| **cluster/terraform/** | Main Terraform config (main.tf, variables.tf, outputs.tf, backend.tf, providers.tf). |
| **cluster/environments/** | Per-environment tfvars (e.g. `development/terraform.tfvars`, `production/terraform.tfvars`). |
| **terraform-library/** | Reusable Terraform modules (vpc, eks, rds, documentDB, efs, redis, eks-general-configs, docs-sync-queue, codepipeline). |
| **ace-<service>/** | One folder per deployed ACE service: Dockerfile(s), K8s manifests (*.deployment.yaml, *.service.yaml, *.ingress.yaml). |
| **helm-charts/** | Helm charts (e.g. ace-litellm, ace-daytona-proxy, ace-slackbot). |
| **scripts/** | Automation scripts: SQS queue creation, Route53 updates (demo, Jira), DB setup (Jira), validation, cleanup. |
| **docs/** | Infrastructure documentation (e.g. EKS pods inventory, demo environment). |
| **demo-env/** | Terraform/config for demo environment (optional/secondary setup). |
| **iam/** | IAM policy documents (e.g. Route53 policy for Jira integration). |
| **local-env/** | Local development references (may be symlink or copy; main local setup is in **local-env** at repo root). |
| **monitoring/** | Monitoring and analysis (Prometheus, Grafana, etc.). |
| **bkp-terraform-library/** | Backup/legacy Terraform modules; prefer **terraform-library/** for new work. |

---

## Service folders (ace-*)

Each deployable ACE service has a folder under ace-infra with a consistent naming pattern and typical contents.

**Naming**: `ace-<service-name>` (e.g. `ace-db-gateway`, `ace-stack-backend` is deployed as **ace-web-backend** in ace-infra, **ace-dashboard-frontend** as **ace-web-frontend**).

**Typical contents per service folder**:

| File / pattern | Purpose |
|----------------|---------|
| **Dockerfile** or **ace.<service>.Dockerfile** | Build image for the service. CI/CD uses this (or the one in the app repo if documented). |
| **ace.<service>.deployment.yaml** | Kubernetes Deployment (replicas, image, envFrom secretRef, ports). |
| **ace.<service>.service.yaml** | Kubernetes Service (ClusterIP or LoadBalancer). |
| **ace.<service>.ingress.yaml** | Ingress for ALB (hostname, path, TLS). Uses ALB Ingress Controller annotations (e.g. `alb.ingress.kubernetes.io/group.name`). |
| **README.md** | Service-specific infra notes (optional but recommended). |
| **scripts/** | Any one-off or DB/queue setup scripts for this service (e.g. ace-jira-integration DB setup). |

**Example service list** (as of documentation date; see repo for current list): ace-alb-ingress, ace-commands-api, ace-configuration, ace-daytona-proxy, ace-db-gateway, ace-docs-api (deprecated), ace-jira-integration, ace-jira-mcp, ace-litellm, ace-llm, ace-mem0, ace-ops-bot, ace-ops-scheduler, ace-passwordbot (deprecated), ace-presidio, ace-prometheus, ace-sandbox, ace-sec-api, ace-sec-bot, ace-slackbot, ace-web-backend, ace-web-frontend.

Deprecated services are not used for new work but may still have manifests for existing deployments.

---

## Cluster Terraform (cluster/terraform and cluster/environments)

- **cluster/terraform/**: Main Terraform configuration. It uses modules from **terraform-library/** to create VPC, EKS, RDS, DocumentDB, EFS, Redis, and EKS add-ons (metrics-server, EBS/EFS CSI drivers, ALB controller, namespaces).
- **cluster/environments/**: One subfolder per environment (e.g. **development**, **production**), each with **terraform.tfvars** for that environment (e.g. `environment`, `namespaces`, `vpc_cidr_block`, `eks_*`, `rds_*`).
- **Workspaces**: Terraform workspaces (e.g. `development`, `production`) are used; select or create the workspace and apply with the matching tfvars file. See [terraform-modules.md](./terraform-modules.md).

---

## terraform-library

Reusable modules used by cluster Terraform (and optionally by demo-env):

| Module | Purpose |
|--------|---------|
| **vpc** | VPC, public/private subnets, Internet Gateway, NAT Gateway, route tables. Subnet tags for ELB (e.g. `kubernetes.io/role/elb`, `kubernetes.io/role/internal-elb`). |
| **eks** | EKS cluster, node group, IAM roles (cluster, node), launch template. |
| **eks-general-configs** | Helm releases: metrics-server, EBS CSI driver, EFS CSI driver; Kubernetes namespaces from variable; optional New Relic; ALB controller (when enabled). |
| **rds** | RDS PostgreSQL (single instance or multi-AZ). Used for main app DB per environment. |
| **documentDB** | DocumentDB cluster (MongoDB-compatible). Used by commands-api and other services that need DocumentDB. |
| **efs** | EFS file system (for persistent shared storage when needed). |
| **redis** | ElastiCache Redis (e.g. cache.t3.micro). Used for cache and session storage. |
| **docs-sync-queue** | SQS FIFO queue for docs sync (Knowledge Base). Often created via script; module exists for optional Terraform management. |
| **codepipeline** | AWS CodePipeline (if used for CI/CD; GitHub Actions is the primary CI/CD). |

---

## Helm charts (helm-charts/)

Some components are deployed via Helm rather than raw YAML:

- **ace-litellm**: LiteLLM proxy for LLM service.
- **ace-daytona-proxy**: Daytona proxy.
- **ace-slackbot**: Slack bot (alternative to raw K8s manifests in ace-slackbot/).

Chart-specific READMEs and values live under each chart folder.

---

## Scripts (scripts/)

| Script | Purpose |
|--------|---------|
| **create-docs-sync-queue.sh** | Creates the SQS FIFO queue for docs sync (Knowledge Base). |
| **update-route53-demo.sh** | Updates Route53 records for demo environment. |
| **apply-route53-jira-policy.sh** | Applies IAM policy for Route53 (Jira integration). |
| **jira-integration-db-setup.sh** | DB setup for ACE–Jira integration. |
| **validate-demo-environment.sh** | Validates demo environment setup. |
| **delete-kb-resources.sh** | Cleanup of Knowledge Base–related resources. |
| **get_secrets.sh** | Helper to fetch secrets (structure only; do not document or log secret values). |
| **devops_setup.sh** | DevOps/setup automation (see script for scope). |
| **test-*.sh** | Test/validation scripts (e.g. DB connectivity, performance). |

See [scripts-and-automation.md](./scripts-and-automation.md) for when and how to use each.

---

## Docs (docs/)

- **docs/eks-pods-inventory.md**: Snapshot of EKS pods by namespace (useful for auditing and troubleshooting). Update periodically via `kubectl get pods -A`.
- **docs/demo-environment/**: Demo environment–specific documentation.

When you add or remove services or namespaces, update the relevant docs and the EKS pods inventory as needed.

---

## Local and demo

- **local-env** (in ace-infra or at repo root): Docker Compose and configs for **local** development. The main local setup is documented in [../environments/local.md](../environments/local.md) and usually lives under a **local-env** directory at the ACE root (sibling to ace-infra).
- **demo-env**: Optional Terraform/config for a dedicated demo cluster or environment (e.g. demo-env-eks). See demo-env README and [../environments/demo.md](../environments/demo.md).

---

## Adding a new service to ace-infra

1. Create a folder **ace-<service-name>**.
2. Add a **Dockerfile** (or ace.<service>.Dockerfile) and Kubernetes manifests: **deployment**, **service**, **ingress** (if the service is exposed via ALB). Use the same naming pattern as existing services (e.g. `ace.<service>.deployment.yaml`).
3. Set the correct **namespace** in metadata (e.g. `dev`, `stg`, `prod`, `apis`, `ace-system` as per environment).
4. Use **envFrom secretRef** for environment variables; secrets are stored in AWS Secrets Manager at **ace/<env>/<service>-secrets** and injected by CI/CD or external-secrets. Do not put secret values in manifests.
5. Document required env vars and the Secrets Manager path in the service's own docs and in [../environments/env-vars-and-secrets.md](../environments/env-vars-and-secrets.md).
6. Add the service to [../architecture/service-catalog.md](../architecture/service-catalog.md) and update [../architecture/overview.md](../architecture/overview.md) if needed.
7. Update **docs/eks-pods-inventory.md** after deployment when documenting inventory.

---

## Links

- **Terraform modules**: [terraform-modules.md](./terraform-modules.md)
- **AWS resources**: [aws-resources.md](./aws-resources.md)
- **Kubernetes and deployment**: [kubernetes-and-deployment.md](./kubernetes-and-deployment.md)
- **Scripts**: [scripts-and-automation.md](./scripts-and-automation.md)
- **Infrastructure rules**: [../rules/infrastructure-rules.md](../rules/infrastructure-rules.md)
- **Environments**: [../environments/](../environments/)

---

*Last updated: March 2025*
