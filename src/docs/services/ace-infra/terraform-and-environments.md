# ace-infra – Terraform and environments

This document describes the Terraform setup in **cluster/terraform/** and **cluster/environments/** as defined in the ace-infra codebase.

---

## Cluster Terraform root (cluster/terraform/)

**main.tf** composes modules from **terraform-library/**:

| Module | Source | Role |
|--------|--------|------|
| **vpc** | terraform-library/vpc | VPC, public/private subnets, AZs, subnet tags for ELB. |
| **eks** | terraform-library/eks | EKS cluster, node group (spot optional), IAM roles, launch template. |
| **general_configs** | terraform-library/eks-general-configs | Metrics server, EBS/EFS CSI drivers, namespaces, ALB controller. |
| **rds** | terraform-library/rds | PostgreSQL RDS (single instance in current config). |
| **documentdb** | terraform-library/documentDB | DocumentDB cluster. |
| **efs** | terraform-library/efs | EFS file system. |
| **redis** | terraform-library/redis | ElastiCache Redis. |

The **docs_sync_queue** module is commented out; the queue is created manually via **scripts/create-docs-sync-queue.sh**.

---

## Variables (variables.tf)

Main variables used by the cluster (with typical or default values from code):

| Variable | Description | Example / default |
|----------|-------------|--------------------|
| aws_region | AWS region | us-east-1 |
| environment | Environment name | production (override per env) |
| vpc_cidr_block | VPC CIDR | 10.0.0.0/16 |
| public_subnets, private_subnets | Subnet CIDRs | e.g. 10.0.1.0/24, 10.0.3.0/24 |
| availability_zones | AZs for subnets | us-east-1a, us-east-1b |
| eks_desired_capacity, eks_min_capacity, eks_max_capacity | Node group sizing | 3, 2, 4 (dev example) |
| eks_instance_types | EC2 types for nodes | list of instance types |
| eks_kubernetes_version | EKS version | 1.31 |
| namespaces | K8s namespaces to create | ["dev"] or ["dev", "stg", "prod"] |
| metrics_server, ebs_csi_driver, efs_csi_driver, alb_controller | EKS add-ons (bool) | true |
| rds_master_username, rds_allocated_storage, rds_instance_class, rds_backup_retention | RDS settings | e.g. dev_ace_admin, 20, db.t3.micro, 7 |

---

## Environments (cluster/environments/)

Each environment has a subfolder with **terraform.tfvars** (and optionally other files). Example from **development/terraform.tfvars**:

- **aws_region**: us-east-1  
- **environment**: development  
- **vpc_cidr_block**, **public_subnets**, **private_subnets**, **availability_zones**  
- **eks_desired_capacity**, **eks_max_capacity**, **eks_min_capacity**, **eks_instance_types**, **eks_kubernetes_version**  
- **rds_master_username**, **rds_allocated_storage**, **rds_instance_class**, **rds_backup_retention**  
- **namespaces**: e.g. ["dev"]

Terraform workspaces (e.g. `development`, `production`) are used; select the workspace and apply with the matching tfvars:

```bash
cd cluster/terraform
terraform workspace select development
terraform plan -var-file=../environments/development/terraform.tfvars
terraform apply -var-file=../environments/development/terraform.tfvars
```

---

## Outputs (outputs.tf)

Outputs expose IDs and endpoints for use by other automation or documentation:

- **current_workspace**, **vpc_id**, **public_subnet_ids**, **private_subnet_ids**
- **eks_cluster_name**, **eks_node_group_role**
- **metrics_server_status**, **alb_controller_status**, **ebs_csi_driver_status**
- **rds_endpoint**, **rds_instance_arn**, **rds_password**
- **document_db_cluster_endpoint**, **document_db_cluster_arn**, **document_db_security_group_id**, **document_db_secrets_manager_arn**
- **efs_id**, **efs_dns_name**, **efs_mount_targets**
- **redis_endpoint**, **redis_port**

DocsSync queue URL/ARN are commented out (queue created via script).

---

## Conventions

- Run **terraform fmt** and **terraform validate** before committing Terraform changes (per ACE project rules).
- Do not store secrets in tfvars; use Terraform variables or external secret stores (e.g. RDS password from module output or Secrets Manager).
- For full module details and backend configuration, see [../../infrastructure/terraform-modules.md](../../infrastructure/terraform-modules.md).
