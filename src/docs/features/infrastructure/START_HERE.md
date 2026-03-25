# Start Here – Infrastructure

This directory documents ACE infrastructure: the **ace-infra** repository, AWS resources, Terraform, Kubernetes, and deployment. Below is each file with a short description.

---

## Files

**[README.md](./README.md)**  
Overview of this directory: purpose, what it covers (ace-infra, AWS, Terraform, K8s), and how to use it. Links to rules and environment docs.

**[ace-infra-repository.md](./ace-infra-repository.md)**  
Structure and layout of the **ace-infra** repository: service folders, cluster Terraform, terraform-library modules, helm-charts, docs, and local/demo configs. Use it to find where to add a new service or change infra.

**[aws-resources.md](./aws-resources.md)**  
AWS resources used by ACE: region (us-east-1), EKS clusters, ECR, VPC and networking, RDS, DocumentDB, ElastiCache (Redis), EFS, Secrets Manager path pattern, and mandatory tags. No sensitive data (no account IDs, no secret values).

**[kubernetes-and-deployment.md](./kubernetes-and-deployment.md)**  
Kubernetes: cluster names, namespaces (dev, stg, demo, prod), ALB Ingress Controller, manifest patterns (Deployment, Service, Ingress), and CI/CD flow (GitHub Actions, ECR, deploy to EKS).

**[terraform-modules.md](./terraform-modules.md)**  
Terraform in ace-infra: cluster (VPC, EKS, RDS, DocumentDB, EFS, Redis), terraform-library modules, workspace and tfvars per environment (development, production), and how to add or change modules.

**[scripts-and-automation.md](./scripts-and-automation.md)**  
Scripts in ace-infra: SQS queue creation (docs-sync), Route53 updates (demo, Jira), DB setup (Jira integration), validation and cleanup scripts. When to use each and where they live.

**[knowledge-base-and-resources.md](./knowledge-base-and-resources.md)**  
Knowledge Base (AWS Bedrock) and all associated resources: provisioning (S3 docs bucket, IAM, Bedrock KB and Data Source, optional S3 Vectors), DocsSync SQS queue, docs-sync worker in commands-api, sync flow (clone repo, S3 sync, ingestion), DB Gateway persistence, and GitHub webhook. Use it to understand or operate KB-related infra.

**[eks-access-and-iam.md](./eks-access-and-iam.md)**  
How to access EKS clusters (kubeconfig, `aws eks update-kubeconfig`), how cluster authentication works (IAM + aws-auth ConfigMap + RBAC), how to allow a user or create a new user for cluster access, and where aws-auth lives.

**[databases-credentials-and-admin-tools.md](./databases-credentials-and-admin-tools.md)**  
Where to find RDS, DocumentDB, and Redis credentials (Terraform outputs, Secrets Manager); how to access RDS, Redis, and DocumentDB with pgAdmin, Redis Insight, and Mongo Express (port-forward, connection details); how to install these admin tools in the cluster if missing.

**[infrastructure-diagrams.md](./infrastructure-diagrams.md)**  
Diagrams of infrastructure: VPC, public vs private subnets, what runs where (ALB, NAT, EKS, RDS, DocumentDB, Redis, EFS), request flow, and network layout. Mermaid diagrams included.

**[monitoring-stack.md](./monitoring-stack.md)**  
Where the ACE monitoring stack is documented (Prometheus/Grafana/Alertmanager + Loki/Promtail), and how to deploy and access it (via `ace-infra/monitoring/` scripts and guides).

**[route53-dns.md](./route53-dns.md)**  
Route53, DNS, and apontamentos: hosted zone, naming convention (e.g. <env>-<service>.ace.ezops.cloud), how records are created/updated, update-route53-demo.sh script, IAM policy for Jira integration, and how to create or update records manually.

---

## Subdirectories

This directory has no subdirectories.

---

*For infrastructure **rules** (region, tags, secrets path, CI/CD), see [../rules/infrastructure-rules.md](../rules/infrastructure-rules.md). For **environment** setup and deployment targets, see [../environments/](../environments/).*
