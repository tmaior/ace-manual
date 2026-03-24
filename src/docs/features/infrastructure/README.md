# Infrastructure

This directory is the **single place** for ACE infrastructure documentation: the **ace-infra** repository, AWS resources, Terraform, Kubernetes (EKS), and deployment automation. It does **not** contain sensitive data (no account IDs, no secret values, no credentials).

---

## Purpose

- **ace-infra**: Repository layout, service folders, Terraform and Kubernetes manifests, scripts.
- **AWS**: Region, EKS clusters, ECR, VPC, RDS, DocumentDB, Redis, EFS, Secrets Manager **path pattern** and tags.
- **Terraform**: Cluster and terraform-library modules, workspaces, per-environment tfvars.
- **Kubernetes**: Namespaces, ALB Ingress, manifest naming and structure, CI/CD flow.

Use this directory when you need to add a new service to the cluster, create a new environment, change Terraform or K8s manifests, or understand where AWS resources are defined and how they are used.

---

## What you will find here

| Document | Content |
|----------|---------|
| [ace-infra-repository.md](./ace-infra-repository.md) | ace-infra repo structure, service folders, cluster, terraform-library, scripts, docs. |
| [aws-resources.md](./aws-resources.md) | AWS region, EKS, ECR, VPC, RDS, DocumentDB, Redis, EFS, Secrets Manager pattern, tags. |
| [kubernetes-and-deployment.md](./kubernetes-and-deployment.md) | EKS clusters and namespaces, ALB, manifest patterns, CI/CD (GitHub Actions, ECR, deploy). |
| [terraform-modules.md](./terraform-modules.md) | Terraform modules (VPC, EKS, RDS, DocumentDB, EFS, Redis, eks-general-configs), workspaces, tfvars. |
| [scripts-and-automation.md](./scripts-and-automation.md) | Scripts for SQS, Route53, DB setup, validation, cleanup; when and where to use them. |
| [knowledge-base-and-resources.md](./knowledge-base-and-resources.md) | Knowledge Base (AWS Bedrock): S3, IAM, Bedrock KB/DataSource, S3 Vectors, DocsSync queue, docs-sync worker, sync flow, DB Gateway, webhook. |
| [eks-access-and-iam.md](./eks-access-and-iam.md) | How to access EKS (kubeconfig), how to allow or create a user (IAM, aws-auth, RBAC). |
| [databases-credentials-and-admin-tools.md](./databases-credentials-and-admin-tools.md) | Where to find RDS/DocumentDB/Redis credentials; pgAdmin, Redis Insight, Mongo Express (access and install). |
| [infrastructure-diagrams.md](./infrastructure-diagrams.md) | VPC, public/private subnets, what runs where, request flow, Mermaid diagrams. |
| [route53-dns.md](./route53-dns.md) | Route53, DNS, naming convention, scripts (demo, Jira), how to create/update records. |

---

## How to use

- **Adding a new service**: Read [ace-infra-repository.md](./ace-infra-repository.md) for folder layout, then [kubernetes-and-deployment.md](./kubernetes-and-deployment.md) for manifest patterns. Follow [../rules/infrastructure-rules.md](../rules/infrastructure-rules.md) (tags, secrets path).
- **Changing Terraform**: Read [terraform-modules.md](./terraform-modules.md) and the relevant module in ace-infra. Run `terraform fmt` and `terraform validate` before PR.
- **Understanding AWS**: Read [aws-resources.md](./aws-resources.md). Secrets and env vars are documented in [../environments/env-vars-and-secrets.md](../environments/env-vars-and-secrets.md) (no values).
- **Knowledge Base (Bedrock, S3, SQS, worker)**: Read [knowledge-base-and-resources.md](./knowledge-base-and-resources.md).
- **EKS access or add a user**: Read [eks-access-and-iam.md](./eks-access-and-iam.md).
- **RDS/Redis/DocumentDB credentials or pgAdmin/Redis Insight/Mongo Express**: Read [databases-credentials-and-admin-tools.md](./databases-credentials-and-admin-tools.md).
- **Diagrams (VPC, subnets, communications)**: Read [infrastructure-diagrams.md](./infrastructure-diagrams.md).
- **Route53 and DNS**: Read [route53-dns.md](./route53-dns.md).
- **Creating a new environment**: Use [../environments/creating-a-new-environment.md](../environments/creating-a-new-environment.md); it references infra (namespace, secrets path, CI/CD, manifests).

---

## Relation to other docs

- **Rules**: [../rules/infrastructure-rules.md](../rules/infrastructure-rules.md) — region, tags, secrets path, CI/CD, ace-infra structure.
- **Environments**: [../environments/](../environments/) — local, dev, stg, demo, prod; env vars and secrets **source** (path pattern only).
- **Architecture**: [../architecture/deployment.md](../architecture/deployment.md) — high-level where ACE runs (EKS, AWS).

