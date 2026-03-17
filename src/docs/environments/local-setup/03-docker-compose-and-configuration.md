# Step 3: Docker Compose and Configuration

This step ensures the Compose file and all supporting config files are in place and that paths match your [directory layout](./02-repositories-and-directory-layout.md).

---

## Main Compose file

- **Location**: `local-env/docker-compose.yaml` (i.e. `ACE_ROOT/local-env/docker-compose.yaml`).
- **Role**: Defines all services (PostgreSQL, Redis, Mongo, LocalStack, configuration, db-gateway, slackbot, commands-api, ops-scheduler, ops-bot, llm, dash-back, dash-front, litellm, optional jira/nginx/letsencrypt), their profiles, ports, environment variables, volumes, and build contexts.

If you do not have a `local-env` folder, you must obtain or recreate this file. The structure is documented in [../local.md](../local.md) and a reference subset is included there; the canonical source is the team's local-env repo or folder.

---

## Paths in the Compose file

The Compose file uses **bind mounts** so that each Node app runs from its repo on the host. Example:

```yaml
volumes:
  - /home/admin/ace/ace-configuration:/app
```

If your `ACE_ROOT` is not `/home/admin/ace`, you have two options:

1. **Replace all absolute paths** in `docker-compose.yaml` with your `ACE_ROOT` (e.g. `/Users/me/ace/ace-configuration`). Do a find-and-replace for the prefix (e.g. `/home/admin/ace` → `$ACE_ROOT` is not valid in YAML; you must use the real path or an env var if the Compose file supports it).
2. **Use variable substitution**: If the Compose file already uses something like `${ACE_ROOT}/ace-configuration`, then create a `.env` file in `local-env/` with:
   ```bash
   ACE_ROOT=/path/to/your/ace
   ```
   and ensure Docker Compose reads it (Compose automatically loads `.env` from the project directory). If the current Compose has hardcoded paths, you must edit it to use `ACE_ROOT` for every repo path and then set `ACE_ROOT` in `.env`.

**Build contexts** (commands-api, llm) also use absolute paths in the current setup, for example:

- commands-api: `context: /home/admin/ace/ace-infra`, `dockerfile: /home/admin/ace/ace-infra/ace-commands-api/local-ace.commands-api.Dockerfile`
- llm: `context: /home/admin/ace/ezrael-bot-llm`, `dockerfile: /home/admin/ace/ace-infra/ace-llm/ace.llm.Dockerfile3`

Replace these with your `ACE_ROOT` or the same variable if you introduce `ACE_ROOT`.

---

## Environment file: local-env/.env

- **Location**: `local-env/.env`.
- **Role**: Overrides for Compose: e.g. `ACE_ROOT`, `POSTGRES_PASSWORD`, `UID`, `GID`, `CERT_DOMAIN`, `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, and any secret or per-machine variable. **Do not commit real secrets.**
- **Action**: Create `.env` from a template or from [04-environment-variables.md](./04-environment-variables.md). Set at least:
  - `ACE_ROOT` (if the Compose file uses it)
  - `POSTGRES_PASSWORD` (and any other DB passwords if not hardcoded in Compose)
  - Tokens and API keys required by the services you will start (Slack, LLM, OAuth, etc.)

---

## Init scripts (LocalStack)

- **Location**: `local-env/init-scripts/`. The Compose file mounts this directory into LocalStack as `/etc/localstack/init/ready.d`.
- **Role**: Scripts in `ready.d` run when LocalStack is ready. The main script is **`sqs.sh`**, which creates the SQS FIFO queues used by ACE (e.g. `ResourceHealthChecks-local.fifo`, `ResourceHealthCheckResults-local.fifo`, `DocsSync-local.fifo`, `LocalCommandsQueue.fifo`, `LocalCommandsOutputQueue.fifo`).
- **Action**: Ensure `local-env/init-scripts/sqs.sh` exists and is executable (`chmod +x sqs.sh`). If you created local-env from scratch, copy this script from the team's local-env or from the content in [../local.md](../local.md) (init scripts section). Without it, ops-scheduler and commands-api will not find the queues when using LocalStack.

---

## LiteLLM config (optional but recommended for LLM)

- **Purpose**: The **litellm** service uses a config file to define models and API keys (Anthropic, OpenRouter, etc.).
- **Location**: Often at repo root or in local-env, e.g. `ACE_ROOT/litellm-config.yaml` or `local-env/litellm-config.yaml`. The Compose file mounts it into the litellm container (e.g. as `/app/config.yaml`).
- **Action**: If you start the `llm` and `litellm` services, you need a valid `litellm-config.yaml`; structure and placeholders are in [../local.md](../local.md). Do not commit real API keys; use env vars or inject at runtime.

---

## Jira integration (optional)

- **jira** service: Usually the app lives at `local-env/jira-app/meu-app-connect` (mounted as `/app`). If you do not have this folder, you can skip the `jira` and `nginx` services (do not use profile `jira` or ensure they are not started).
- **nginx**: Uses `local-env/volumes/nginx/` (certs and `nginx.conf.template`). Needed only if you expose Jira over HTTPS locally.
- **letsencrypt**: One-shot container to generate wildcard cert (profile `cert`); requires `CERT_DOMAIN` and AWS credentials for Route53. Skip if you are not using Jira over HTTPS.

---

## Checklist before starting services

- [ ] `local-env/docker-compose.yaml` exists and all repo paths (and build contexts) match your `ACE_ROOT` or `.env` `ACE_ROOT`.
- [ ] `local-env/.env` exists with required variables (see [04-environment-variables.md](./04-environment-variables.md)); no real secrets committed.
- [ ] `local-env/init-scripts/sqs.sh` exists and is executable (for LocalStack SQS).
- [ ] (If using LLM/litellm) LiteLLM config file exists and is mounted as in Compose.
- [ ] (If using Jira) `local-env/jira-app/meu-app-connect` and nginx volumes exist; optionally cert generated for HTTPS.

Next: [04-environment-variables.md](./04-environment-variables.md) to fill in `.env` and required env vars.
