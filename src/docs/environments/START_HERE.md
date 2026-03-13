# Start Here – Environments

This directory documents how to set up and deploy ACE in each environment. Below is each file with a short description.

---

## Files

**[README.md](./README.md)**  
Overview of this directory: what it is for, which environments exist, and how to use it. Links to parent docs, local-env, and ace-infra.

**[local.md](./local.md)**  
Local development setup: prerequisites (Docker, Node), how to run with docker-compose (local-env), which services must be up (backend, db-gateway, Redis, PostgreSQL, frontend), order of startup, and health checks.

**[development.md](./development.md)**  
Development environment (EKS): cluster development-ace-eks (us-east-1), namespace dev. How to deploy, access, and key secrets path (ace/dev/...).

**[staging.md](./staging.md)**  
Staging environment (EKS): cluster development-ace-eks (us-east-1), namespace stg. Pre-production validation, deploy flow, access, and secrets path (ace/stg/...).

**[demo.md](./demo.md)**  
Demo environment (EKS): namespace demo in cluster development-ace-eks (us-east-1). Purpose, how to deploy and access, secrets path if used.

**[production.md](./production.md)**  
Production environment (EKS): cluster production-ace-eks (us-east-1), namespaces prod, apis, ace-system. Safeguards, deploy (approvals), access, secrets (ace/prod/...), rollback and monitoring.

**[env-vars-and-secrets.md](./env-vars-and-secrets.md)**  
Where environment variables and secrets come from per environment (local: .env/docker-compose; EKS: Secrets Manager ace/<env>/<service>-secrets). Important vars by service or concern. CI/CD secrets in GitHub Environments.

**[troubleshooting.md](./troubleshooting.md)**  
Common issues (Redis, DB Gateway, CORS, wrong base URL), per-environment differences (URLs, logging), and where to look (logs, K8s, docker-compose).

---

*For high-level deployment architecture, see [../architecture/deployment.md](../architecture/deployment.md). For infrastructure rules (region, tags, secrets), see [../rules/infrastructure-rules.md](../rules/infrastructure-rules.md).*
