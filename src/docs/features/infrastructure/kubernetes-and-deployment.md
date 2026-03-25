# Kubernetes and Deployment

This document describes how ACE uses **Kubernetes (EKS)** and the **deployment flow**: cluster names, namespaces, ALB Ingress Controller, manifest patterns, and CI/CD (GitHub Actions, ECR, deploy to EKS). It does not contain sensitive data (no image URLs with account IDs, no secret values).

---

## EKS clusters

| Cluster name            | Purpose                          | Typical namespaces (ACE)      |
|-------------------------|----------------------------------|-------------------------------|
| **development-ace-eks** | Development, staging, demo       | dev, stg, demo, ace-system    |
| **production-ace-eks** | Production                       | prod, apis, ace-system         |

- **Region**: us-east-1.
- **kubeconfig**: Use `aws eks update-kubeconfig --name <cluster-name> --region us-east-1` to configure kubectl. Required permissions: EKS DescribeCluster and (if using IAM auth) STS.

---

## Namespaces

Namespaces isolate workloads per environment or purpose. Use the **correct namespace** in every manifest for the target environment.

| Namespace     | Use |
|---------------|-----|
| **dev**       | Development environment workloads. |
| **stg**       | Staging environment workloads. |
| **demo**      | Demo environment workloads. |
| **prod**      | Production application workloads. |
| **apis**      | Shared API/gateway workloads in production (e.g. ace-db-gateway). |
| **ace-system** | System-level workloads (e.g. docker-cleanup, cluster utilities). |
| **kube-system** | EKS system components (CoreDNS, AWS node, EBS/EFS CSI, metrics-server, ALB controller). |
| **monitoring** | Prometheus, Grafana, Alertmanager. |
| **cert-manager** | TLS certificate management. |
| **amazon-cloudwatch** | CloudWatch agent, Fluent Bit (logging). |

The `monitoring` namespace hosts the ACE observability stack (Prometheus/Grafana/Alertmanager + Loki/Promtail). For deploy/access and operational validation, see [monitoring-stack.md](./monitoring-stack.md).

Other namespaces may exist (e.g. wiki-dev, wiki-stg, bmt-wiki-js, sandbox). Do not deploy ACE application pods to **default** unless documented. When adding a new environment, create the namespace (via Terraform or kubectl) and use it consistently in manifests and CI/CD.

---

## ALB Ingress Controller

- **Controller**: AWS Load Balancer Controller (Helm chart in kube-system). Manages Application Load Balancers (ALBs) and target groups based on Ingress resources.
- **Annotations**: Ingress resources use annotations such as `alb.ingress.kubernetes.io/group.name` to group ingress and share an ALB. Use the same group name as other ACE ingress in that environment for consistent routing and TLS.
- **Subnets**: Public subnets are tagged `kubernetes.io/role/elb = 1`; private subnets `kubernetes.io/role/internal-elb = 1`. The controller uses these to place ALBs.
- **TLS**: TLS can be configured via cert-manager or ALB HTTPS listeners; hostnames follow the convention per environment (e.g. dev-*.ace.ezops.cloud, stg-*.ace.ezops.cloud).

---

## Manifest patterns

Manifests live in **ace-infra**, one folder per service (e.g. ace-db-gateway, ace-web-backend).

### Deployment

- **File naming**: `ace.<service>.deployment.yaml` (e.g. `ace.db-gateway.deployment.yaml`).
- **Metadata**: `name`, `namespace` (must match target environment).
- **Image**: Image is pulled from ECR. The image URL is set by CI/CD (e.g. from ECR registry in us-east-1). Do not hardcode account IDs in documentation.
- **Secrets**: Use `envFrom.secretRef` pointing to a Kubernetes Secret. The Secret is populated from AWS Secrets Manager (`ace/<env>/<service>-secrets`) by CI/CD or an external-secrets mechanism. Never put secret values in the manifest repo.
- **Ports**: Expose the container port the app listens on (e.g. 80, 8080).
- **Health checks**: Define `livenessProbe` and `readinessProbe` when possible (e.g. HTTP GET /health). Document the path in the service docs.

