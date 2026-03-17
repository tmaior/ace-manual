# ace-infra – Scripts

This document lists the main scripts under **scripts/** in the ace-infra repository and their purpose, as reflected in the codebase. Run each script from the repo root or with the correct working directory and prerequisites (AWS CLI, kubectl, etc.) as required.

---

## Script list

| Script | Purpose |
|--------|---------|
| **create-docs-sync-queue.sh** | Creates the SQS FIFO queue used for docs sync (Knowledge Base). Use when the queue is not managed by Terraform (docs_sync_queue module is commented out in cluster Terraform). |
| **update-route53-demo.sh** | Updates Route53 records for the demo environment (e.g. CNAMEs for demo hosts). |
| **apply-route53-policy.sh** | Applies the general Route53 IAM policy (demo/other). |
| **apply-route53-jira-policy.sh** | Creates/updates the Route53 IAM policy for Jira integration hostnames (e.g. dev-jira, stg-jira, jira.ace.ezops.cloud) and attaches it to the EKS roles (e.g. ace-dev-eks-role, ace-prod-eks-role) so CI/CD can upsert CNAMEs. |
| **jira-integration-db-setup.sh** | Creates and configures the database (and optionally schema) used by ace-jira-integration in each environment. Run against the target RDS (or dev DB) with appropriate connection details. |
| **validate-demo-environment.sh** | Validates the demo environment setup (resources, connectivity, etc.). |
| **delete-kb-resources.sh** | Cleanup of Knowledge Base–related resources (use with care). |
| **get_secrets.sh** | Helper to fetch secrets (structure only; do not log or document secret values). |
| **devops_setup.sh** | DevOps/setup automation; see script for scope and usage. |
| **test-database-connectivity.sh** | Tests database connectivity (e.g. to RDS). |
| **test-performance-basic.sh** | Basic performance tests. |
| **test-route53-update.sh** | Tests Route53 update flow. |

---

## Usage notes

- Scripts that modify AWS resources (Route53, IAM, SQS) require appropriate AWS credentials and permissions.
- **apply-route53-jira-policy.sh** and **jira-integration-db-setup.sh** are described in the ace-infra PR template for the Jira integration feature; run them when setting up or updating Jira integration in an environment.
- For detailed usage, parameters, and prerequisites, open the script or see [../../infrastructure/scripts-and-automation.md](../../infrastructure/scripts-and-automation.md).

---

## Related

- [Repository structure](./repository-structure.md) – where **scripts/** sits in the repo.
- [../../infrastructure/scripts-and-automation.md](../../infrastructure/scripts-and-automation.md) – when and how to use each script in the broader infra workflow.
