# ace-infra

Infrastructure as code for the ACE system: Terraform, Kubernetes, AWS.

## Purpose

- Single place for Terraform (cluster, VPC, EKS, RDS, DocumentDB, EFS, Redis), Kubernetes manifests per service, Dockerfiles, Helm charts, and automation scripts.
- All ACE AWS and EKS resources follow the patterns and modules defined in ace-infra.

## Documentation

- **This folder**: Central reference; full infra docs live under **infrastructure/**.
- **Infrastructure**: [../../infrastructure/](../../infrastructure/) – [ace-infra-repository.md](../../infrastructure/ace-infra-repository.md), [terraform-modules.md](../../infrastructure/terraform-modules.md), [kubernetes-and-deployment.md](../../infrastructure/kubernetes-and-deployment.md), [aws-resources.md](../../infrastructure/aws-resources.md).

## Related

- [Architecture service catalog](../../architecture/service-catalog.md)
- [Infrastructure rules](../../rules/infrastructure-rules.md)
