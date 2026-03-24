# Step 2: Repositories and Directory Layout

The local-env Docker Compose file uses **absolute host paths** to mount each ACE repository into its container. To bring up an environment equal to local-env, you must have all required repos cloned in a **common parent directory** and either use the same absolute path as in the Compose file or **replace every path** in the Compose file with your own path.

---

## Concept: ACE_ROOT

- **ACE_ROOT** (or "common parent directory") is the directory that contains all ACE repos and, typically, the **local-env** folder.
- **Example**: `/home/admin/ace` — so that `ace-configuration`, `ace-db-gateway`, `ace-stack-backend`, etc. live as siblings under `ace`, and `local-env` is `ace/local-env`.
- The Compose file in local-env references paths like `/home/admin/ace/ace-configuration`, `/home/admin/ace/ace-db-gateway`, etc. If your ACE_ROOT is different (e.g. `/Users/me/ace`), you **must** replace these paths in `docker-compose.yaml` (or use a `.env` with `ACE_ROOT` and ensure the Compose file uses `${ACE_ROOT}/ace-configuration`, etc.; see [03-docker-compose-and-configuration.md](./03-docker-compose-and-configuration.md)).

---

## Required repositories

Clone the following repositories as **siblings** under your chosen ACE_ROOT. Use the same names so that paths are predictable.

| Repository | Purpose in local-env |
|------------|------------------------|
| **ace-configuration** | configuration service (port 3030); DB and Redis config. |
| **ace-db-gateway** | db-gateway service (3031); central DB access. |
| **ace-stack-backend** | dash-back (3041); NestJS backend, JWT, dashboard API. |
| **ace-dashboard-frontend** | dash-front (3042); React/Vite dashboard. |
| **ace-slackbot** | slackbot (3033). |
| **ace-ops-bot** | ops-bot (3038). |
| **ace-commands-api** | commands-api (3035); built from ace-infra Dockerfile, app code mounted. |
| **ace-ops-scheduler** | ops-scheduler (3036); depends on LocalStack. |
| **ace-infra** | Dockerfiles (e.g. commands-api, llm) and optional K8s/manifests; build context for commands-api and llm. |
| **ezrael-bot-llm** | Build context for the **llm** service; Dockerfile in ace-infra (ace-llm). |

Optional (can be commented out or not started):

- **ace-sec-bot** — sec-bot (3037); often commented out in Compose.
- **ace-jira-integration** — Jira app may live inside local-env as `local-env/jira-app/meu-app-connect` (see local-env layout below).

---

## Where is local-env?

- **Option A**: local-env is a folder **inside** the same parent (e.g. `ACE_ROOT/local-env`). So structure is:
  ```
  ACE_ROOT/
  ├── local-env/           # docker-compose.yaml, .env, init-scripts, volumes
  ├── ace-configuration/
  ├── ace-db-gateway/
  ├── ace-stack-backend/
  ├── ace-dashboard-frontend/
  ├── ace-slackbot/
  ├── ace-ops-bot/
  ├── ace-commands-api/
  ├── ace-ops-scheduler/
  ├── ace-infra/
  ├── ezrael-bot-llm/
  └── ace-manual/           # this documentation repo
  ```
- **Option B**: local-env is in another repo or path. Then you must either copy `docker-compose.yaml`, `init-scripts/`, and any needed volumes into a folder under ACE_ROOT, or edit the Compose file so that all volume paths point to your ACE_ROOT (e.g. `ACE_ROOT/ace-configuration`).

The important point: **every volume path in docker-compose.yaml that points to an ACE repo must resolve to the correct absolute path on your machine.** If the Compose file has hardcoded paths (e.g. `/home/admin/ace/ace-configuration`), either:

1. Use the same path (create `/home/admin/ace` and clone there), or  
2. Find-and-replace in `docker-compose.yaml` all occurrences of `/home/admin/ace` with your `ACE_ROOT`, or  
3. Introduce an `ACE_ROOT` variable in the Compose file and set it in `.env` (see [03-docker-compose-and-configuration.md](./03-docker-compose-and-configuration.md)).

---

## Clone order and commands

Use your organization's clone URLs (GitHub, etc.). Example with a placeholder `GIT_BASE`:

```bash
export ACE_ROOT=/path/to/your/ace   # e.g. /home/admin/ace or $HOME/ace
mkdir -p "$ACE_ROOT" && cd "$ACE_ROOT"

git clone <GIT_BASE>/ace-configuration.git
git clone <GIT_BASE>/ace-db-gateway.git
git clone <GIT_BASE>/ace-stack-backend.git
git clone <GIT_BASE>/ace-dashboard-frontend.git
git clone <GIT_BASE>/ace-slackbot.git
git clone <GIT_BASE>/ace-ops-bot.git
git clone <GIT_BASE>/ace-commands-api.git
git clone <GIT_BASE>/ace-ops-scheduler.git
git clone <GIT_BASE>/ace-infra.git
git clone <GIT_BASE>/ezrael-bot-llm.git
# Optional:
# git clone <GIT_BASE>/ace-sec-bot.git
```

If local-env is a separate repo:

```bash
git clone <GIT_BASE>/local-env.git   # or wherever it lives
```

If local-env is not a repo but you have the Compose file and configs (e.g. from a colleague or from ace-infra), create the folder and put `docker-compose.yaml`, `init-scripts/`, and `.env` there (see [03-docker-compose-and-configuration.md](./03-docker-compose-and-configuration.md)).

---

## Verify layout

After cloning, verify that:

- `$ACE_ROOT/ace-configuration` exists and contains `package.json` (or equivalent).
- `$ACE_ROOT/ace-db-gateway`, `ace-stack-backend`, `ace-dashboard-frontend`, `ace-slackbot`, `ace-ops-bot`, `ace-commands-api`, `ace-ops-scheduler`, `ace-infra`, `ezrael-bot-llm` exist.
- `$ACE_ROOT/local-env/docker-compose.yaml` exists (or you have a copy and will fix paths).

Do **not** proceed to starting Compose until paths in `docker-compose.yaml` match this layout (see next document).
