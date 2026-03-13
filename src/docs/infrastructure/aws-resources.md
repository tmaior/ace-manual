# AWS Resources

This document describes the **AWS resources** used by the ACE system. It covers region, services (EKS, ECR, VPC, RDS, DocumentDB, Redis, EFS), Secrets Manager **path pattern**, and mandatory tags. **No sensitive data** is included (no account IDs, no secret values, no credentials).

---

## Region

- **All ACE infrastructure** runs in **us-east-1** (N. Virginia).
- Do not create ACE resources in other regions unless an approved design explicitly requires it (e.g. multi-region DR).

---

## EKS (Kubernetes)

- **Clusters**: ACE uses two EKS clusters:
  - **development-ace-eks**: Development, staging, and demo workloads (namespaces: dev, stg, demo, ace-system, etc.).
  - **production-ace-eks**: Production workloads (namespaces: prod, apis, ace-system).
- **Naming pattern**: `<environment>-ace-eks` (e.g. development, production).
- **Node groups**: Managed node groups; instance types and capacity are defined in Terraform (cluster/environments/*.tfvars). Spot instances may be used for cost optimization where appropriate.
- **Networking**: EKS is deployed in **private subnets**; nodes have outbound access via NAT Gateway. Public subnets are tagged for ELB; private subnets for internal ELB where used.

---

## ECR (Container Registry)

- **Region**: us-east-1.
- **Naming**: Repository names follow the pattern **ace/<service>** (e.g. `ace/db-gateway`, `ace/web-backend`, `ace/web-frontend`, `ace/slackbot`, `ace/commands-api`, `ace/llm`, `ace/jira-integration`, `ace/jira-mcp`). Some repos may use a prefix or variant (e.g. `ace/slackbot-dev` for development).
- **Usage**: CI/CD (GitHub Actions) builds images from Dockerfiles (in ace-infra or in the app repo) and pushes to ECR. Kubernetes manifests reference the image by full ECR URL (account ID and region are configured in CI/CD; do not hardcode in docs).
- **Image tagging**: Tags are set by the pipeline (e.g. branch name, commit SHA, or environment tag like `latest`, `dev`, `stg`, `prod`).

---

## VPC and Networking

- **VPC**: One VPC per environment (e.g. development, production), created by Terraform module **terraform-library/vpc**.
- **CIDR**: Defined in cluster/environments/*.tfvars (e.g. `10.0.0.0/16`).
- **Subnets**: Public and private subnets across multiple Availability Zones (e.g. us-east-1a, us-east-1b, us-east-1c). Default pattern uses two or three AZs.
- **Public subnets**: Tagged with `kubernetes.io/role/elb = 1` for ALB created by the AWS Load Balancer Controller.
- **Private subnets**: Tagged with `kubernetes.io/role/internal-elb = 1` for internal load balancers. EKS node groups use private subnets.
- **NAT Gateway**: One NAT Gateway in a public subnet for outbound traffic from private subnets.
- **Internet Gateway**: Attached to the VPC for public subnet outbound/inbound.

---

## RDS (PostgreSQL)

- **Purpose**: Main relational database for ACE (configuration, app data). Applications access it via **ace-db-gateway**; the backend and other services do not connect to RDS directly from application code in production (except where explicitly designed).
- **Engine**: PostgreSQL (e.g. version 17).
- **Deployment**: Terraform module **terraform-library/rds**; one RDS instance per environment (e.g. development-env-db, production-env-db).
- **Instance class**: Defined in tfvars (e.g. db.t3.micro for dev).
- **Storage**: Allocated storage and backup retention defined in tfvars. Encryption and multi-AZ can be enabled per environment.
- **Network**: RDS is in **private subnets**, not publicly accessible. Access from EKS is via security groups and the DB Gateway or allowed CIDRs.
- **Master username**: Stored in Terraform variables/tfvars (sensitive); not documented here. Pattern is often environment-specific (e.g. <env>_ace_admin).

---

## DocumentDB (MongoDB-compatible)

- **Purpose**: Used by services that require MongoDB-compatible storage (e.g. ace-commands-api).
- **Deployment**: Terraform module **terraform-library/documentDB**; cluster name pattern `<environment>-docdb-cluster`.
- **Instance**: Single instance or small cluster per environment (e.g. db.t3.medium, instance count 1) for cost control.
- **Network**: In private subnets; access from EKS only. Allowed CIDR is the VPC CIDR.
- **Backup**: Backup retention and window defined in Terraform.

---

## ElastiCache (Redis)

- **Purpose**: Cache and session storage for backend, bots, and ops-scheduler.
- **Deployment**: Terraform module **terraform-library/redis**; project name pattern `<environment>-ace-redis`.
- **Node type**: Defined in Terraform (e.g. cache.t3.micro).
- **Port**: 6379 (default).
- **Network**: In private subnets; access from EKS via security groups.

---

## EFS (Elastic File System)

- **Purpose**: Shared file storage when needed (e.g. persistent volumes for LLM or other services).
- **Deployment**: Terraform module **terraform-library/efs**; creation token pattern `<environment>-efs`.
- **Performance**: generalPurpose; encryption enabled; lifecycle policy (e.g. transition to IA after 7 days) can be set.
- **Access**: Mount targets in private subnets; NFS access from EKS pods via EFS CSI driver.

---

## Secrets Manager

- **Path pattern**: **ace/<env>/<service>-secrets**
  - Examples (path names only): `ace/dev/ace-stack-backend-secrets`, `ace/dev/ace-db-gateway-secrets`, `ace/stg/ace-slackbot-secrets`, `ace/prod/ace-db-gateway-secrets`.
- **Content**: **Only environment variables** that the application uses at runtime. Each secret is a key-value set that CI/CD or Kubernetes (e.g. External Secrets Operator) injects as env vars into the pod. Do not use these secrets for arbitrary config or large binary data.
- **Environments**: dev, stg, demo, prod (and any new environment short name). Never commit secret values or document them in repos.
- **CI/CD secrets**: Stored in **GitHub Environments**, not in AWS Secrets Manager for pipeline credentials (deploy keys, GitHub tokens, etc.). See [../rules/infrastructure-rules.md](../rules/infrastructure-rules.md).

---

## SQS

- **Docs Sync queue**: FIFO queue for **Knowledge Base** document synchronization (producer: ace-stack-backend; consumer: ace-commands-api docs-sync worker). Created via script **scripts/create-docs-sync-queue.sh** or optionally via Terraform module **terraform-library/docs-sync-queue**. Queue URL is configured as `QUEUE_DOCS_SYNC_URL` in backend and docs-sync worker. For the full picture of KB resources (S3, Bedrock, IAM, S3 Vectors, sync flow), see [knowledge-base-and-resources.md](./knowledge-base-and-resources.md).
- **Other queues**: Any other SQS queues (e.g. for async jobs) are defined in Terraform or created via scripts; document their purpose and URL env var in the service docs.

---

## Knowledge Base (AWS Bedrock) and related resources

The **Knowledge Base** feature uses several AWS resources that are **provisioned by ace-stack-backend** (not by Terraform), plus the DocsSync SQS queue:

| Resource type | Purpose |
|---------------|---------|
| **Bedrock Knowledge Base** | Vector KB with embedding model (e.g. amazon.titan-embed-text-v2), linked to S3 Vectors storage. |
| **Bedrock Data Source** | S3 data source pointing to the project docs bucket; hierarchical chunking. |
| **S3 (docs bucket)** | One bucket per project (naming pattern `<prefix>-kb`) for synced markdown; versioning and public access blocked. |
| **S3 Vectors** | Vector bucket and index for KB embeddings (optional; created by backend). |
| **IAM** | Role and policies for Bedrock (S3, Bedrock, S3 Vectors); created by backend. |
| **SQS (DocsSync)** | FIFO queue for sync jobs; created by script or Terraform. See [knowledge-base-and-resources.md](./knowledge-base-and-resources.md). |

For provisioning flow, sync flow (clone repo → S3 sync → ingestion), docs-sync worker, and env vars, see **[knowledge-base-and-resources.md](./knowledge-base-and-resources.md)**.

---

## Mandatory resource tags

Every AWS resource that belongs to ACE must have:

| Tag key    | Tag value   | Purpose |
|-----------|-------------|---------|
| **Project** | **ACE**   | Identify project for cost allocation and filtering. |
| **Environment** | **<ENV>-ACE** | e.g. dev-ACE, staging-ACE, prod-ACE. |

Use these in Terraform `tags` blocks and when creating resources manually. See [../rules/infrastructure-rules.md](../rules/infrastructure-rules.md).

---

## IAM

- **EKS cluster role**: IAM role for the EKS control plane (AssumeRole by eks.amazonaws.com). Created by Terraform (terraform-library/eks).
- **EKS node role**: IAM role for worker nodes (AssumeRole by ec2.amazonaws.com). Policies: AmazonEKSWorkerNodePolicy, AmazonEKS_CNI_Policy, AmazonEC2ContainerRegistryReadOnly.
- **ALB controller**: IAM policy and service account for the AWS Load Balancer Controller (see ace-infra README and eks-general-configs for install steps). Allows creating and managing ALBs and target groups.
- **Route53**: IAM policies for Jira integration (Route53 updates) live in **ace-infra/iam/** (e.g. route53-ace-jira-policy.json). Applied via scripts.

Do not document IAM role ARNs or account IDs in this manual.

---

## Summary table

| Resource type   | Region   | Naming / pattern |
|----------------|----------|-------------------|
| EKS            | us-east-1 | <environment>-ace-eks |
| ECR            | us-east-1 | ace/<service> |
| VPC            | us-east-1 | <environment>-environment (Terraform name) |
| RDS            | us-east-1 | <environment>-env-db |
| DocumentDB     | us-east-1 | <environment>-docdb-cluster |
| Redis          | us-east-1 | <environment>-ace-redis |
| EFS            | us-east-1 | <environment>-efs (creation token) |
| Secrets Manager | us-east-1 | ace/<env>/<service>-secrets |
| Tags           | —        | Project=ACE, Environment=<ENV>-ACE |

---

## Links

- **Terraform (where resources are defined)**: [terraform-modules.md](./terraform-modules.md)
- **Kubernetes (EKS usage)**: [kubernetes-and-deployment.md](./kubernetes-and-deployment.md)
- **Infrastructure rules**: [../rules/infrastructure-rules.md](../rules/infrastructure-rules.md)
- **Env vars and secrets (source of vars)**: [../environments/env-vars-and-secrets.md](../environments/env-vars-and-secrets.md)

---

*Last updated: March 2025*
