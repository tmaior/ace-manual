# Step 1: Prerequisites

Before bringing up the local ACE environment, the following must be installed and working on the host. Verify each item; do not proceed to repository layout or Docker Compose until all checks pass.

---

## Required

### 1. Docker

- **Purpose**: All ACE services (PostgreSQL, Redis, Mongo, Node apps, LocalStack, etc.) run as containers.
- **Version**: Docker Engine 20.10+ (Compose V2 is used).
- **Verify**:
  ```bash
  docker --version
  docker run --rm hello-world
  ```
- **Note**: The user running `docker` must be able to start containers without `sudo` (add user to `docker` group if needed).

### 2. Docker Compose (Compose V2)

- **Purpose**: The local stack is defined in `local-env/docker-compose.yaml` and started with `docker compose` (V2).
- **Version**: Compose V2 (plugin or standalone `docker compose`).
- **Verify**:
  ```bash
  docker compose version
  ```
- **Important**: Use `docker compose` (with a space), not `docker-compose` (hyphen). If only the hyphenated command exists, install the Compose V2 plugin or standalone.

### 3. Git

- **Purpose**: Clone ACE repositories and optional local-env (if not already present).
- **Verify**:
  ```bash
  git --version
  ```

### 4. Node.js (optional on host, required in containers)

- **Purpose**: App containers use **Node 20** (image `node:20`). If you run any service manually on the host (e.g. for debugging), use a compatible Node version.
- **Version**: Node 20.x (LTS). Check each repo's `.nvmrc` or `package.json` engines if different.
- **Verify** (optional):
  ```bash
  node --version   # should be v20.x if you run apps on host
  yarn --version   # or npm --version
  ```

### 5. Enough resources

- **Purpose**: The full stack runs many containers (databases, Redis, Mongo, LocalStack, several Node apps, optional LLM/LiteLLM, Jira/nginx).
- **Recommendation**: At least 4 GB RAM for a minimal set (database + dashboard); 8 GB or more for full stack including LLM and bots. Sufficient disk for images and volumes (PostgreSQL, Redis data).
- **Verify**: Ensure Docker has enough memory and disk (Docker Desktop: Settings → Resources; Linux: system free memory and disk).

---

## Optional but useful

- **AWS CLI**: If you use real AWS SQS or Secrets Manager for local (e.g. to fetch dev secrets), or run the letsencrypt container for Jira HTTPS (Route53 DNS challenge). Not required if using only LocalStack and local `.env`.
- **kubectl**: Not required for local-env; only for EKS (dev/stg/prod).
- **pgAdmin / Redis Insight**: Provided as services in Compose (pgadmin on 3080, redisinsight on 3540). No need to install on host unless you prefer a local client.

---

## Summary checklist

Before proceeding to [02-repositories-and-directory-layout.md](./02-repositories-and-directory-layout.md):

- [ ] `docker --version` works and `docker run hello-world` succeeds.
- [ ] `docker compose version` works (Compose V2).
- [ ] `git --version` works.
- [ ] (Optional) Node 20 and yarn/npm available if you will run services on the host.
- [ ] Docker has enough memory and disk for the stack.

If any check fails, install or fix that component before continuing. See [08-troubleshooting.md](./08-troubleshooting.md) for common Docker/Compose issues.
