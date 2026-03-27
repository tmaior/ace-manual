# Rules

This directory is the **central place for all project rules, conventions, and standards** that govern how the ACE system is developed, documented, and operated. The rules here apply to everyone working on ACE—developers, reviewers, and AI agents—and are the main reference for "how we do things" across repositories and services.

---

## Why this directory exists

- **Single source of truth**: Instead of scattering rules across wikis, chats, or tribal knowledge, the `rules/` folder holds the canonical description of our processes and standards. When in doubt, the answer is here (or in a service's own `docs/` for repo-specific details).
- **Consistency**: By documenting branch strategy, PR format, naming, documentation requirements, and infrastructure conventions in one structure, we keep behavior consistent across the many ACE services and repos.
- **Onboarding and automation**: New team members and AI agents can read these docs to understand what is required before opening a PR, merging code, or changing infra. The same rules are used to guide automated workflows and agent behavior.
- **Living documentation**: These rules are updated when we change how we work. When you change a process or convention, you update the corresponding rule file and, if needed, the [main rules](./main-rules.md) or this README.

---

## What you will find here

Each markdown file in this directory describes one area of rules. All of them are written in **English** and include both human-readable guidance and short **"Agents"** notes for AI assistants.

| Document | What it covers |
|----------|----------------|
| [main-rules.md](./main-rules.md) | **Core rules that always apply**: English for code and docs, document every change, use the right doc structure, keep index/README/START_HERE in sync, respect standardizations, read rules before acting, and never invent—read or ask. Start here for the non‑negotiable principles. |
| [ai-agent-ace-workflow.md](./ai-agent-ace-workflow.md) | **AI agents in sandboxes** (e.g. Daytona): feature branches, **push to remote** so work survives session loss, use **project secrets** for GitHub/AWS without unnecessary prompts, **run local-env** and **smoke-test** before claiming done, share **URLs**, open **PR** after user confirms. Read first for agent-driven ACE implementation. |
| [application-requirements.md](./application-requirements.md) | **Application requirements**: Rules, standardizations, behaviors, and procedures that **every** ACE application must follow. Single entry point for including a new app and for ensuring all apps meet code, docs, security, infra, and deployment requirements. |
| [pr-rules.md](./pr-rules.md) | **Pull requests**: Branch flow and base branches, which PR template to use for each branch pair, PR body structure (features, improvements, bug fixes, testing, dependencies, etc.), how to generate `PR.md`, using `gh pr create`, pre-PR checklist, and PR review process (what reviewers check, approval before merge). |
| [development-rules.md](./development-rules.md) | **Development workflow**: Feature branches from latest `development`, local testing before commit/push (feature branches + development for unchanged apps), merging into `development`, using and updating each service's `docs/`, creating new docs within the rules, and following existing standardizations (e.g. pagination like the rest of the app). |
| [infrastructure-rules.md](./infrastructure-rules.md) | **Infrastructure and AWS**: Region (us-east-1), mandatory resource tags (Project=ACE, Environment=<ENV>-ACE), analyzing options before creating resources, Kubernetes/EKS usage, secrets (env vars via `ace/<env>/<service>-secrets`), CI/CD and deployments, and the ace-infra repository structure. |
| [documentation-rules.md](./documentation-rules.md) | **Documentation**: Document everything that is done; when and where to document (central vs service `docs/`); structure and kebab-case file names; types of docs (setup, architecture, API, debugging); quality and working examples; docs in the same PR as the change; service `docs/` as the starting point and kept up to date. |
| [standardization-rules.md](./standardization-rules.md) | **Standardizations**: Language (English, no emojis, TypeScript); naming (camelCase, PascalCase, kebab-case for branches/docs); APIs and URLs (env vars, no hardcoding); Conventional Commits; error handling and logging; security (validation, JWT); doc structure; infra tags and naming; and the rule to **reuse existing patterns** instead of inventing new ones. |
| [gitflow-rules.md](./gitflow-rules.md) | **Gitflow**: Branch naming (kebab-case), long-lived branches (development, staging, main, production for frontend), feature/bugfix/hotfix branches, repository-specific flow (most repos vs ace-dashboard-frontend dual PRs **staging→main** and **staging→production**), Conventional Commits, no direct push to protected branches, and keeping branches up to date before opening a PR. |
| [security-rules.md](./security-rules.md) | **Security**: Input validation, JWT on protected endpoints, secrets (no commit/hardcode), safe error messages, rate limiting for bots, permissions and authorization. |
| [testing-rules.md](./testing-rules.md) | **Testing**: Simple behavior-based tests (URLs, UI, forms, DB checks) before PR; no unit tests or committed test code; optional local shell scripts (not committed); what to document in the PR Testing section. |

---

## How to use this directory

### If you are new to the project

1. Read **[main-rules.md](./main-rules.md)** first. It defines the principles that override everything else (language, documentation, structure, no invention).
2. If you are an **AI agent** (or run in an **ephemeral sandbox** such as Daytona), read **[ai-agent-ace-workflow.md](./ai-agent-ace-workflow.md)** next for branches, push cadence, secrets, local validation, and PR handoff.
3. If you are **adding or auditing an application**, read **[application-requirements.md](./application-requirements.md)** for the rules that every app must follow.
4. Then read **[gitflow-rules.md](./gitflow-rules.md)** and **[development-rules.md](./development-rules.md)** so you know how to branch, test, and merge.
5. Before opening a PR, read **[pr-rules.md](./pr-rules.md)** and use the correct template and description format.
6. When writing code or docs, use **[standardization-rules.md](./standardization-rules.md)** for naming, commits, APIs, and patterns; use **[documentation-rules.md](./documentation-rules.md)** for where and how to document.
7. If you touch infrastructure or deployment, read **[infrastructure-rules.md](./infrastructure-rules.md)**.
8. For security (validation, auth, secrets), read **[security-rules.md](./security-rules.md)**. For how to test (simple checks, no unit tests), read **[testing-rules.md](./testing-rules.md)**.

### If you are an AI agent

- For **implementation work** in ACE repos from a **sandbox or automated environment**, read **[ai-agent-ace-workflow.md](./ai-agent-ace-workflow.md)** first (push discipline, secrets, local validation, PR timing), then the rest as below.
- At the **start of every task**, determine which rules apply (main, AI agent workflow, application requirements, PR, development, documentation, standardization, gitflow, infrastructure, security, testing) and read the relevant files. Do not invent behavior or conventions; follow what is written here and in the service `docs/`.
- When implementing a **feature** (e.g. pagination, a new endpoint), search the codebase for existing implementations and **reuse the same pattern** (see [standardization-rules.md](./standardization-rules.md) and [development-rules.md](./development-rules.md)).
- When **creating or moving documentation**, follow [documentation-rules.md](./documentation-rules.md) and [main-rules.md](./main-rules.md) (structure, kebab-case, index/README/START_HERE).
- When **creating a PR**, follow [pr-rules.md](./pr-rules.md) (template, `PR.md`, `gh pr create`). When **changing code**, plan the doc update in the same PR ([documentation-rules.md](./documentation-rules.md)).

### If you are changing how we work

- Update the **relevant rule file** (and this README if the list of documents or their purpose changes). Put the doc update in the **same PR** as the process or convention change.
- If the change affects several areas (e.g. a new branch strategy and a new PR template), update all affected rule files and the central [index](../index.md) / [README](../README.md) / [START_HERE](../START_HERE.md) as required by [main-rules.md](./main-rules.md).

---

## Relationship with other rule sources

- **`.cursor/rules/`** (e.g. `ace-project-01.mdc`, `ace-core-rules.mdc`): These files are applied automatically in the Cursor IDE and summarize structure, gitflow, and critical patterns. The **detailed** and **narrative** version of the rules lives here (e.g. in ace-manual: `src/docs/features/rules/`). Keep both aligned: when you change a process, update the rule file in `rules/` and, if needed, the corresponding `.cursor/rules` file.
- **Service `docs/`** (e.g. `ace-db-gateway/docs/`): Each service can have its own rules and conventions (API style, folder layout, repo-specific workflows). This `rules/` directory describes **cross-repo** and **project-wide** rules. Service-level rules must not contradict these; they add or specialize.

---

## Summary

- **Purpose**: One place for all ACE project rules so that humans and agents know how to develop, document, and operate the system consistently.
- **Contents**: Main rules, **AI agent ACE workflow**, PR rules, development rules, infrastructure rules, documentation rules, standardization rules, gitflow rules, security rules, and testing rules—each in its own file with clear sections and agent-oriented notes.
- **How to follow**: Read the main rules first, then the rules that apply to your task (gitflow, development, PR, documentation, standardization, infrastructure). When adding something that already exists elsewhere, reuse the existing pattern. When changing a process, update the corresponding rule file and entry points (index, README, START_HERE) as required.

For the overall documentation layout and entry points, see [../README.md](../README.md) and [../START_HERE.md](../START_HERE.md).
