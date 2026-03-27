# Start Here – Rules

This directory contains all project rules, conventions, and standards for the ACE system. Below is a list of each file with a short description of what you will find in it.

---

## Files

**[README.md](./README.md)**  
Overview of the rules directory: why it exists, what each document covers, and how to use it (for new contributors, AI agents, and when changing processes). Also describes the relationship with `.cursor/rules/` and service-level docs.

**[main-rules.md](./main-rules.md)**  
Core rules that always apply: English for code and docs, document every change, use the correct doc structure, keep index/README/START_HERE in sync, respect standardizations, read rules before acting, and never invent—read or ask.

**[ai-agent-ace-workflow.md](./ai-agent-ace-workflow.md)**  
Mandatory playbook for **AI agents** in **ephemeral sandboxes** (e.g. Daytona): feature branches, **push to remote** so commits survive session loss, use **project secrets** for GitHub/AWS proactively, run **local-env** and **smoke tests** before claiming done, share **URLs**, open **PR** after user confirmation.

**[application-requirements.md](./application-requirements.md)**  
Rules and procedures that **every** ACE application must follow: docs, code, APIs, security, logging, infrastructure, deployment, testing. Use when including a new application or auditing compliance.

**[pr-rules.md](./pr-rules.md)**  
Pull request flow, base branches, PR template selection by branch pair (including **staging → production** for ace-dashboard-frontend when applicable), PR body structure (features, improvements, bug fixes, testing, dependencies, etc.), generating `PR.md`, and using `gh pr create`. Includes pre-PR checklist and repository-specific notes.

**[development-rules.md](./development-rules.md)**  
Development workflow: feature branches from latest `development`, local testing before commit/push, merging into `development`, using and updating each service's `docs/`, creating new docs within the rules, and following existing standardizations (e.g. pagination like the rest of the app).

**[infrastructure-rules.md](./infrastructure-rules.md)**  
Infrastructure and AWS: region (us-east-1), mandatory resource tags (Project=ACE, Environment=<ENV>-ACE), analyzing options before creating resources, Kubernetes/EKS, secrets (env vars via `ace/<env>/<service>-secrets`), CI/CD and deployments, and ace-infra repository structure.

**[documentation-rules.md](./documentation-rules.md)**  
When and where to document; document everything that is done. Structure and kebab-case file names, types of docs (setup, architecture, API, debugging), quality and working examples, and keeping docs in the same PR as the change. Service `docs/` as the starting point.

**[standardization-rules.md](./standardization-rules.md)**  
Naming (camelCase, PascalCase, kebab-case), APIs and URLs (env vars, no hardcoding), Conventional Commits, error handling and logging, security (validation, JWT), doc structure, infra tags and naming. Rule to reuse existing patterns instead of inventing new ones.

**[gitflow-rules.md](./gitflow-rules.md)**  
Branch naming (kebab-case), long-lived branches (development, staging, main, production for frontend), feature/bugfix/hotfix branches, repository-specific flow (most repos vs ace-dashboard-frontend dual promotion **staging → main** and **staging → production**), Conventional Commits, no direct push to protected branches, and keeping branches up to date before opening a PR.

**[security-rules.md](./security-rules.md)**  
Input validation, JWT on protected endpoints, secrets (no commit or hardcode), safe error messages, rate limiting for bots, and permissions and authorization. Consolidates security requirements for all services.

**[testing-rules.md](./testing-rules.md)**  
Simple behavior-based testing only: calling URLs, viewing UIs, filling forms, checking the database. No unit tests or committed test code; optional local shell scripts for running checks (not committed). What to document in the PR Testing section.

---

## Subdirectories

This directory has no subdirectories.
