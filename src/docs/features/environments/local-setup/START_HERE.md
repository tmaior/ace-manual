# Start Here – Local Environment Setup

This folder contains **step-by-step documentation** to bring up a local ACE environment that behaves like **local-env** (Docker Compose). Follow the documents in order so that the result is equivalent to the team's local setup.

---

## Files

**[README.md](./README.md)**  
Overview of this folder: purpose, who it is for (including AI agents), and how to use the guide. Read this first for context.

**[01-prerequisites.md](./01-prerequisites.md)**  
What must be installed before starting: Docker, Docker Compose, Node.js version, Git, and optional tools. Includes version requirements and how to verify each.

**[02-repositories-and-directory-layout.md](./02-repositories-and-directory-layout.md)**  
Which ACE repositories to clone, where to put them (common parent directory, `ACE_ROOT`), and how paths in docker-compose must match. Critical for avoiding "path not found" and volume mount failures.

**[03-docker-compose-and-configuration.md](./03-docker-compose-and-configuration.md)**  
Where the Compose file lives (local-env), init-scripts (LocalStack SQS), nginx/Jira volumes, and LiteLLM config. How to obtain or copy these files and what to adapt (e.g. `ACE_ROOT`).

**[04-environment-variables.md](./04-environment-variables.md)**  
List of environment variables by service, which are required vs optional, where to get values (.env, team, Secrets Manager for reference). Placeholders only; never commit real secrets.

**[05-startup-sequence.md](./05-startup-sequence.md)**  
Exact order to start services: database first, then LocalStack (and init-scripts), then configuration, then core apps, then bots/scheduler/LLM. Includes commands and how long to wait between steps.

**[06-profiles-and-services-reference.md](./06-profiles-and-services-reference.md)**  
Docker Compose profiles (`database`, `apps`, `dashboard`, `all`, etc.) and which services belong to each. Use this to choose minimal vs full stack and to understand service dependencies.

**[07-health-checks-and-validation.md](./07-health-checks-and-validation.md)**  
How to verify that the environment is working: health endpoints, expected URLs, database and Redis checks. Defines "success" for the local setup.

**[08-troubleshooting.md](./08-troubleshooting.md)**  
Common failures (wrong paths, Redis/DB not ready, CORS, wrong host/port), how to diagnose (logs, docker compose ps), and fixes. Use this when something does not start or respond.

**[09-hostname-and-dns.md](./09-hostname-and-dns.md)**  
Which hostname to use: `localdash.ace.ezops.cloud` is for a developer's personal machine; agents must use the Daytona proxy URL or **`ace-development.ace.ezops.cloud`**, which can be registered in Route53 (hosted zone ace.ezops.cloud). Read this when configuring URLs for the local stack.

---

## Recommended order for agents

1. Read [README.md](./README.md).
2. Follow [01-prerequisites.md](./01-prerequisites.md) and verify every item.
3. Follow [02-repositories-and-directory-layout.md](./02-repositories-and-directory-layout.md) to clone repos and set `ACE_ROOT`.
4. Follow [03-docker-compose-and-configuration.md](./03-docker-compose-and-configuration.md) to ensure Compose and config files are in place and paths are correct.
5. Follow [04-environment-variables.md](./04-environment-variables.md) to prepare `.env` (or equivalent) without committing secrets.
6. Read [09-hostname-and-dns.md](./09-hostname-and-dns.md) and use **ace-development.ace.ezops.cloud** (or the Daytona proxy URL), not localdash; register in Route53 if using ace-development.
7. Follow [05-startup-sequence.md](./05-startup-sequence.md) to start services in the correct order.
8. Use [07-health-checks-and-validation.md](./07-health-checks-and-validation.md) to confirm the environment works.
9. If something fails, use [08-troubleshooting.md](./08-troubleshooting.md) and [06-profiles-and-services-reference.md](./06-profiles-and-services-reference.md) for reference.

---

*The result of following this guide should be a running local ACE stack equivalent to **local-env**. For a single-page summary of local setup, see [../local.md](../local.md).*
