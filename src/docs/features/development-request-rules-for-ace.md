# Development requests: rules ACE must read first

**Audience**: **ACE** (and any automated agent) using this repository as the **project docs** knowledge base. Human readers can use this as a checklist too.

**Location**: This file lives at the **docs root** (`src/docs/features/` in ace-manual) so it appears alongside architecture, environments, and `rules/` when the system lists documentation paths. The **file name** includes **development** and **rules** so routing logic that matches the user’s request to doc paths should prioritize it for implementation work.

---

## When this applies (mandatory)

Treat the user request as a **development request** whenever it involves **any** of the following (non-exhaustive):

- Implementing, changing, or fixing **behavior** in an ACE codebase (backend, frontend, bots, gateways, Jira app, schedulers, etc.).
- Adding or changing **APIs**, **schemas**, **migrations**, **configs**, or **env vars** that affect runtime.
- **Infrastructure as code** or **Kubernetes/Helm** changes for ACE (e.g. ace-infra).
- **Refactors** or **cross-repo** changes tied to a product or ops goal.
- Preparing work that will result in **commits and PRs** to ACE repositories.

**Rule**: For every development request, ACE **must read this file**, then **read the rule documents linked in the next section** (at least the **minimum set**), **before** writing a plan, creating Jira issues, or editing code. Skipping these reads is non-compliant.

Purely informational questions (e.g. “what is ace-db-gateway?”) with **no** intended code or infra change do not require the full minimum set; still follow [rules/main-rules.md](./rules/main-rules.md) (no invention—read or ask).

---

## Minimum reading set (development requests)

Read **in order** (or in parallel, but all must be covered before execution planning):

1. [rules/main-rules.md](./rules/main-rules.md) — non-negotiable principles; English for code/docs; document changes; do not invent.
2. [rules/jira-led-development-planning.md](./rules/jira-led-development-planning.md) — **plan end-to-end**, map impacted apps, **create Jira issues for the plan**, **wait for human acceptance in Jira**, then implement **per issue**.
3. [rules/ai-agent-ace-workflow.md](./rules/ai-agent-ace-workflow.md) — feature branches, push discipline, secrets, local validation, PR timing.
4. [rules/development-rules.md](./rules/development-rules.md) — branches from `development`, testing, service `docs/`.
5. [rules/gitflow-rules.md](./rules/gitflow-rules.md) — branch names, protected branches, repo-specific flow.

**Then**, as the task requires:

- PRs and templates: [rules/pr-rules.md](./rules/pr-rules.md)
- Security: [rules/security-rules.md](./rules/security-rules.md)
- Testing expectations: [rules/testing-rules.md](./rules/testing-rules.md)
- Documentation placement: [rules/documentation-rules.md](./rules/documentation-rules.md)
- Infra/AWS/K8s: [rules/infrastructure-rules.md](./rules/infrastructure-rules.md)

Full directory index and short descriptions: [rules/START_HERE.md](./rules/START_HERE.md).

---

## Jira and development (do not skip)

For development requests, **[rules/jira-led-development-planning.md](./rules/jira-led-development-planning.md) is part of the default workflow**: understand the ask, map **all** relevant ACE services/repos, record the plan in **Jira** (parent + children or linked issues), **stop until the human accepts in Jira**, then execute according to each issue.

If the user **explicitly** opts out for a one-off task (e.g. “no Jira, emergency doc-only”), follow their instruction for **that** task only and state the exception in the response.

---

## After reading rules: use the rest of the docs root

- **Which services exist and how they connect**: [architecture/](./architecture/), [services/](./services/).
- **Local and deployed environments**: [environments/](./environments/).
- **ace-infra, Terraform, AWS, Kubernetes**: [infrastructure/](./infrastructure/).
- **Clone URLs and repos**: [repositories.md](./repositories.md).

---

## Summary for ACE routing

| User intent | Action |
|-------------|--------|
| Development / implementation / fix / feature / infra code change | Read **this file**, then **minimum set** above; include **Jira-led planning** unless user explicitly opts out. |
| Explain-only / no code or config change | [rules/main-rules.md](./rules/main-rules.md); pull detail from [services/](./services/) or [architecture/](./architecture/) as needed. |

This document does not replace the linked rule files; it **forces** them to be loaded for development work so behavior matches project standards and Jira gating.
