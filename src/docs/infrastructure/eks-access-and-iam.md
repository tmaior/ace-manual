# EKS Cluster Access and IAM

This document describes **how to access** the ACE EKS clusters and **how to grant or create access** for a user. It does not contain sensitive data (no account IDs, no role ARNs that reveal account, no credentials).

---

## How to access the EKS clusters

### Prerequisites

- **AWS CLI** installed and configured (e.g. `aws configure`, or environment variables `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_SESSION_TOKEN` when using temporary credentials).
- **kubectl** installed.
- **IAM permissions**: The identity you use must be allowed to call `eks:DescribeCluster` (and, for IAM authentication, the cluster must trust that identity; see below).

### Configure kubeconfig

EKS uses IAM for cluster authentication. To add the cluster to your kubeconfig and switch context:

```bash
# Development cluster (dev, stg, demo)
aws eks update-kubeconfig --name development-ace-eks --region us-east-1

# Production cluster
aws eks update-kubeconfig --name production-ace-eks --region us-east-1
```

This creates (or updates) an entry in `~/.kube/config` with the cluster endpoint and the `aws` auth provider. When you run `kubectl`, the AWS CLI (or SDK) is used to generate a short-lived token; your IAM identity must be mapped in the cluster (see "How cluster authentication works" below).

### Verify access

```bash
# Current context
kubectl config current-context

# List nodes (requires list permission on nodes)
kubectl get nodes

# List pods in a namespace (e.g. dev)
kubectl get pods -n dev
```

If you get "Unauthorized" or "Forbidden", your IAM user or role is not mapped in the cluster's `aws-auth` ConfigMap, or your RBAC role does not allow the action.

---

## How cluster authentication works

1. **kubectl** sends requests to the EKS API server with an auth token.
2. The token is obtained via the **aws** exec plugin in kubeconfig: it calls the AWS API (e.g. `eks:DescribeCluster` and STS) to get a signed token for your IAM identity.
3. EKS maps that IAM identity (user ARN or role ARN) to a Kubernetes user or group using the **aws-auth** ConfigMap in the `kube-system` namespace.
4. **RBAC** (Role / RoleBinding / ClusterRole / ClusterRoleBinding) then grants permissions to that user or group (e.g. list pods, get logs).

So to "allow a user to access the cluster" you need:

- The user (or a role they assume) to be **allowed by IAM** to call `eks:DescribeCluster` (and optionally `sts:GetCallerIdentity`) for the cluster.
- The same IAM identity to be **mapped** in `aws-auth` to a Kubernetes user or group.
- **RBAC** to grant that user/group the desired permissions (e.g. read-only, admin in a namespace).

---

## How to allow a user to access the cluster

### Option A: IAM user mapped in aws-auth (typical for human users)

1. **Create or use an IAM user** in the same AWS account as the cluster. The user does not need a console password for kubectl; they need access keys (or SSO/assume-role) so that `aws eks update-kubeconfig` and `kubectl` can authenticate.
2. **Ensure the user can call EKS**: Attach a policy that allows at least:
   - `eks:DescribeCluster` for the cluster (resource: cluster ARN).
   - If using the default `aws` auth flow, no extra STS permissions are needed beyond what the EKS API uses.
3. **Edit the aws-auth ConfigMap** in the cluster so that this IAM user is mapped to a Kubernetes user or group:
   - Connect to the cluster with an identity that already has access (e.g. cluster creator or CI role).
   - Open the ConfigMap: `kubectl -n kube-system edit configmap aws-auth`.
   - Under `mapUsers`, add an entry (example; do not use actual ARNs in docs):
     - `userarn`: IAM user ARN (e.g. `arn:aws:iam::<account-id>:user/<username>`).
     - `username`: Kubernetes username (e.g. the IAM user name or a friendly name).
     - Optional: `groups`: list of Kubernetes groups (e.g. `system:masters` for full admin, or a custom group that you bind via RBAC).
4. **RBAC**: If you used a custom group, create a Role/ClusterRole and a RoleBinding/ClusterRoleBinding that grants that group the desired permissions. If you used `system:masters`, the user already has full access (use with care).

