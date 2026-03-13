# Databases: Credentials and Admin Tools (pgAdmin, Redis Insight, Mongo Express)

This document describes **where to find credentials** for RDS, DocumentDB, and Redis, and **how to access and use** the admin tools (pgAdmin, Redis Insight, Mongo Express) that run inside the EKS cluster. It also explains **how to install** these tools if they are not yet deployed. **No actual passwords or connection strings** are documented.

---

## Where to find RDS credentials

- **Terraform (cluster root)**  
  The RDS instance is created by the **terraform-library/rds** module. The module uses **manage_master_user_password = true**, so AWS RDS stores the **master password** in **AWS Secrets Manager** (not in Terraform state or tfvars).

- **Username**  
  The master username is set in **cluster/environments/<env>/terraform.tfvars** (e.g. `rds_master_username`). It is not a secret; the password is.

- **Password**  
  - **Terraform output**: The module exposes **master_user_secret** (the Secrets Manager secret ARN that contains the password). From the cluster Terraform root:  
    `terraform output` (or `terraform output -json`) after applying. Use that ARN to retrieve the secret value with AWS CLI or the Secrets Manager console (requires IAM permission to read the secret).  
  - **AWS Console**: Secrets Manager → find the secret associated with the RDS instance (name/ARN from Terraform output).  
  - **CLI** (example; replace secret-id with the ARN or name):  
    `aws secretsmanager get-secret-value --secret-id <secret-id> --region us-east-1 --query SecretString --output text`  
    The JSON may contain `username` and `password`.

- **Endpoint**  
  **Terraform output**: **rds_instance_endpoint** from the cluster Terraform (or from the rds module output). Format: `<identifier>.<id>.us-east-1.rds.amazonaws.com`. Port is usually **5432** for PostgreSQL.

- **Applications**  
  Applications in the cluster get RDS URL and credentials from **Secrets Manager** (path pattern `ace/<env>/<service>-secrets`) or from a Kubernetes Secret populated by CI/CD. They do not read Terraform state.

---

## Where to find DocumentDB credentials

- **Terraform**  
  The **terraform-library/documentDB** module creates the cluster and, in many setups, a **Secrets Manager secret** for the master user.

- **Outputs**  
  - **cluster_endpoint**: endpoint hostname for the DocumentDB cluster (port typically **27017**).  
  - **secrets_manager_arn**: ARN of the secret that holds the credentials (if the module creates it).  
  Retrieve the secret value with AWS CLI or the console (IAM permission required). Connection string format is MongoDB-style (e.g. `mongodb://user:password@<cluster_endpoint>:27017/?tls=true&...`). **Do not document the actual connection string or password.**

- **Applications**  
  Services (e.g. commands-api) receive the DocumentDB URL and credentials via **Secrets Manager** (`ace/<env>/<service>-secrets`) or a K8s Secret.

---

## Where to find Redis credentials

- **Terraform**  
  The **terraform-library/redis** module creates an ElastiCache (Redis) replication group. If no `auth_token` is set, Redis is accessed without a password (network-restricted by security group).

- **Endpoint**  
  From Terraform outputs (redis module) or from the ElastiCache console: primary endpoint (host and port **6379**).

- **In the cluster**  
  Redis is often deployed **inside** the cluster (e.g. `redis-deployment` in namespace `dev`). In that case, applications use the **Kubernetes service DNS name** (e.g. `redis-deployment.dev.svc.cluster.local:6379`). Credentials, if any, come from env or Secrets Manager.

---

## Accessing RDS, Redis, and DocumentDB with admin tools

The admin tools **pgAdmin** (PostgreSQL), **Redis Insight** (Redis), and **Mongo Express** (DocumentDB/MongoDB) can be deployed in the cluster (e.g. in namespace **dev**). Access is typically via **port-forward** (or, if configured, an Ingress).

### Prerequisites

- **kubeconfig** configured for the target cluster (see [eks-access-and-iam.md](./eks-access-and-iam.md)).
- **Credentials** for the database (from Terraform outputs + Secrets Manager, or from the team; see above). Never commit them.

### pgAdmin (PostgreSQL / RDS)

- **Deployment**: In ACE, pgAdmin is often deployed via **Helm** in the cluster (e.g. from **terraform-library/eks-general-configs**), in the namespace defined by `pg_namespace` (e.g. `dev`). Service type is typically **ClusterIP**.
- **Access**:
  1. List the pgAdmin pod: `kubectl get pods -n dev -l app.kubernetes.io/name=pgadmin4` (or similar; check Helm release labels).
  2. Port-forward: `kubectl port-forward -n dev svc/pgadmin4 8080:80` (adjust service name and port if different).
  3. Open `http://localhost:8080` in the browser. Log in with the **pgAdmin UI credentials** (email/password). These may be set in the Helm values or in a Secret; obtain from the team or from the values used in Terraform/Helm. **Do not document default passwords.**
- **Adding the RDS server in pgAdmin**: Create a new server in pgAdmin with:
  - **Host**: RDS endpoint (from Terraform output `rds_instance_endpoint`).
  - **Port**: 5432.
  - **Username**: RDS master username (from tfvars).
  - **Password**: From Secrets Manager (see "Where to find RDS credentials" above).
  - **SSL**: Prefer or require, depending on cluster policy.

### Redis Insight (Redis)

