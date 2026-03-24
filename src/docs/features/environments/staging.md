# Staging Environment (EKS)

This document describes the **staging** environment used for pre-production validation before releasing to production.

---

## Purpose

- **Pre-production validation** — test releases, integrations, and config in an environment that mirrors production (same cluster pattern, different namespace and data).
- **Same cluster as development** — staging uses namespace **stg** in cluster **development-ace-eks** (us-east-1), not a separate cluster.

---

## Cluster and namespace

| Item | Value |
|------|--------|
| **Cluster** | **development-ace-eks** |
| **Region** | us-east-1 |
| **Namespace** | **stg** |

**kubectl context** (example): `arn:aws:eks:us-east-1:<account>:cluster/development-ace-eks`

```bash
kubectl config set-context --current --namespace=stg
kubectl get pods
```

---

## How to deploy

- **CI/CD**: **GitHub Actions** — typically triggered from a specific branch or tag; may require **approvals** for staging (see GitHub Environment protection rules).
- **Manifests**: From **ace-infra**; staging deployments use namespace **stg** in the same cluster as dev. Ensure manifests and pipelines target the correct namespace and image tags.
- **Branch strategy**: Follow project Gitflow; staging is usually deployed from a defined branch (e.g. `staging` or via tag). See [../rules/gitflow-rules.md](../rules/gitflow-rules.md).

---

## Access and URLs

- **Ingress/ALB**: Staging services are exposed via the cluster Ingress; hostnames or paths usually differ from dev (e.g. `stg.*` or separate path). Check ace-infra Ingress and Route53 for exact URLs.
- Use staging URLs for QA, UAT, and integration tests; never use production credentials or data in staging without explicit design.

---

## Secrets

- **Path pattern**: **`ace/stg/<service>-secrets`** in **AWS Secrets Manager** (e.g. `ace/stg/ace-stack-backend-secrets`).
- Staging should use its own secrets (and optionally its own DB or RDS instance) so production is not affected. See [../rules/infrastructure-rules.md](../rules/infrastructure-rules.md).

---

## Differences from dev and production

| Aspect | Development (dev) | Staging (stg) | Production (prod) |
|--------|-------------------|---------------|-------------------|
| Cluster | development-ace-eks | development-ace-eks | production-ace-eks |
| Namespace | dev | stg | prod, apis, ace-system |
| Purpose | Daily dev, integration | Pre-prod validation | Live system |
| Approvals | As defined | Often required | Required |
| Secrets path | ace/dev/... | ace/stg/... | ace/prod/... |

---

## Links

- **architecture/deployment.md** — [../architecture/deployment.md](../architecture/deployment.md)
- **ace-infra** — Manifests and pipelines for stg namespace.
- **env-vars-and-secrets** — [env-vars-and-secrets.md](./env-vars-and-secrets.md)
- **production** — [production.md](./production.md)
- **troubleshooting** — [troubleshooting.md](./troubleshooting.md)
