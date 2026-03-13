# Creating a New Environment

This document defines **all steps** required to create a new ACE environment (EKS), following the same patterns as **development**, **staging**, and **production**. Use it when you need a new isolated environment (e.g. a second staging, a dedicated QA, or a customer-specific demo).

---

## Purpose and scope

- **When to use**: New environment needed for a new namespace (same cluster) or a new cluster, with its own URLs, secrets, and CI/CD target.
- **Reference environments**: Structure and steps are derived from [development.md](./development.md), [staging.md](./staging.md), and [production.md](./production.md). The new environment will have its own doc in the same format.
- **Out of scope**: Local setup (see [local.md](./local.md)); changes to existing envs only (use the specific env doc and ace-infra runbooks).

---

## Prerequisites

- **Access**: AWS (us-east-1) with permissions for EKS, Secrets Manager, ECR, and (if used) Route53; GitHub repo access and permission to manage Environments and workflows in ACE repos.
- **Repos**: **ace-infra** (manifests, scripts, Terraform), each application repo (CI/CD workflows), and **ace-manual** (this doc and env docs).
- **Decisions**: Environment **short name** (e.g. `qa`, `preprod`), **purpose**, and whether it will use the **existing cluster** (e.g. development-ace-eks) with a new namespace or a **new cluster** (e.g. qa-ace-eks). Default is same cluster + new namespace.

---

## Step 1 — Define environment identity

| Item | Example | Notes |
|------|--------|--------|
| **Short name** | `qa` | Lowercase; used in namespace, secrets path, URL prefix, branch names. |
| **Full name** | QA | Used in docs and GitHub Environment display name. |
| **Purpose** | Pre-release QA | One sentence; document in the new env doc. |
| **Cluster** | development-ace-eks | Or production-ace-eks; or new cluster name (e.g. qa-ace-eks). |
| **Namespace(s)** | `qa` | One main namespace is typical; prod uses prod, apis, ace-system. |

Document these in a short table in the new environment doc (see Step 10).

---

## Step 2 — Create Kubernetes namespace(s)

- **Where**: Cluster chosen in Step 1 (e.g. via ace-infra Terraform/manifests or `kubectl`).
- **Action**: Ensure the namespace exists. If the cluster is managed by Terraform in ace-infra, add the new namespace to the list of namespaces and apply. Otherwise create it explicitly:

```bash
kubectl create namespace qa
```

- **Tags/labels**: If your cluster uses labels for cost or governance, add the same pattern as other envs (e.g. `Environment=<short>-ACE`). See [../rules/infrastructure-rules.md](../rules/infrastructure-rules.md).

---

## Step 3 — Create secrets in AWS Secrets Manager

- **Path pattern**: **`ace/<short>/<service>-secrets`** (e.g. `ace/qa/ace-stack-backend-secrets`, `ace/qa/ace-db-gateway-secrets`).
- **Action**: For each service that will run in the new environment, create a secret at that path with the **same keys** as dev/stg/prod (see [env-vars-and-secrets.md](./env-vars-and-secrets.md) and each service’s docs). Values must be specific to this environment (e.g. own DB URL, Redis, JWT secret; do not reuse production secrets).
- **Services to consider**: ace-stack-backend, ace-db-gateway, ace-dashboard-frontend (build-time vars may come from pipeline or env-specific config), ace-slackbot, ace-ops-bot, ace-sec-bot, ace-commands-api, ace-ops-scheduler, ace-configuration, and any other deployed service.
- **Rule**: Only environment variables for the application; no arbitrary config. See [../rules/infrastructure-rules.md](../rules/infrastructure-rules.md).

---

## Step 4 — Create or configure GitHub Environment

- **Action**: In GitHub (org or repo), create an **Environment** with the same name as the short name or full name (e.g. `qa` or `QA`), so workflows can target it.
- **Secrets**: Add any **CI/CD secrets** needed by the workflows (e.g. ECR push, kubectl/deploy credentials, tokens). Do not store production-level secrets in repository variables; use Environment secrets.
- **Protection rules**: If the environment should require approvals (e.g. like staging/production), configure **Environment protection rules** (required reviewers, wait timer). For a “dev-like” env, approvals may be optional.

---

## Step 5 — Add or adapt CI/CD workflows

- **Where**: Each application repo that must deploy to the new environment (e.g. ace-dashboard-frontend, ace-stack-backend, ace-db-gateway, ace-slackbot, ace-ops-scheduler).
- **Action**:
  - Add a workflow file (e.g. `eks-deploy.qa.yaml`) or duplicate and adjust an existing one (e.g. from `eks-deploy.dev.yaml` or `eks-deploy.demo.yaml`).
  - Set **trigger** (branch or tag), e.g. push to `qa` or `qa/*`.
  - Set **GitHub Environment** to the new environment (e.g. `environment: qa`).
  - Set **namespace** to the new namespace (e.g. `qa`).
  - Set **image tag** pattern (e.g. `qa`, `qa-<timestamp>`).
  - Ensure the workflow uses **ace-infra** manifests and replaces or passes the correct namespace and ingress host (see Step 6).
- **Branch strategy**: Document in the new env doc which branch(es) trigger deploys (e.g. `qa` or `development` → qa). Align with [../rules/gitflow-rules.md](../rules/gitflow-rules.md).

---

## Step 6 — Add or adapt manifests and ingress in ace-infra

