# Infrastructure Rules

Rules for infrastructure and cloud resources used by the ACE system. All infrastructure is hosted on AWS.

---

## 1. AWS region: us-east-1

- **All infrastructure** for the ACE system runs on **AWS** in the **us-east-1** (N. Virginia) region.
- Do not create ACE resources in other regions unless explicitly required by an approved design (e.g. multi-region DR). Default to us-east-1 for new resources.

**Agents**: When creating or documenting infrastructure (Terraform, CloudFormation, or manual), use or assume **us-east-1** as the target region for ACE.

---

## 2. Mandatory resource tags

- **Every resource** that belongs to the ACE system **must** have the following tags:
  - **`Project`** = **`ACE`**
  - **`Environment`** = **`<ENV>-ACE`**  
    where `<ENV>` is the environment name (e.g. `dev`, `staging`, `prod`), so the value is something like `dev-ACE`, `staging-ACE`, `prod-ACE`.
- These tags are used for cost allocation, filtering, and governance. Do not omit them on new resources.

**Agents**: When defining or creating AWS (or other) resources for ACE, ensure each resource has `Project = ACE` and `Environment = <ENV>-ACE`. In Terraform or similar, add these tags to the appropriate resource or module (e.g. `tags` block or provider default tags).

---

## 3. Analyze before creating resources

- **Before creating any new resource**, the options must be **analyzed** and evaluated against the **current architecture**.
- Consider:
  - How the resource fits into the **existing** infrastructure (networking, security, naming, dependencies).
  - **Cost**: Choose the option that **best combines** correct behavior in our infra with **low cost**.
- Do not add resources without evaluating alternatives (e.g. managed vs self-managed, instance sizes, storage classes, reserved vs on-demand).

**Agents**: When asked to add a new AWS (or infra) resource, first describe or review the current architecture and list viable options (e.g. service type, tier, region). Then recommend or implement the option that fits the architecture and minimizes cost while meeting requirements.

---

## 4. Kubernetes (EKS)

- ACE workloads run on **EKS** (cluster name pattern: **`<environment>-ace-eks`**, e.g. `development-ace-eks`).
- Use the **correct namespace** for each environment and service: e.g. `dev`, `stg`, `ace-system`, `apis`, as defined in the cluster. Do not deploy ACE application pods to `default` or ad-hoc namespaces unless the namespace is part of the project’s namespace list.
- Kubernetes manifests (Deployment, Service, Ingress, ConfigMap, etc.) live under **ace-infra** in per-service folders (e.g. `ace-jira-integration/`, `ace-llm/`). Each service typically has a Dockerfile and YAML manifests; keep the same structure for new services.
- Ingress and ALB: Use the existing ALB Ingress Controller and group names (e.g. `alb.ingress.kubernetes.io/group.name`) for consistent routing and TLS.

**Agents**: When adding or changing K8s manifests in ace-infra, set the appropriate `namespace` in metadata and follow existing naming (e.g. `*-deployment.yaml`, `*-service.yaml`, `*-ingress.yaml`). Do not create new namespaces without aligning with the cluster’s `namespaces` variable or documented convention.

---

## 5. Secrets and configuration

- **Secrets** must not be stored in code or in plain text in repos. Use **AWS Secrets Manager** (or equivalent) for runtime secrets.
- Secrets with the path pattern **`ace/<env>/<service>-secrets`** (e.g. `ace/dev/jira-integration-secrets`, `ace/stg/jira-integration-secrets`, `ace/prod/jira-integration-secrets`) are **only for environment variables that the applications will use**. They are not for storing arbitrary values or configuration. Their sole purpose is to be loaded and exposed as environment variables to the apps. New services that need env vars at runtime should follow this pattern.
- CI/CD secrets (e.g. for GitHub Actions, deploy credentials) are stored in **GitHub Environments** and used by workflows. Do not put production secrets in repository variables or in workflow files.
- When adding a new service that needs env vars from Secrets Manager, document the required keys and the path in the service’s README or docs (e.g. under ace-infra or the service repo).

**Agents**: When introducing a new service that needs env vars, document the expected secret path and keys; use AWS Secrets Manager and inject into the app as environment variables (e.g. via K8s or deployment pipeline). Do not use `ace/<env>/<service>-secrets` for general config or arbitrary data—only for values that become app env vars. Do not commit secrets or hardcode them in manifests or code.

---

## 6. CI/CD and deployments

- **CI/CD** for the ACE system uses **GitHub Actions**. Typical flow: build on push/tag, push images to ECR, deploy to EKS (dev/stg/prod) using manifests from ace-infra.
- **Deploys** to production (or sensitive environments) must use **approvals** (e.g. GitHub Environment protection rules). Do not bypass approval requirements for production.
- **Docker images** for ACE services are built from Dockerfiles in ace-infra (or in the service repo when specified) and pushed to **ECR** in us-east-1. Image naming and tagging should follow the project’s convention (e.g. by service name and tag).

**Agents**: When adding or changing deployment workflows, ensure production (and optionally staging) deploys require approvals. Use GitHub Environments for secrets; build and push images to ECR in us-east-1; apply K8s manifests from ace-infra.

---

## 7. ace-infra repository structure

- **<service-name>/** – One folder per deployed service (e.g. ace-jira-integration, ace-llm) containing Dockerfile(s) and Kubernetes manifests (deployment, service, ingress). Scripts (e.g. DB setup, queue creation) may live in **scripts/**.
- **docs/** – Infrastructure documentation (e.g. EKS pods inventory, runbooks). Update when adding services or changing topology.

**Agents**: When adding a new ACE service to the cluster, create a folder under ace-infra with the same naming pattern (e.g. `ace-<service>/`), add Dockerfile and K8s YAMLs, and document the service and any scripts.

---

## Summary for AI agents

| Topic | Rule |
|-------|------|
| Region | All ACE infra on AWS in **us-east-1**. |
| Tags | Every resource: **Project** = **ACE**, **Environment** = **<ENV>-ACE**. |
| New resources | Analyze options and current architecture; choose the option that fits best and keeps cost low. |
| Kubernetes | EKS cluster `<env>-ace-eks`; use correct namespaces; manifests in ace-infra per-service folders; follow existing ALB/Ingress patterns. |
| Secrets | **`ace/<env>/<service>-secrets`** = env vars for apps only (not arbitrary config); CI/CD secrets in **GitHub Environments**; document required keys. |
| CI/CD | GitHub Actions; production deploys require **approvals**; images to ECR us-east-1. |
| ace-infra layout | One folder per service (Dockerfile + K8s), scripts/, docs/. |
