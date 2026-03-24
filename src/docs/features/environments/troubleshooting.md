# Troubleshooting – Environments

This document lists **common issues** and **per-environment differences** when setting up or running ACE. Use it together with the relevant env doc (local, development, staging, demo, production) and service docs.

---

## Common issues

### Redis connection failed

- **Symptom**: Service (backend, bot, ops-scheduler) cannot connect to Redis; connection refused or timeout.
- **Local**: Ensure Redis is running (e.g. `docker-compose up -d redis` in local-env). Check `REDIS_HOST` and `REDIS_PORT` (e.g. `localhost` and `6379`, or service name `redis` if in same compose network).
- **EKS**: Check that Redis is deployed in the same namespace and that `REDIS_HOST` points to the K8s service name (e.g. `redis-deployment` or the actual service name). Verify the Redis pod is Running and the service exists (`kubectl get svc -n <namespace>`).
- **Docs**: See [local.md](./local.md), [env-vars-and-secrets.md](./env-vars-and-secrets.md), and the bot/backend service docs.

### DB Gateway unreachable

- **Symptom**: Backend (or caller) gets connection refused, timeout, or 502 when calling DB Gateway.
- **Local**: Ensure ace-db-gateway is running and that `ACE_GATEWAY_URL` / `DB_GATEWAY_URL` in the backend matches the gateway URL (e.g. `http://db-gateway-service:port` or `http://localhost:4xxx`). Check docker-compose network and port mapping.
- **EKS**: Verify DB Gateway pod is Running in the same (or reachable) namespace; base URL must point to the K8s service. Check Ingress if calling from outside the cluster.
- **Docs**: [../architecture/integrations.md](../architecture/integrations.md), [env-vars-and-secrets.md](./env-vars-and-secrets.md).

### CORS or wrong base URL

- **Symptom**: Frontend gets CORS errors or calls go to the wrong API host.
- **Cause**: Frontend is built or configured with the wrong API base URL (e.g. `VITE_API_URL`). In SPA builds, the value is often baked in at build time.
- **Fix**: Set the correct env var for the environment (local vs dev/stg/prod) before building or ensure the deploy pipeline injects the right value. See [env-vars-and-secrets.md](./env-vars-and-secrets.md) and frontend repo docs.

### Login or JWT validation fails

- **Symptom**: 401 on login or on authenticated requests.
- **Checks**: Backend and DB Gateway must use the same JWT secret (or validation source). Credentials in DB/identity store must be correct. Token expiry and clock skew can cause 401.
- **Docs**: [../architecture/data-flow.md](../architecture/data-flow.md), [../architecture/security.md](../architecture/security.md), [../rules/security-rules.md](../rules/security-rules.md).

### Pod not starting or CrashLoopBackOff (EKS)

- **Checks**: `kubectl describe pod <name> -n <namespace>` for events; `kubectl logs <pod> -n <namespace>` for application errors. Common causes: missing or wrong env vars (secrets not mounted), wrong image tag, failed health check, resource limits.
- **Secrets**: Ensure the secret path (e.g. `ace/dev/<service>-secrets`) exists in Secrets Manager and is mounted or injected as env vars as expected by the deployment.

### "You are not associated with this project. Please contact an ACE Administrator." (Slack / ACE APP)

- **Symptom**: In Slack, when a user mentions **@ACE** (or interacts with the ACE app), the ACE APP replies with: *"You are not associated with this project. Please contact an ACE Administrator."*
- **Cause**: The user is **not registered or associated with the project** that owns the Slack channel. ACE only allows access when the Slack user is linked to that project in the dashboard.

**Fix (step-by-step)** — An ACE administrator should do the following:

1. **Log in** to the ACE dashboard (same environment as the Slack workspace where the error appeared).

2. **Find the project linked to the channel** (if you don’t know it yet):
   - In the sidebar, go to **Admin Panel** → **Channel Management** (`/admin-panel/channel-management`).
   - The list shows each Slack channel and its **Project** (and optionally Client).  
   - To find by channel: in Slack, get the **channel ID** (e.g. right‑click the channel → **View channel details** or copy the channel link; the ID is in the URL, e.g. `C082Q3XEXT7`). In Channel Management, use the **search** box and type that ID (or the channel name, if names are loaded). The row shows the **project name** (and ID) linked to that channel.  
   - Note the **project name** (and ID if visible) for the next step.

3. **Link the user to that project** (choose one of the following):

   **Option A — Via Project Management (edit project users)**  
   - Go to **Admin Panel** → **Project Management** (`/admin-panel/project-management`).  
   - Find the project from step 2 (use the search/filter if needed).  
   - **Edit** the project (e.g. click the project row or the edit action).  
   - In the project form, open the **Users** (or “Project users”) section.  
   - **Add** the user who received the error (search by name or email if the list is large).  
   - **Save** the project. The backend updates the project–user association.

   **Option B — Via External Linkages / Add External Users** (if the user is managed there)  
   - Go to **Admin Panel** → **External Linkages** (Super User) or **Add External Users** (other admins).  
   - Find the user (e.g. by email or name).  
   - Open the “Manage” / “Linkages” view for that user and **add the project** (the one from step 2) to the user. Save.

4. Ask the user to try **@ACE** again in the same Slack channel. If the user is now linked to the project that owns the channel, the error should stop.

If the error persists, confirm that the channel is really linked to that project in **Channel Management** and that the Slack account is the one associated with the dashboard user (e.g. same email).

---

## Per-environment differences

| Aspect | Local | Development (dev) | Staging (stg) | Demo | Production (prod) |
|--------|--------|--------------------|---------------|------|--------------------|
| **Cluster** | N/A | development-ace-eks | development-ace-eks | development-ace-eks | production-ace-eks |
| **Namespace** | N/A | dev | stg | demo | prod, apis, ace-system |
| **Base URLs** | localhost / compose service names | Dev ingress URLs | Stg ingress URLs | Demo URLs | Prod ingress URLs |
| **Secrets** | .env / docker-compose | ace/dev/... | ace/stg/... | ace/demo/... | ace/prod/... |
| **Logging** | Often verbose (debug) | As configured | As configured | As configured | Usually info; avoid verbose in prod |
| **Feature flags** | Per .env or default | Per deploy config | Per deploy config | Per deploy config | Production values only |
| **Approvals** | N/A | As defined | Often required | As defined | Required |

Use the correct cluster, namespace, and URLs for each environment; mixing them causes wrong backend, DB, or secrets. **Demo** uses **production** resources (RDS, Redis, Gateway, SQS) for shared services—see [demo.md](./demo.md).

---

## Where to look

- **Local**: Docker Compose logs (`docker-compose logs <service>`), service dev server output, `.env` (do not commit).
- **EKS**: `kubectl logs`, `kubectl describe pod`, CloudWatch (if configured), and the monitoring namespace. See ace-infra and [production.md](./production.md) for monitoring and rollback.
- **Service-specific**: Each service’s `docs/` (setup, debugging, architecture) in its repo. See [../services/](../services/).
- **Infra rules**: [../rules/infrastructure-rules.md](../rules/infrastructure-rules.md), [../rules/security-rules.md](../rules/security-rules.md).

---

## Links

- **Local** — [local.md](./local.md)
- **Development** — [development.md](./development.md)
- **Staging** — [staging.md](./staging.md)
- **Demo** — [demo.md](./demo.md)
- **Production** — [production.md](./production.md)
- **Env vars and secrets** — [env-vars-and-secrets.md](./env-vars-and-secrets.md)
- **Architecture** — [../architecture/](../architecture/) (data-flow, integrations, deployment, security)
