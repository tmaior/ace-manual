# Terraform Modules

This document describes the **Terraform** setup in ace-infra: the cluster root configuration, **terraform-library** modules, workspaces, and per-environment tfvars. It does not include sensitive values (no passwords, no account IDs in examples).

---

## Where Terraform lives

| Path | Purpose |
|------|---------|
| **cluster/terraform/** | Root module: invokes terraform-library modules to create VPC, EKS, RDS, DocumentDB, EFS, Redis, and EKS add-ons. |
| **cluster/environments/** | Per-environment tfvars: `development/terraform.tfvars`, `production/terraform.tfvars`. |
| **terraform-library/** | Reusable modules: vpc, eks, eks-general-configs, rds, documentDB, efs, redis, docs-sync-queue, codepipeline. |
| **demo-env/** | Optional separate root for demo (e.g. demo-env-eks); has its own main.tf, variables, backend, and config-vars. |

---

## Cluster root (cluster/terraform)

**main.tf** wires the environment and calls the library modules:

- **module "vpc"**: VPC, public/private subnets, IGW, NAT, route tables. Subnet tags for ELB (`kubernetes.io/role/elb`, `kubernetes.io/role/internal-elb`). Inputs: name, cidr_block, public_subnets, private_subnets, availability_zones, tags.
- **module "eks"**: EKS cluster, node group, IAM roles (cluster + node), launch template. Inputs: cluster_name (`<environment>-ace-eks`), private_subnets, instance_types, desired/max/min capacity, kubernetes_version, use_spot, tags.
- **module "general_configs"** (eks-general-configs): Metrics server (Helm), EBS CSI driver, EFS CSI driver, Kubernetes namespaces from variable, optional New Relic, ALB controller. Inputs: cluster_id, cluster_endpoint, cluster_name, region, metrics_server, namespaces, ebs_csi_driver, efs_csi_driver, alb_controller, alb_controller_vpc_id.
- **module "rds"**: RDS PostgreSQL. Inputs: name (`<environment>-env-db`), engine_type, instance_class, engine_version, subnet_ids, vpc_id, vpc_cidr_block, backup and storage options, tags.
- **module "documentdb"**: DocumentDB cluster. Inputs: cluster_name, subnet_ids, instance_class, instance_count, master_username, vpc_id, allowed_cidr_blocks, backup options, tags.
- **module "efs"**: EFS file system. Inputs: efs_creation_token, efs_subnet_ids, efs_vpc_id, efs_allowed_cidr_blocks, performance mode, encryption, transition_to_ia, tags.
- **module "redis"**: ElastiCache Redis. Inputs: project_name, subnet_ids, node_type, port, vpc_id, region, tags.

**Docs Sync SQS**: The queue is created via script (**scripts/create-docs-sync-queue.sh**). A Terraform module **terraform-library/docs-sync-queue** exists; it can be enabled in main.tf when queue creation is moved to Terraform (uncomment the module block and remove or adjust the script).

**variables.tf**: Defines inputs (aws_region, environment, vpc_cidr_block, public_subnets, private_subnets, availability_zones, eks_*, namespaces, metrics_server, ebs_csi_driver, efs_csi_driver, alb_controller, rds_*, etc.). Defaults are overridden by tfvars.

**outputs.tf**: Exposes outputs (e.g. VPC ID, subnet IDs, EKS cluster name/endpoint, RDS endpoint) for use by other configs or scripts.

**backend.tf** / **providers.tf**: Backend configuration (e.g. S3, DynamoDB for state locking) and required providers (aws, kubernetes, helm). Do not document backend bucket names or keys if they are sensitive.

---

## Workspaces and tfvars

- **Workspaces**: Use Terraform workspaces to separate state (e.g. `development`, `production`). Create/select before plan/apply:
  - `terraform workspace select development` or `terraform workspace new development`
  - `terraform workspace select production` or `terraform workspace new production`
- **Tfvars**: Apply with the matching tfvars file:
  - Development: `terraform plan -var-file=../environments/development/terraform.tfvars`
  - Production: `terraform plan -var-file=../environments/production/terraform.tfvars`
- **Sensitive variables**: RDS master username and any passwords are in tfvars or in a secret store; never commit them. Use `terraform.tfvars` in .gitignore if it contains secrets, or use a backend that supports sensitive variables.

---

## terraform-library modules (summary)

| Module | Main resources |
|--------|----------------|
| **vpc** | aws_vpc, aws_subnet (public/private), aws_internet_gateway, aws_nat_gateway, aws_eip, route tables and associations. |
| **eks** | aws_eks_cluster, aws_eks_node_group, aws_iam_role (cluster + node), launch template, policy attachments. |
| **eks-general-configs** | helm_release (metrics-server, ebs-csi-driver, efs-csi-driver), kubernetes_namespace(s), optional New Relic, ALB controller setup. |
| **rds** | aws_db_instance (PostgreSQL), subnet group, security group. |
| **documentDB** | aws_docdb_cluster, aws_docdb_cluster_instance, subnet group, security group. |
| **efs** | aws_efs_file_system, aws_efs_mount_target, security group. |
| **redis** | aws_elasticache_replication_group (or cluster), subnet group, security group. |
| **docs-sync-queue** | aws_sqs_queue (FIFO). |
| **codepipeline** | AWS CodePipeline resources (if used). |

Each module has **variables.tf**, **outputs.tf**, and **main.tf**. Read the module source when changing or adding resources.

---

## Adding or changing Terraform

1. **New resource in existing module**: Edit the module in **terraform-library/<module>** and expose variables/outputs as needed. Update **cluster/terraform/main.tf** if you add or change module inputs.
2. **New module**: Create **terraform-library/<name>** with main.tf, variables.tf, outputs.tf. Wire it in **cluster/terraform/main.tf** and add variables to cluster/terraform/variables.tf and to **cluster/environments/*.tfvars**.
3. **New environment**: Add **cluster/environments/<env>/terraform.tfvars** and use a Terraform workspace for that environment. Document the new environment in [../environments/](../environments/) and in [creating-a-new-environment.md](../environments/creating-a-new-environment.md).
4. **Before PR**: Run `terraform fmt` and `terraform validate` in the root and in any modified module. See [../rules/pr-rules.md](../rules/pr-rules.md) (ace-infra).

---

## demo-env

**demo-env/** is a separate Terraform root (optional) for a dedicated demo cluster (e.g. demo-env-eks in another region or account). It has:

- main.tf, variables.tf, outputs.tf, providers.tf, backend.tf
- config-vars (e.g. demo.tfvars)

Use it only when the project uses a separate demo cluster; the main **demo** namespace for ACE is on **development-ace-eks** (see [../environments/demo.md](../environments/demo.md)).

---

## Links

- **ace-infra layout**: [ace-infra-repository.md](./ace-infra-repository.md)
- **AWS resources (what Terraform creates)**: [aws-resources.md](./aws-resources.md)
- **Infrastructure rules**: [../rules/infrastructure-rules.md](../rules/infrastructure-rules.md)
- **Creating a new environment**: [../environments/creating-a-new-environment.md](../environments/creating-a-new-environment.md)

