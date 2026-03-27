# ace-infra – Overview

## What it is

**ace-infra** is the infrastructure-as-code repository for the ACE (Automated Cloud Engineer) platform. It holds Terraform (VPC, EKS, RDS, DocumentDB, EFS, Redis), Kubernetes manifests per service (Deployment, Service, Ingress), Dockerfiles used by CI/CD, Helm charts for some components, IAM policies, and automation scripts. All ACE AWS and EKS resources follow the patterns and modules defined here.

## Purpose

- **Single source of truth**: One place for cluster provisioning (Terraform modules and cluster root), per-service Kubernetes definitions, and operational scripts. No ACE AWS or EKS resources should be created outside ace-infra unless an approved design requires it.
- **Reusable Terraform**: The **terraform-library/** provides modules (vpc, eks, rds, documentDB, efs, redis, eks-general-configs, docs-sync-queue, codepipeline) consumed by **cluster/terraform/** to create the full environment per workspace (e.g. development, production).
- **Consistent service deployment**: Each deployable ACE service has a folder **ace-&lt;service-name&gt;** with Dockerfile(s) and K8s manifests (deployment, service, ingress when exposed via ALB). CI/CD builds images and applies these manifests.
- **Operational automation**: Scripts under **scripts/** handle SQS queue creation, Route53 updates (demo, Jira), DB setup (e.g. Jira integration), validation, and cleanup.

## High-level architecture

- **Cluster Terraform** (`cluster/terraform/`): Composes terraform-library modules to create VPC, EKS cluster and node group, RDS PostgreSQL, DocumentDB, EFS, Redis, and EKS add-ons (metrics-server, EBS/EFS CSI drivers, ALB controller, namespaces). Per-environment values live in **cluster/environments/&lt;env&gt;/terraform.tfvars**; Terraform workspaces align with environments.
- **Kubernetes manifests**: Each **ace-*/** folder contains YAML for Deployment (image, envFrom secretRef, ports), Service, and optionally Ingress (ALB Ingress Controller annotations, hostname, TLS). Namespaces (e.g. `dev`, `stg`, `prod`) are defined in cluster Terraform and used in manifest metadata.
- **Secrets**: Services use **envFrom secretRef**; secret values are stored in AWS Secrets Manager (e.g. `ace/<env>/<service>-secrets`) and injected by CI/CD or external-secrets. No secret values are stored in manifests.
- **Helm**: Some components (e.g. ace-litellm, ace-daytona-proxy, ace-slackbot) are deployed via **helm-charts/** instead of raw YAML.

## Main components

| Component | Role |
|-----------|------|
| **cluster/terraform/** | Terraform root: main.tf (modules), variables.tf, outputs.tf, backend and provider config. |
| **cluster/environments/** | One folder per environment with terraform.tfvars (region, VPC, EKS sizing, RDS, namespaces). |
| **terraform-library/** | Reusable modules: vpc, eks, eks-general-configs, rds, documentDB, efs, redis, docs-sync-queue, codepipeline. |
| **ace-&lt;service&gt;/** | Per-service Dockerfile and K8s manifests (deployment, service, ingress). |
| **helm-charts/** | Helm charts for selected components (e.g. ace-litellm, ace-daytona-proxy, ace-slackbot). |
| **scripts/** | Automation: create-docs-sync-queue, update-route53-demo, apply-route53-jira-policy, jira-integration-db-setup, validate-demo-environment, delete-kb-resources, tests. |
| **iam/** | IAM policy documents (e.g. Route53 policy for Jira integration). |

## What you can do with this repo

- Provision or update an ACE environment (development, production) by running Terraform in **cluster/terraform/** with the appropriate workspace and tfvars.
- Add or change a service by editing or creating an **ace-&lt;service&gt;/** folder (Dockerfile + deployment/service/ingress) and updating CI/CD to build and deploy.
- Run operational scripts (SQS, Route53, DB setup, validation) as documented in [scripts.md](./scripts.md).
- Extend Terraform by adding or modifying modules in **terraform-library/** and wiring them in **cluster/terraform/main.tf**.

## Related documentation

- [Repository structure](./repository-structure.md)
- [Terraform and environments](./terraform-and-environments.md)
- [Kubernetes manifests and services](./kubernetes-manifests-and-services.md)
- [Scripts](./scripts.md)
- Infrastructure (cross-repo): [../../infrastructure/](../../infrastructure/) – [ace-infra-repository.md](../../infrastructure/ace-infra-repository.md), [terraform-modules.md](../../infrastructure/terraform-modules.md), [kubernetes-and-deployment.md](../../infrastructure/kubernetes-and-deployment.md).
- [Architecture service catalog](../../architecture/service-catalog.md), [Infrastructure rules](../../rules/infrastructure-rules.md).
