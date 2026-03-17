# Start Here – ace-infra

**ace-infra** is the infrastructure-as-code repository for ACE: Terraform (VPC, EKS, RDS, DocumentDB, EFS, Redis), Kubernetes manifests per service, Helm charts, IAM policies, and automation scripts. All ACE AWS and EKS resources are defined or referenced here.

## Contents

- **[README.md](./README.md)** – Overview of this folder, purpose, and how to use the documentation.
- **[index.md](./index.md)** – Simple list of contents of this directory.
- **[overview.md](./overview.md)** – What ace-infra is, purpose, high-level architecture, main components, and what you can do with the repo.
- **[repository-structure.md](./repository-structure.md)** – Top-level layout, terraform-library modules, and ace-* service folders as reflected in the codebase.
- **[terraform-and-environments.md](./terraform-and-environments.md)** – Cluster Terraform (main.tf, variables, outputs), cluster/environments tfvars, workspaces, and conventions.
- **[kubernetes-manifests-and-services.md](./kubernetes-manifests-and-services.md)** – Manifest pattern for ace-* folders (Deployment, Service, Ingress) and example (ace-jira-integration); namespaces and adding a service.
- **[scripts.md](./scripts.md)** – List and purpose of scripts under scripts/ (SQS, Route53, DB setup, validation, tests).

## Where to find more

- **Repository**: `ace-infra/` (sibling to ace-manual). Code, Terraform, K8s manifests, and repo-level docs (e.g. `docs/eks-pods-inventory.md`).
- **Infrastructure (cross-repo)**: [../../infrastructure/](../../infrastructure/) – [ace-infra-repository.md](../../infrastructure/ace-infra-repository.md), [terraform-modules.md](../../infrastructure/terraform-modules.md), [kubernetes-and-deployment.md](../../infrastructure/kubernetes-and-deployment.md), [scripts-and-automation.md](../../infrastructure/scripts-and-automation.md).
- **Architecture**: [../../architecture/service-catalog.md](../../architecture/service-catalog.md). **Rules**: [../../rules/infrastructure-rules.md](../../rules/infrastructure-rules.md).