### Service

- **File naming**: `ace.<service>.service.yaml`.
- **Type**: Usually ClusterIP for internal access; LoadBalancer only when explicitly required.
- **Selector**: Must match the Deployment pod labels (e.g. `app: ace-db-gateway`).
- **Port**: Target the same port as the container.

### Ingress

- **File naming**: `ace.<service>.ingress.yaml`.
- **Host**: Set the hostname for the environment (e.g. `dev-dashboard.ace.ezops.cloud`, `stg-api-ace.ace.ezops.cloud`). Use the same pattern as existing ingress in that namespace.
- **Paths**: Define path and backend service/port. Use the same ALB group name as other ingress in the namespace so they share one ALB.
- **TLS**: Configure if the environment uses HTTPS (cert-manager or ALB listener).

---

## CI/CD flow

- **Pipeline**: **GitHub Actions** (primary). Workflows live in each application repo and in ace-infra when needed.
- **Build**: On push or tag, the workflow runs lint/type check, builds the Docker image using the Dockerfile (from ace-infra or the app repo as documented), and pushes to **ECR** in us-east-1. Image tag is derived from branch, commit, or environment.
- **Deploy**: The workflow applies Kubernetes manifests from **ace-infra** (Deployment, Service, Ingress). It uses the correct **namespace** and **image tag** for the target environment (dev, stg, prod). Secrets are injected from AWS Secrets Manager (path `ace/<env>/<service>-secrets`) or via External Secrets; they are not stored in the repo or in workflow file content.
- **Environments**: GitHub **Environments** (e.g. development, staging, production) define the deploy target and hold CI/CD secrets (e.g. ECR push, kubeconfig or OIDC for EKS). **Production** (and optionally staging) deploys require **approvals** (Environment protection rules).
- **Branch strategy**: Deploy to dev from `development` (or the branch configured for dev); to prod from the branch defined for production. See [../rules/gitflow-rules.md](../rules/gitflow-rules.md) and [../environments/](../environments/).

---

## Updating a manifest

1. Edit the YAML in the correct **ace-infra/ace-<service>** folder.
2. Keep **namespace** and **image** (tag) consistent with the environment. Image tag is usually set by the pipeline; do not hardcode a specific tag unless required for pinning.
3. For new env vars, document the key and the Secrets Manager path in the service docs and in [../environments/env-vars-and-secrets.md](../environments/env-vars-and-secrets.md). Ensure the secret exists in AWS and is mounted as a K8s Secret in the namespace.
4. Run `kubectl apply -f ...` locally for testing, or rely on the pipeline to apply on merge. Do not commit secret values.
5. After deploy, update **docs/eks-pods-inventory.md** in ace-infra if you maintain a pod inventory.

---

## EKS pods inventory

ace-infra **docs/eks-pods-inventory.md** contains a snapshot of pods per namespace (e.g. for development-ace-eks). Useful for auditing and troubleshooting. To refresh:

```bash
aws eks update-kubeconfig --name development-ace-eks --region us-east-1
kubectl get pods -A -o wide
```

Update the doc when adding/removing services or namespaces. Do not include sensitive data (e.g. env vars, secret names that reveal internal naming beyond `*-secrets`).

---

## Links

- **ace-infra layout**: [ace-infra-repository.md](./ace-infra-repository.md)
- **AWS (EKS, ECR)**: [aws-resources.md](./aws-resources.md)
- **Infrastructure rules**: [../rules/infrastructure-rules.md](../rules/infrastructure-rules.md)
- **Environments and deploy targets**: [../environments/](../environments/)
- **Creating a new environment**: [../environments/creating-a-new-environment.md](../environments/creating-a-new-environment.md)

