# Production Environment (EKS)

This document describes the **production** environment where the live ACE system runs. Production has stricter safeguards and approval requirements.

---

## Purpose

- **Live system** — serves real users and data. All changes must follow the release process; no ad-hoc or direct push to production.
- **Stability and security** — use approvals, rollback plan, and monitoring. See [../rules/infrastructure-rules.md](../rules/infrastructure-rules.md).

---

## Cluster and namespaces

| Item | Value |
|------|--------|
| **Cluster** | **production-ace-eks** |
| **Region** | us-east-1 |
| **Namespaces (ACE)** | **prod**, **apis**, **ace-system** (and others as defined in the cluster) |

**kubectl context** (example): `arn:aws:eks:us-east-1:<account>:cluster/production-ace-eks`

```bash
kubectl config set-context --current --namespace=prod
kubectl get pods
```

---

## How to deploy

- **CI/CD**: **GitHub Actions** — production deploys **must** use **approvals** (e.g. GitHub Environment protection rules for `production`). Do not bypass approval requirements.
- **Branch strategy**: PRs to production go from **development** (or the branch defined in project Gitflow). See [../rules/gitflow-rules.md](../rules/gitflow-rules.md).
- **Manifests**: From **ace-infra**; production workloads use namespaces **prod**, **apis**, **ace-system** as appropriate. Image tags and secrets must be production-specific.
- **No direct push** — do not push directly to production branches or apply manifests manually without following the approved process.

---

## Access and URLs

- **Ingress/ALB**: Production services are exposed via the cluster Ingress; use production hostnames and TLS. URLs are environment-specific; document in ace-infra or runbooks.
- **Internal access**: Within the cluster, use K8s service DNS. Restrict external access to production to authorized users and systems only.

---

## Secrets

- **Path pattern**: **`ace/prod/<service>-secrets`** in **AWS Secrets Manager** (e.g. `ace/prod/ace-stack-backend-secrets`, `ace/prod/ace-db-gateway-secrets`).
- **CI/CD secrets** (deploy credentials, GitHub tokens) are in **GitHub Environments**, not in repo variables or workflow file content. See [../rules/infrastructure-rules.md](../rules/infrastructure-rules.md) and [env-vars-and-secrets.md](./env-vars-and-secrets.md).

---

## Rollback and monitoring

- **Rollback**: Have a rollback plan before each production deploy (e.g. previous image tag, revert commit, or K8s rollback). Document in ace-infra or runbooks.
- **Monitoring**: Use the cluster’s monitoring stack (e.g. CloudWatch, Prometheus, namespace **monitoring**) and set alerts for errors, latency, and availability. See ace-infra and ops docs.

---

## Links

- **architecture/deployment.md** — [../architecture/deployment.md](../architecture/deployment.md)
- **infrastructure rules** — [../rules/infrastructure-rules.md](../rules/infrastructure-rules.md)
- **ace-infra** — Production manifests and pipelines.
- **env-vars-and-secrets** — [env-vars-and-secrets.md](./env-vars-and-secrets.md)
- **staging** — [staging.md](./staging.md) (pre-production validation)
- **troubleshooting** — [troubleshooting.md](./troubleshooting.md)
