# ace-infra – Kubernetes manifests and services

This document describes the Kubernetes manifest pattern used for ACE services in **ace-infra** and gives a concrete example from the codebase.

---

## Pattern per service (ace-* folder)

Each deployable service has a folder **ace-&lt;service-name&gt;** with:

1. **Dockerfile** (or **ace.&lt;service&gt;.Dockerfile**) – used by CI/CD to build the image (or as documented for that service).
2. **ace.&lt;service&gt;.deployment.yaml** – Deployment: replicas, image (e.g. ECR), container port, **envFrom secretRef** for environment variables.
3. **ace.&lt;service&gt;.service.yaml** – Service (ClusterIP or LoadBalancer) pointing to the deployment.
4. **ace.&lt;service&gt;.ingress.yaml** (optional) – Ingress for ALB: hostname, path, TLS; uses **alb** ingress class and ALB annotations.

Secrets are not stored in manifests; they come from Kubernetes Secrets populated by CI/CD or external-secrets (e.g. from AWS Secrets Manager path `ace/<env>/<service>-secrets`).

---

## Example: ace-jira-integration

From the **ace-jira-integration/** folder in the repo.

**Deployment** (`ace.jira-integration.deployment.yaml`):

- **metadata**: name `jira-integration`, namespace `dev`.
- **spec**: 1 replica, selector `app: jira-integration`, template labels and (optional) annotations (e.g. `tags.datadoghq.com/env: development`).
- **containers**: name `jira-integration`, image from ECR (e.g. `975635808270.dkr.ecr.us-east-1.amazonaws.com/ace/jira-integration:latest`), **containerPort 3000**, **envFrom secretRef** `jira-integration-secrets`.

**Service** (`ace.jira-integration.service.yaml`): Exposes the deployment on a port (e.g. 3000); referenced by the Ingress as `jira-integration-service` port 3000.

**Ingress** (`ace.jira-integration.ingress.yaml`):

- **metadata**: name `jira-integration-ingress`, namespace `dev`.
- **annotations**: `alb.ingress.kubernetes.io/group.name: apis-group`, `group.order`, `scheme: internet-facing`, `certificate-arn`, `ssl-policy`, `target-type: ip`, `ssl-redirect`, `listen-ports` for HTTP/HTTPS.
- **spec**: ingressClassName **alb**, rule host **jira.ace.ezops.cloud**, path `/`, backend service **jira-integration-service** port 3000.

---

## Namespaces

Namespaces (e.g. `dev`, `stg`, `prod`) are created by the **eks-general-configs** Terraform module from the **namespaces** variable in **cluster/environments/&lt;env&gt;/terraform.tfvars**. Each manifest must set **metadata.namespace** to the correct value for the target environment.

---

## Adding or changing a service

1. Create or update the **ace-&lt;service-name&gt;** folder.
2. Add or adjust Dockerfile and the three YAML files (deployment, service, ingress if the service is exposed via ALB).
3. Use the same naming convention: **ace.&lt;service&gt;.deployment.yaml**, **ace.&lt;service&gt;.service.yaml**, **ace.&lt;service&gt;.ingress.yaml**.
4. Set **metadata.namespace** to the intended namespace for that environment.
5. Use **envFrom secretRef** only; document required env vars and Secrets Manager path in the service docs and in [../../environments/](../../environments/) as needed.
6. Update [../../infrastructure/ace-infra-repository.md](../../infrastructure/ace-infra-repository.md) and [repository-structure.md](./repository-structure.md) if you add a new service folder, and update **docs/eks-pods-inventory.md** in ace-infra after deployment.

For full step-by-step and links, see [../../infrastructure/ace-infra-repository.md#adding-a-new-service-to-ace-infra](../../infrastructure/ace-infra-repository.md#adding-a-new-service-to-ace-infra).
