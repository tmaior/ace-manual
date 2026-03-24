# Hostname and DNS for Local Environment

Services in the local stack (frontend, API, DB Gateway, LLM, LocalStack) need a hostname that resolves to the machine where Docker is running, so that the browser and callbacks (e.g. from Slack or external tools) can reach the correct URLs. This document explains the two typical setups: **developer (personal)** and **agent / automated (Daytona or shared hostname)**.

---

## Two use cases

### 1. Developer (personal machine)

- **Hostname**: **`localdash.ace.ezops.cloud`**
- **Usage**: A developer points this hostname to their own machine's IP (e.g. via DNS or `/etc/hosts`). Used for personal local testing.
- **Setup**: Either register the A record in Route53 (pointing to the developer's IP) or add `127.0.0.1 localdash.ace.ezops.cloud` to `/etc/hosts` on that machine.
- **Note**: This is the hostname you may see in the team's local-env Compose or env examples; it is **not** the one an automated agent should assume.

### 2. Agent / automated environment (Daytona or shared URL)

- **Hostname**: Use the URL configured in the **Daytona proxy** (if the agent runs in Daytona), or use **`ace-development.ace.ezops.cloud`**.
- **Usage**: The AI agent (or any automated/CI local setup) should use a stable, reachable hostname that points to the environment where the stack runs (e.g. Daytona workspace or a dedicated dev VM).
- **Setup**:
  - **Option A — Daytona**: Use the hostname/URL provided by the Daytona proxy for the workspace. Configure all `VITE_*`, `FRONTEND_URL`, `API_HOST`, `LOCAL_AWS_ENDPOINT`, and queue URLs to use that hostname and the same ports (3042, 3041, 3031, 3040, 3066, etc.).
  - **Option B — ace-development.ace.ezops.cloud**: Create and use **`ace-development.ace.ezops.cloud`** as the public hostname for the agent's environment. Register it in **Route53**, in the hosted zone **`ace.ezops.cloud`**, pointing to the IP of the machine or proxy that serves the stack (e.g. Daytona proxy IP or the dev server IP). Then use `ace-development.ace.ezops.cloud` in the Compose file and `.env` for all URLs that the browser or external callers use (same ports as above).

---

## Route53 (ace.ezops.cloud)

- **Hosted zone**: **`ace.ezops.cloud`**
- **Agent hostname**: **`ace-development.ace.ezops.cloud`** can be registered in this zone.
- **Record**: Create an **A** record (or CNAME if appropriate) for `ace-development.ace.ezops.cloud` pointing to the IP (or target) of the environment where the local stack is reachable (e.g. Daytona proxy or dev VM). Ensure the machine/proxy exposes the same ports (3042, 3041, 3031, 3040, 3066) as in the Compose file.

Subdomains for specific services (e.g. Jira) can follow the same pattern (e.g. `jira.ace-development.ace.ezops.cloud`) if needed; document them in the Compose or env for that service.

---

## Summary for the agent

- **Do not** assume `localdash.ace.ezops.cloud` is your hostname; that is for a developer's personal machine.
- **Do** use either:
  1. The hostname/URL provided by the **Daytona proxy** for your workspace, or  
  2. **`ace-development.ace.ezops.cloud`**, after it is registered in Route53 (hosted zone `ace.ezops.cloud`) and points to your environment's IP/proxy.
- Set all public-facing URLs in the Compose file and `.env` to use that hostname with the correct ports (e.g. `http://ace-development.ace.ezops.cloud:3041` for the API, `:3042` for frontend, `:3031` for DB Gateway, `:3040` for LLM, `:3066` for LocalStack).

---

## See also

- [04-environment-variables.md](./04-environment-variables.md) — where these URLs are set (VITE_*, FRONTEND_URL, queue URLs, etc.).
- [../local.md](../local.md) — Host name and DNS section for local development in general.