- **Deployment**: Manifests in **ace-infra/redisinsights/** (e.g. `dev.manifest.yaml`, `prod.manifest.yaml`). Service is **ClusterIP**; app listens on port **5540** (or 80 in the service mapping).
- **Access**:
  1. Port-forward: `kubectl port-forward -n dev svc/redisinsight-service 5540:80` (or the port used by the container, e.g. 5540).
  2. Open `http://localhost:5540` in the browser.
- **Adding Redis**: In Redis Insight, add a database connection:
  - **Host**: Kubernetes service name of the Redis deployment (e.g. `redis-deployment.dev.svc.cluster.local`) or the ElastiCache endpoint if Redis is external.
  - **Port**: 6379.
  - **Password**: Only if Redis is configured with auth (from env or Secrets Manager).

### Mongo Express (DocumentDB / MongoDB)

- **Deployment**: Manifests in **ace-infra/mongo-express/** (e.g. `dev.mongo-express.yaml`, `prod.mongo-express.yaml`). Service is **ClusterIP**; app listens on port **8081**.
- **Access**:
  1. Port-forward: `kubectl port-forward -n dev svc/mongo-express 8081:8081`.
  2. Open `http://localhost:8081` in the browser. If basic auth is enabled, use the credentials configured in the deployment (from a Secret or env; obtain from the team). **Do not document them.**
- **Connection**: Mongo Express is configured with **ME_CONFIG_MONGODB_URL** (and optionally TLS). The URL points to the DocumentDB cluster endpoint (from Terraform). Credentials and URL are in the manifest or in a Secret; use the same endpoint and credentials as the applications that use DocumentDB.

---

## Installing admin tools if they are not in the cluster

### pgAdmin

- **Via Terraform (recommended)**  
  The **terraform-library/eks-general-configs** module can deploy pgAdmin with Helm. In **cluster/terraform/main.tf** the module is already used; pgAdmin is enabled there with `helm_release.pgadmin4`. It uses the `pg_namespace` variable (e.g. `dev`).  
  - **Credentials**: Set `env.email` and `env.password` via Helm `set` or a values file. **Use a strong password and prefer storing it in a Secret** (reference the secret in Helm values) instead of plain values in Terraform.  
  - **Preconfigured RDS server**: The Helm chart can define a server (host, port, username); the password should come from a Secret or from Secrets Manager, not from Terraform in plain text.  
  - After enabling or changing pgAdmin in Terraform, run `terraform apply` and then use `kubectl port-forward` as above.

- **Manually with Helm**  
  From a machine with kubeconfig and Helm:  
  - Add repo: `helm repo add runix https://helm.runix.net` (or the chart repo used by the project).  
  - Install in the desired namespace (e.g. `dev`):  
    `helm install pgadmin4 runix/pgadmin4 -n dev --set service.type=ClusterIP --set env.email=... --set env.password=...`  
  Use a Secret for the password when possible.

### Redis Insight

- **Manifests** in **ace-infra/redisinsights/** are ready to apply.  
  - **dev**: `kubectl apply -f ace-infra/redisinsights/dev.manifest.yaml`  
  - **prod**: `kubectl apply -f ace-infra/redisinsights/prod.manifest.yaml`  
  - Adjust **namespace** in the YAML if your namespace is different (e.g. `stg`).  
  - Then use port-forward to the service `redisinsight-service` (or the one defined in the manifest).

### Mongo Express

- **Manifests** in **ace-infra/mongo-express/** (e.g. `dev.mongo-express.yaml`, `prod.mongo-express.yaml`).  
  - **Credentials and URL**: The deployment needs **ME_CONFIG_MONGODB_URL** (and optionally ME_CONFIG_BASICAUTH_* for the UI). **Do not commit real URLs or passwords.** Use a Kubernetes Secret and `envFrom` or `valueFrom`.  
  - **DocumentDB TLS**: If DocumentDB requires TLS, mount the CA bundle (e.g. from a Secret) and set **ME_CONFIG_MONGODB_CA_FILE** and **ME_CONFIG_MONGODB_SSL**.  
  - Apply: `kubectl apply -f ace-infra/mongo-express/dev.mongo-express.yaml` (after replacing placeholders with Secret references).  
  - Then port-forward to the `mongo-express` service on port 8081.

---

## Summary table

| Resource   | Where credentials / endpoint come from           | Admin tool   | How to access tool                          |
|-----------|---------------------------------------------------|-------------|---------------------------------------------|
| **RDS**   | Terraform output (endpoint, master_user_secret); Secrets Manager for password; tfvars for username | pgAdmin     | Port-forward to pgadmin4 service in cluster |
| **Redis** | Terraform (ElastiCache) or K8s service DNS; auth if configured | Redis Insight | Port-forward to redisinsight-service         |
| **DocumentDB** | Terraform outputs (cluster_endpoint, secrets_manager_arn); Secrets Manager for password | Mongo Express | Port-forward to mongo-express service       |

---

## Security notes

- **Never commit** RDS, DocumentDB, or Redis passwords, connection strings, or pgAdmin/Mongo Express UI passwords to the repository.
- Prefer **Secrets Manager** or **Kubernetes Secrets** for all admin-tool and DB credentials. In Terraform/Helm, reference secrets by name/ARN instead of plain values.
- Restrict **who can port-forward** to these tools (cluster access is required; see [eks-access-and-iam.md](./eks-access-and-iam.md)). Do not expose pgAdmin, Redis Insight, or Mongo Express on a public Ingress without strong auth and network controls.

---

## Links

- [eks-access-and-iam.md](./eks-access-and-iam.md) — How to get cluster access and port-forward
- [aws-resources.md](./aws-resources.md) — RDS, DocumentDB, Redis
- [terraform-modules.md](./terraform-modules.md) — Where RDS/DocumentDB/Redis are defined
- [../environments/env-vars-and-secrets.md](../environments/env-vars-and-secrets.md) — Env vars and Secrets Manager pattern

---

*Last updated: March 2025*
