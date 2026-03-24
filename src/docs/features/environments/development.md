# Development Environment (EKS)

This document describes the **development** environment where ACE runs on EKS for integration and daily development.

---

## Purpose

- **Integration and daily dev** — run ACE services in the cloud for testing, demos, and integration without running everything locally.
- **Namespace isolation** — development workloads run in namespace **dev**; staging uses **stg** and demo uses **demo** in the same cluster.

---

## Cluster and namespace

| Item | Value |
|------|--------|
| **Cluster** | **development-ace-eks** |
| **Region** | us-east-1 |
| **Namespace** | **dev** (main namespace for development workloads) |
| **Other namespaces in cluster** | stg (staging), demo (demo), ace-system (shared/system) |

**kubectl context** (example): `arn:aws:eks:us-east-1:<account>:cluster/development-ace-eks`

```bash
# List namespaces
kubectl get namespaces

# Work in dev namespace
kubectl config set-context --current --namespace=dev
kubectl get pods
```

---

## How to deploy

- **CI/CD**: **GitHub Actions** — build on push, push images to **ECR** (us-east-1), deploy to EKS using manifests from **ace-infra**.
- **Branch**: Deployment to **dev** is triggered from branch **development** in all repositories. See each repo’s workflow (e.g. `eks-deploy.dev.yaml`) for the exact trigger and steps.
- **Manifests**: K8s manifests live under **ace-infra** in per-service folders; namespace in manifests should be **dev** for development workloads.

Do not push directly to protected branches; use the defined pipeline and branch strategy. See [../rules/gitflow-rules.md](../rules/gitflow-rules.md) and [../rules/infrastructure-rules.md](../rules/infrastructure-rules.md).

---

## Access and URLs

- **Ingress/ALB**: Services in **dev** are exposed via the cluster’s ALB Ingress Controller. Base URLs and hostnames are defined in Ingress resources and possibly in Route53 (see ace-infra).
- **URL convention**: Development uses the **`dev-`** prefix for hostnames (e.g. `dev-dashboard.ace.ezops.cloud`, `dev-api-ace.ace.ezops.cloud`). Production URLs do not use this prefix.
- **Internal access**: Within the cluster, use K8s service DNS (e.g. `http://ace-db-gateway.dev.svc.cluster.local`). External access uses the public/private URLs defined for the environment.
- Document the actual dev URLs in ace-infra or in a runbook; they are environment-specific.

---

## Secrets

- **Path pattern**: **`ace/dev/<service>-secrets`** in **AWS Secrets Manager** (e.g. `ace/dev/ace-stack-backend-secrets`, `ace/dev/ace-db-gateway-secrets`).
- These secrets are **environment variables** for the application at runtime. They are injected into pods (e.g. via deployment pipeline or K8s external secrets); see [../rules/infrastructure-rules.md](../rules/infrastructure-rules.md).
- **CI/CD secrets** (deploy credentials, GitHub tokens) live in **GitHub Environments**, not in these paths.

---

## Links

- **architecture/deployment.md** — [../architecture/deployment.md](../architecture/deployment.md)
- **ace-infra** — Terraform and K8s manifests; per-service folders for dev namespace.
- **env-vars-and-secrets** — [env-vars-and-secrets.md](./env-vars-and-secrets.md)
- **troubleshooting** — [troubleshooting.md](./troubleshooting.md)