- **Manifests**: For each service deployed to the new environment, ensure there are manifests (or parameterized templates) that use the **new namespace** and, if needed, env-specific image tags. Manifests typically live in per-service folders under ace-infra.
- **Ingress**: Add or duplicate Ingress resources with **hostnames** for the new environment. Convention: **`<short>-<service>.ace.ezops.cloud`** (e.g. `qa-dashboard.ace.ezops.cloud`, `qa-api-ace.ace.ezops.cloud`). Use the same ALB Ingress Controller and group name as other envs.
- **Route53**: If DNS is managed in Route53, create or update records for the new hostnames (A/ALIAS to the ALB). Some envs use a script (e.g. `update-route53-demo.sh`); add a similar script or step for the new env if needed.
- **Tags**: Ensure any new AWS resources (if you create any) have **Project = ACE** and **Environment = &lt;short&gt;-ACE** (e.g. `qa-ACE`). See [../rules/infrastructure-rules.md](../rules/infrastructure-rules.md).

---

## Step 7 — Document base URLs and access

- **List**: In the new environment doc, add an **Access and URLs** (or **Workloads and URLs**) section with the exact hostnames and, if applicable, internal K8s service DNS.
- **Internal access**: Same pattern as dev/stg: e.g. `http://ace-db-gateway.qa.svc.cluster.local` for services in namespace `qa`.
- **External access**: Document the public URLs (e.g. https://qa-dashboard.ace.ezops.cloud) and note any restrictions (e.g. VPN, IP allowlist) if applicable.

---

## Step 8 — Update env-vars-and-secrets and conventions

- **env-vars-and-secrets.md**: Add a row for the new environment in the “Where vars come from per environment” table: **Environment** = full name, **Source** = `ace/<short>/<service>-secrets`, **Notes** = short note (e.g. “Same injection pattern as dev”).
- **Conventions**: Ensure the new env uses **environment variables** for all base URLs and secrets (no hardcoding). See [../rules/standardization-rules.md](../rules/standardization-rules.md).

---

## Step 9 — Update troubleshooting and per-environment table

- **troubleshooting.md**: In the “Per-environment differences” table, add a **column** for the new environment (e.g. **QA (qa)**) with Cluster, Namespace, Base URLs, Secrets path, Logging, Feature flags, Approvals as for dev/stg/prod.
- **Other runbooks**: If ace-infra or ops has a central list of environments or namespaces, add the new one there.

---

## Step 10 — Create the environment document

- **File**: Create **`<short>.md`** in `ace-manual/src/docs/environments/` (e.g. `qa.md`), following the structure of [development.md](./development.md), [staging.md](./staging.md), or [production.md](./production.md).
- **Sections to include**:
  - **Purpose** — one short paragraph (from Step 1).
  - **Cluster and namespace** — table: Cluster, Region, Namespace(s); example `kubectl` context and commands.
  - **How to deploy** — CI/CD (GitHub Actions), branch strategy, manifests location, approvals if any.
  - **Access and URLs** — from Step 7 (hostnames, internal DNS).
  - **Secrets** — path pattern `ace/<short>/<service>-secrets` and pointer to env-vars-and-secrets.md.
  - **Differences from other envs** (optional) — short table vs dev/stg/prod if helpful.
  - **Links** — architecture/deployment.md, ace-infra, env-vars-and-secrets.md, troubleshooting.md, other env docs.
- **Language**: English. Keep it concise; refer to other docs for details.

---

## Step 11 — Update index and entry points

- **index.md**: Add a link to the new env doc: `- [<short>.md](./<short>.md)` (e.g. `- [qa.md](./qa.md)`).
- **README.md** (environments): If it lists environments, add the new one.
- **START_HERE.md** (environments): If it points to env list or “all envs”, ensure the new env is mentioned or linked where appropriate.

---

## Step 12 — Deploy and validate

- **First deploy**: Trigger the new workflow(s) (e.g. push to the branch that deploys to the new env) and verify that images are built, pushed to ECR, and applied to the correct namespace.
- **Smoke checks**: Open the dashboard URL, call the API health endpoint, and (if applicable) check DB Gateway and Redis. Confirm pods are Running and Ingress routes correctly.
- **Secrets**: Confirm apps start without secret-related errors; verify no production credentials are used in the new env.
- **Docs**: After validation, treat the new env doc as the source of truth for that environment; update it when URLs, namespace, or deploy process change.

---

## Summary checklist (reference only)

| Step | What |
|------|------|
| 1 | Define short name, purpose, cluster, namespace(s). |
| 2 | Create K8s namespace(s). |
| 3 | Create AWS Secrets Manager entries `ace/<short>/<service>-secrets` for each service. |
| 4 | Create GitHub Environment and set CI/CD secrets and protection rules. |
| 5 | Add/adapt workflows (trigger, environment, namespace, image tag). |
| 6 | Add/adapt ace-infra manifests and Ingress; Route53 if needed; tags. |
| 7 | Document base URLs and access in the new env doc. |
| 8 | Update env-vars-and-secrets.md. |
| 9 | Update troubleshooting.md per-environment table (and other runbooks). |
| 10 | Create `<short>.md` env doc (purpose, cluster, deploy, URLs, secrets, links). |
| 11 | Update index.md, README.md, START_HERE.md. |
| 12 | Deploy, smoke-test, and validate. |

---

## Links

- **Existing envs** — [development.md](./development.md), [staging.md](./staging.md), [production.md](./production.md), [demo.md](./demo.md)
- **Secrets and vars** — [env-vars-and-secrets.md](./env-vars-and-secrets.md)
- **Infrastructure rules** — [../rules/infrastructure-rules.md](../rules/infrastructure-rules.md) (region, tags, secrets path, EKS)
- **Gitflow** — [../rules/gitflow-rules.md](../rules/gitflow-rules.md)
- **Architecture deployment** — [../architecture/deployment.md](../architecture/deployment.md)
- **Troubleshooting** — [troubleshooting.md](./troubleshooting.md)
