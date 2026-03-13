# Environments

Environment setup, configuration, and deployment for ACE. This directory is the **single place** for how to run ACE locally and how to deploy and access each EKS environment (development, staging, demo, production).

---

## Purpose

This directory holds documentation for:

- **Local development** (docker-compose, required services) — see [local.md](./local.md)
- **Development (EKS)** — cluster development-ace-eks, namespace dev — see [development.md](./development.md)
- **Staging (EKS)** — cluster development-ace-eks, namespace stg — see [staging.md](./staging.md)
- **Demo (EKS)** — cluster development-ace-eks, namespace demo — see [demo.md](./demo.md)
- **Production (EKS)** — cluster production-ace-eks, namespaces prod/apis/ace-system — see [production.md](./production.md)
- **Environment variables and secrets** — see [env-vars-and-secrets.md](./env-vars-and-secrets.md)
- **Troubleshooting** — common issues and per-environment differences — see [troubleshooting.md](./troubleshooting.md)
- **Creating a new environment** — step-by-step guide to add a new EKS environment (e.g. qa, preprod) — see [creating-a-new-environment.md](./creating-a-new-environment.md)

---

## Clusters (reference)

| Cluster | Region | Namespaces (ACE) |
|---------|--------|------------------|
| **development-ace-eks** | us-east-1 | dev, stg, demo, ace-system |
| **production-ace-eks** | us-east-1 | prod, apis, ace-system |

---

## How to use

- **New developer / running locally**: Start with [START_HERE.md](./START_HERE.md), then [local.md](./local.md).
- **Deploying to dev/stg/demo/prod**: Read the corresponding env file ([development.md](./development.md), [staging.md](./staging.md), [demo.md](./demo.md), [production.md](./production.md)) and [env-vars-and-secrets.md](./env-vars-and-secrets.md).
- **Creating a new environment**: Follow [creating-a-new-environment.md](./creating-a-new-environment.md) (namespace, secrets, CI/CD, manifests, docs).
- **Something broken**: Check [troubleshooting.md](./troubleshooting.md) and the relevant env doc.

---

## Related

- **local-env/** (repo or folder): Docker-compose and local config files. This directory documents how to use them.
- **ace-infra/**: Terraform, K8s manifests, and infra as code. Env docs reference ace-infra for deploy flow and manifests.
- **architecture/deployment.md**: High-level where ACE runs (EKS, AWS, local). This directory goes into **how** to set up and deploy in each env.
- **rules/infrastructure-rules.md**: Region (us-east-1), tags, secrets path, CI/CD. Keep env-vars-and-secrets and env docs aligned with it.

---

*Last updated: March 2025*
