# Route53, DNS, and Apontamentos

This document describes how **DNS and apontamentos** (pointers) are managed for ACE: **Route53** hosted zones, record types, conventions, and the scripts used to create or update records. It does not contain sensitive data (no hosted zone IDs that reveal account, no API keys).

---

## Overview

- **DNS** for ACE environments is managed in **AWS Route53** (or, for some setups, an external DNS provider). The main documented pattern uses a **single hosted zone** (e.g. `ace.ezops.cloud`) for environment hostnames.
- **Records** point to **ALBs** (created by the AWS Load Balancer Controller from Kubernetes Ingress). When a new service or environment is deployed, its Ingress gets an ALB; the ALB hostname is then registered in Route53 so that users can reach the service via a friendly name (e.g. `dev-dashboard.ace.ezops.cloud`).

---

## Hosted zone and naming convention

- **Hosted zone**: One zone per domain (e.g. **ace.ezops.cloud**). The zone ID is used by scripts and CI; obtain it from the Route53 console or via `aws route53 list-hosted-zones`.
- **Naming convention** for ACE:
  - **<env>-<service>.ace.ezops.cloud**  
    Examples: `dev-dashboard.ace.ezops.cloud`, `stg-api-ace.ace.ezops.cloud`, `demo-dashboard.ace.ezops.cloud`, `prod-api-ace.ace.ezops.cloud`.
  - **Jira integration** (if used): `dev-jira.ace.ezops.cloud`, `stg-jira.ace.ezops.cloud`, `jira.ace.ezops.cloud` (see IAM policy below).
- **Record type**: Usually **CNAME** (or **A/ALIAS** if the provider supports alias to the ALB). The **target** is the ALB hostname (e.g. `k8s-<ingress>-<hash>.<region>.elb.amazonaws.com`).

---

## How records are created or updated

1. **Kubernetes Ingress** is applied (e.g. by CI/CD). The AWS Load Balancer Controller creates an **ALB** and sets the Ingress status with the ALB hostname.
2. **DNS record** must point that hostname (e.g. `dev-dashboard.ace.ezops.cloud`) to the ALB. This can be done:
   - **Manually**: In Route53, create or update a CNAME (or ALIAS) record with the ALB hostname.
   - **Script**: Use the **update-route53-demo.sh** script for the **demo** environment (see below).
   - **CI/CD or automation**: A step in the pipeline (or a separate job) can read the Ingress hostname and call Route53 **ChangeResourceRecordSets** to upsert the record.

---

## Script: update-route53-demo.sh

- **Location**: **ace-infra/scripts/update-route53-demo.sh**
- **Purpose**: Updates **Route53** records for the **demo** environment so that demo hostnames point to the ALB(s) of the demo Ingress resources.
- **Steps** (high level):
  1. Reads **hosted zone** (e.g. `ace.ezops.cloud`) and **namespace** (`demo`).
  2. For each app (e.g. demo-dashboard, demo-api-ace, demo-db-gateway), gets the **Ingress** ALB hostname from the cluster (`kubectl get ingress -n demo`).
  3. **UPSERT**s a **CNAME** record: `<record>.ace.ezops.cloud` → ALB hostname.
- **Prerequisites**: `kubectl` configured for the cluster (e.g. development-ace-eks), **AWS CLI** with permissions to `route53:ListHostedZones`, `route53:ListResourceRecordSets`, `route53:ChangeResourceRecordSets`, `route53:GetChange`. The script checks kubectl, AWS, and namespace before running.
- **Usage**: Run from a machine that has cluster and AWS access:  
  `./scripts/update-route53-demo.sh`  
  (from the ace-infra repo root, or pass the correct path.)
- **Convention**: Demo records follow the pattern **demo-<app>.ace.ezops.cloud** (e.g. `demo-dashboard.ace.ezops.cloud` → web-frontend Ingress ALB).

---

## Route53 IAM policy (Jira integration)

- **File**: **ace-infra/iam/route53-ace-jira-policy.json**
- **Purpose**: IAM policy that allows **upserting CNAME records** only for specific record names used by the **ACE–Jira integration** (e.g. `dev-jira.ace.ezops.cloud`, `stg-jira.ace.ezops.cloud`, `jira.ace.ezops.cloud`). This restricts the scope of automation so that only those records can be changed.
- **Actions**: `route53:ListHostedZones`, `route53:ListResourceRecordSets`, `route53:ChangeResourceRecordSets`, `route53:GetChange`. Conditions limit **ChangeResourceRecordSets** to CNAME and to the listed normalized record names.
- **Applying the policy**: Use **ace-infra/scripts/apply-route53-jira-policy.sh** to create or update the IAM policy and attach it to the role/user used by the Jira integration (e.g. a role assumed by the app that updates DNS for Jira callbacks). Do not document the role ARN or account ID in this manual.

---

## Creating or updating a record manually (Route53)

1. Open **Route53** → **Hosted zones** → select the zone (e.g. `ace.ezops.cloud`).
2. **Create record** (or edit existing):
   - **Record name**: e.g. `dev-dashboard` (full name will be `dev-dashboard.ace.ezops.cloud`).
   - **Record type**: CNAME (or A – Alias if you choose Alias to ALB).
   - **Value**: ALB hostname from the Ingress (e.g. `k8s-xxx-xxx.us-east-1.elb.amazonaws.com`). For Alias, choose “Alias to Application Load Balancer” and select the ALB.
   - **TTL**: e.g. 60 or 300 (scripts may use a low TTL for demo).
3. Save. Propagation is usually quick (seconds to a few minutes).

---

## Finding the ALB hostname for an Ingress

```bash
# List Ingress in namespace and show hostname
kubectl get ingress -n <namespace> -o wide

# Or JSON path for the first ingress ALB hostname
kubectl get ingress -n <namespace> -o jsonpath='{.items[0].status.loadBalancer.ingress[0].hostname}'
```

Use that hostname as the **target** of the CNAME (or as the Alias target) in Route53.

---

## TLS and certificates

- **HTTPS**: TLS can be handled at the **ALB** (ACM certificate) or in the application. For ACM, request or import a certificate in **us-east-1** and attach it to the ALB listener (often done by the Ingress annotation or by the AWS Load Balancer Controller).
- **Certificate validation**: If using DNS validation, create the **CNAME** record that ACM provides in the same hosted zone so that the certificate is issued. After that, the main app CNAME (e.g. `dev-dashboard.ace.ezops.cloud`) can point to the ALB.

---

## Summary

| Topic | Where / how |
|-------|-------------|
| **Hosted zone** | Route53, one zone per domain (e.g. ace.ezops.cloud). |
| **Naming** | <env>-<service>.ace.ezops.cloud (e.g. dev-dashboard, stg-api-ace, demo-dashboard). |
| **Record type** | CNAME (or A/ALIAS) to ALB hostname. |
| **Demo updates** | Script **scripts/update-route53-demo.sh** (kubectl + AWS CLI). |
| **Jira integration** | IAM policy **iam/route53-ace-jira-policy.json**; apply with **scripts/apply-route53-jira-policy.sh**. |
| **ALB hostname** | From `kubectl get ingress -n <namespace>`. |

---

## Links

- [scripts-and-automation.md](./scripts-and-automation.md) — update-route53-demo.sh, apply-route53-jira-policy.sh
- [kubernetes-and-deployment.md](./kubernetes-and-deployment.md) — Ingress and ALB
- [../environments/creating-a-new-environment.md](../environments/creating-a-new-environment.md) — Ingress hostnames for new environments

