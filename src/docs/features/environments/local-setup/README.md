# Local Environment Setup Guide

This directory holds **detailed, step-by-step documentation** to bring up a local ACE environment that **behaves the same as local-env** (the team's Docker Compose setup). The documentation is written so that a human or an **AI agent** can follow it from scratch and end up with a working local stack.

---

## Purpose

- **Goal**: Produce a local ACE environment equivalent to **local-env**: same services, same ports, same dependencies (PostgreSQL, Redis, Mongo, LocalStack SQS, optional Jira/LLM).
- **Audience**: Developers and **AI agents** that need to run ACE locally without prior familiarity with the repo layout or Compose file.
- **Outcome**: After following the docs in order, the user/agent has a running stack that can be validated with the health checks described in this folder.

---

## What you will find

| Document | Purpose |
|----------|---------|
| [01-prerequisites.md](./01-prerequisites.md) | Install and verify Docker, Compose, Node, Git; optional tools. |
| [02-repositories-and-directory-layout.md](./02-repositories-and-directory-layout.md) | Clone which repos, where; `ACE_ROOT` and path requirements for Compose. |
| [03-docker-compose-and-configuration.md](./03-docker-compose-and-configuration.md) | Location of docker-compose.yaml, init-scripts, nginx/LiteLLM config; what to copy/adapt. |
| [04-environment-variables.md](./04-environment-variables.md) | Env vars per service; required vs optional; where to get values (no real secrets in docs). |
| [05-startup-sequence.md](./05-startup-sequence.md) | Order to start services and exact commands; wait times and dependencies. |
| [06-profiles-and-services-reference.md](./06-profiles-and-services-reference.md) | Compose profiles and which services they include; minimal vs full stack. |
| [07-health-checks-and-validation.md](./07-health-checks-and-validation.md) | How to verify the stack is up: endpoints, URLs, DB/Redis checks. |
| [08-troubleshooting.md](./08-troubleshooting.md) | Common errors, diagnosis, and fixes. |
| [09-hostname-and-dns.md](./09-hostname-and-dns.md) | Hostname for developer (localdash) vs agent (ace-development or Daytona); Route53 in ace.ezops.cloud. |

---

## How to use this guide

1. **Start at [START_HERE.md](./START_HERE.md)** for a short overview and the recommended order of documents.
2. **Follow the numbered docs in order** (01 → 02 → 03 → 04 → 05). Do not skip prerequisites or directory layout; wrong paths are a common cause of failure.
3. **Prepare environment variables** before starting all services (doc 04). Use placeholders or values from the team; never commit secrets.
4. **Run the startup sequence** (doc 05) and then **validate** with doc 07.
5. **If something fails**, use doc 08 (troubleshooting) and doc 06 (profiles/services) to narrow down the problem.

---

## Relationship to other docs

- **local-env**: The actual Docker Compose and config files live in the **local-env** folder (sibling to ace-* repos, often under the same parent as ace-manual). This guide **describes** how to use local-env and how to replicate its layout and behavior; it does not replace the need for the local-env directory and its contents.
- **[../local.md](../local.md)**: Single-page summary of local development (prerequisites, profiles, services, env vars). Use it for quick reference; use **this folder** for a full, ordered procedure.
- **[../env-vars-and-secrets.md](../env-vars-and-secrets.md)**: Where secrets and env vars come from per environment (local vs EKS). This folder's [04-environment-variables.md](./04-environment-variables.md) focuses on **local only** and lists vars needed for local-env.

---

## For AI agents

- **Read [rules/main-rules.md](../../rules/main-rules.md)** as required by the project; then use this folder as the **single source of steps** for bringing up the local environment.
- **Do not invent** paths, ports, or service names; they are defined in local-env and in these docs. If something is unclear, check [08-troubleshooting.md](./08-troubleshooting.md) and the main [../local.md](../local.md); if still unclear, ask the user.
- **Execute steps in order**: prerequisites → repos and paths → configuration → env vars → startup → health checks. Skipping steps (e.g. starting Compose before fixing `ACE_ROOT`) will cause failures that are documented in troubleshooting.