**Example structure** (do not paste real ARNs in documentation):

```yaml
mapUsers: |
  - userarn: arn:aws:iam::ACCOUNT_ID:user/devops-jane
    username: devops-jane
    groups:
      - developers
```

Then ensure a RoleBinding binds the `developers` group to a Role that allows, for example, `get`, `list`, `watch` on pods and logs in namespace `dev`.

### Option B: IAM role (assume role) for users or CI

1. **Create an IAM role** that trusts the IAM users or another account (e.g. for SSO or CI). The role must have a policy that allows `eks:DescribeCluster` (and any other EKS/STS actions required).
2. **Users (or CI)** assume that role (e.g. `aws sts assume-role --role-arn <role-arn> --role-session-name <session>` and export the returned credentials, or use `aws configure` with a profile that uses `role_arn` and `source_profile`).
3. **Map the role** in `aws-auth` under `mapRoles`:
   - `rolearn`: the IAM role ARN.
   - `username`: Kubernetes username for that role (e.g. `ops-role`).
   - `groups`: e.g. `system:masters` or a custom group with RBAC.
4. After assuming the role, run `aws eks update-kubeconfig --name <cluster> --region us-east-1` and use `kubectl`; the token will be for the assumed role.

This is the typical pattern for **CI/CD** (e.g. GitHub Actions): the workflow assumes a role that is mapped in `aws-auth` and then runs `kubectl apply`.

---

## How to "create a new user" for cluster access

- **If by "user" you mean a human**:
  1. Create an **IAM user** in AWS (or use an existing one). Attach a policy that allows `eks:DescribeCluster` for the cluster.
  2. Give the user **access keys** (or configure SSO/assume-role) so they can run `aws` and `kubectl`.
  3. Add the user to the **aws-auth** ConfigMap as in Option A.
  4. Optionally create **RBAC** (Role + RoleBinding) to limit them to specific namespaces or actions.
- **If by "user" you mean a service account for an application** (e.g. a pod that needs to call the Kubernetes API):
  - Use a **Kubernetes ServiceAccount** and, if the app runs outside the cluster, **IRSA** (IAM Roles for Service Accounts) or an IAM role assumed by the app. Map that role in `aws-auth` under `mapRoles` if the app needs to use kubectl from outside the cluster. For in-cluster access, RBAC (RoleBinding to the ServiceAccount) is enough; no aws-auth needed for in-cluster API calls.

---

## Where aws-auth lives and who can edit it

- **ConfigMap**: `kube-system/aws-auth`.
- **Who can edit**: Identities that have Kubernetes permissions to update that ConfigMap (usually cluster admin or a role mapped to `system:masters`). Do not grant broad ConfigMap write in `kube-system` to untrusted users.
- **Best practice**: Prefer managing `aws-auth` via Terraform (e.g. `kubernetes_config_map` or a module that updates aws-auth from a list of users/roles) so that changes are versioned and reviewed. If you edit by hand, back up the ConfigMap before changing.

---

## Summary

| Goal | Steps |
|------|--------|
| **Access cluster** | Configure AWS CLI, run `aws eks update-kubeconfig --name <cluster> --region us-east-1`, then use `kubectl`. |
| **Allow a new human user** | Create/use IAM user with EKS DescribeCluster; add user to `kube-system` aws-auth `mapUsers`; optionally add RBAC. |
| **Allow a role (e.g. CI)** | Create IAM role with EKS permissions; add role to aws-auth `mapRoles`; assume role before running kubectl. |
| **Find why access is denied** | Check IAM policy (eks:DescribeCluster, STS); check aws-auth (user/role mapped?); check RBAC (group or user has permission?). |

---

## Links

- [kubernetes-and-deployment.md](./kubernetes-and-deployment.md) — Cluster names and namespaces
- [aws-resources.md](./aws-resources.md) — EKS and region
- AWS docs: [EKS access entries](https://docs.aws.amazon.com/eks/latest/userguide/access-entries.html) (newer model) and [aws-auth ConfigMap](https://docs.aws.amazon.com/eks/latest/userguide/add-user-role.html)

