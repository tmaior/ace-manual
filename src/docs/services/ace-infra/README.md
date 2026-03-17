# ace-infra

Infrastructure as code for the ACE system: Terraform, Kubernetes, AWS.

## Purpose

- Single place for Terraform (cluster, VPC, EKS, RDS, DocumentDB, EFS, Redis), Kubernetes manifests per service, Dockerfiles, Helm charts, IAM policies, and automation scripts.
- All ACE AWS and EKS resources follow the patterns and modules defined in ace-infra.

## What you will find here

| Document | Description |
|----------|-------------|
| [overview.md](./overview.md) | What ace-infra is, purpose, high-level architecture, main components, and what you can do. |
| [repository-structure.md](./repository-structure.md) | Top-level layout, terraform-library modules, and ace-* service folders. |
| [terraform-and-environments.md](./terraform-and-environments.md) | Cluster Terraform, variables, environments (tfvars), workspaces, outputs. |
| [kubernetes-manifests-and-services.md](./kubernetes-manifests-and-services.md) | K8s manifest pattern and example; adding or changing a service. |
| [scripts.md](./scripts.md) | Scripts under scripts/ and their purpose. |

## How to use

- **New to ace-infra**: Read [START_HERE.md](./START_HERE.md), then [overview.md](./overview.md).
- **Working with Terraform**: Use [terraform-and-environments.md](./terraform-and-environments.md) and [../../infrastructure/terraform-modules.md](../../infrastructure/terraform-modules.md).
- **Adding or changing a service**: Use [kubernetes-manifests-and-services.md](./kubernetes-manifests-and-services.md) and [../../infrastructure/ace-infra-repository.md](../../infrastructure/ace-infra-repository.md).
- **Running scripts**: Use [scripts.md](./scripts.md) and [../../infrastructure/scripts-and-automation.md](../../infrastructure/scripts-and-automation.md).

## Related

- **Infrastructure (cross-repo)**: [../../infrastructure/](../../infrastructure/) – full infra docs (ace-infra layout, Terraform modules, Kubernetes, AWS, scripts).
- [Architecture service catalog](../../architecture/service-catalog.md)
- [Infrastructure rules](../../rules/infrastructure-rules.md)
