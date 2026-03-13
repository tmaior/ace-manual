# Scripts and Automation

This document describes the **scripts** and automation in ace-infra: what each script does, when to use it, and where it lives. It does not document secret values or credentials.

---

## Scripts location

All scripts below live under **ace-infra/scripts/** unless otherwise noted.

---

## SQS

### create-docs-sync-queue.sh

- **Purpose**: Creates the **SQS FIFO queue** used for Knowledge Base document synchronization.
- **Producer**: ace-stack-backend (sends script + project_id via `QUEUE_DOCS_SYNC_URL`).
- **Consumer**: ace-commands-api docs-sync worker.
- **Usage**: Run once per environment (or per account/region) where the queue is needed. Configure the output **queue_url** as `QUEUE_DOCS_SYNC_URL` in backend and docs-sync worker env vars.
- **Terraform alternative**: Module **terraform-library/docs-sync-queue** can create the queue; when enabled in cluster/terraform/main.tf, the script may be retired for that environment. See [terraform-modules.md](./terraform-modules.md).

---

## Route53

### update-route53-demo.sh

- **Purpose**: Updates **Route53** DNS records for the **demo** environment (e.g. CNAME/A/ALIAS to the demo ALB or ingress).
- **When**: After creating or changing demo ingress/hostnames, or when onboarding demo to a new domain/ALB.
- **Prerequisites**: AWS CLI configured with permissions for Route53; correct hosted zone and record names. Do not commit credentials or expose zone IDs in docs if sensitive.

### apply-route53-jira-policy.sh

- **Purpose**: Applies the **IAM policy** required for ACE–Jira integration to update Route53 (e.g. for Jira app callback URLs or DNS validation).
- **When**: One-time or when the policy document (iam/route53-ace-jira-policy.json) changes.
- **Related**: **iam/route53-ace-jira-policy.json** defines the policy. See ace-jira-integration docs.

### apply-route53-policy.sh

- **Purpose**: Generic Route53 policy application (if used). Check script content for exact scope.
- **When**: As needed for other Route53-related IAM setup.

### test-route53-update.sh

- **Purpose**: Tests that Route53 updates (e.g. after applying policy or changing records) work as expected.
- **When**: After running update or apply scripts; use in CI or manually.

---

## Database and integration setup

### jira-integration-db-setup.sh

- **Purpose**: **Database setup** for the ACE–Jira integration (tables, schema, or seed data).
- **When**: Once per environment where Jira integration is deployed, or after schema changes.
- **Location**: ace-infra/scripts/ (or under ace-jira-integration if moved). Document any required env vars (e.g. DB host, user) in the service docs; do not document passwords.

---

## Validation and cleanup

### validate-demo-environment.sh

- **Purpose**: Validates the **demo** environment (e.g. ingress, services, DNS, or connectivity checks).
- **When**: After deploying or changing demo; can be run in CI or manually.

### delete-kb-resources.sh

- **Purpose**: **Cleanup** of Knowledge Base–related resources (e.g. S3, queues, or temporary data). Use with care; confirm scope before running.
- **When**: When decommissioning or resetting KB-related resources in an environment.

### test-database-connectivity.sh

- **Purpose**: Tests **database connectivity** (e.g. from a pod or runner to RDS/DocumentDB). Useful for troubleshooting and after network or security group changes.
- **When**: After infra changes or when diagnosing connection failures. Do not log or document connection strings.

### test-performance-basic.sh

- **Purpose**: Basic **performance** or load checks (e.g. HTTP or DB). Scope is defined in the script.
- **When**: Ad-hoc or in CI for smoke tests.

---

## DevOps and secrets

### devops_setup.sh

- **Purpose**: General **DevOps/setup** automation (e.g. install tools, configure env, or one-time setup). See script content for exact actions.
- **When**: Onboarding or when documented in runbooks.

### get_secrets.sh

- **Purpose**: **Helper** to fetch or list secrets (e.g. from AWS Secrets Manager) for local or script use. Use only in secure contexts; never log or commit secret values.
- **When**: When you need to populate local .env or verify that a secret exists (key names only). Do not document script output or secret values.

---

## Usage guidelines

- **Execution**: Run scripts from the ace-infra repo root or from the directory indicated in the script. Ensure AWS CLI (and optionally kubectl) are configured for the target account and region.
- **Permissions**: Scripts may require IAM permissions for SQS, Route53, Secrets Manager, or RDS. Use the least privilege; do not document credentials or account IDs in this manual.
- **Idempotency**: Prefer scripts that can be run multiple times safely (e.g. create queue only if not exists). Document any destructive behavior (e.g. delete-kb-resources.sh) clearly.
- **Updates**: When you add or change a script, update this document and the ace-infra **docs/** as needed. Keep [ace-infra-repository.md](./ace-infra-repository.md) in sync if you add a new script category or path.

---

## Links

- **ace-infra layout**: [ace-infra-repository.md](./ace-infra-repository.md)
- **Terraform (SQS module)**: [terraform-modules.md](./terraform-modules.md)
- **Environments**: [../environments/](../environments/)
- **Creating a new environment**: [../environments/creating-a-new-environment.md](../environments/creating-a-new-environment.md)

---

*Last updated: March 2025*
