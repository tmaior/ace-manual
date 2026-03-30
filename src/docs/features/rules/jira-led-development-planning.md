# Jira-led development planning and execution

**Discovery**: ACE loads this file as part of the **development** rule set. The docs root entry that forces reading these rules (including this file) is [development-request-rules-for-ace.md](../development-request-rules-for-ace.md).

This document defines the **mandatory workflow** when a stakeholder asks ACE to **implement or change** the platform: the agent must **plan first**, **record the plan in Jira**, wait for **explicit human acceptance**, then **implement according to the Jira breakdown**.

It complements [ai-agent-ace-workflow.md](./ai-agent-ace-workflow.md) (branches, push, secrets, local validation, PRs), [development-rules.md](./development-rules.md), and [gitflow-rules.md](./gitflow-rules.md). When this workflow applies, it **defers bulk implementation** until after Jira acceptance, even if other docs speak generally about “implementation.”

**Agents**: For a short entry point and read order, see [AI-AGENT-QUICKSTART.md](./AI-AGENT-QUICKSTART.md). For a decision tree, see [golden-path-flowchart.md](./golden-path-flowchart.md).

---

## Critical rule (read before Phase E)

**Do not start bulk implementation until Phase D is complete.** After you create Jira issues in Phase C, you must **stop** and wait for human acceptance. Research-only activity (reading code and docs to build the plan) is allowed in Phases A–B; **feature branches, production-bound commits, and PRs for the scoped work** belong in Phase E **after** approval.

---

## 1. When this workflow applies

Use this workflow when the user (or ticket) requests **development** in the sense of: new behavior, bugfix across services, infra change, or any task that **touches one or more ACE repositories or services** and is not explicitly scoped as “doc-only” or “answer a question only.”

**Agents**: If the user says they want **planning in Jira first**, **review before coding**, or **break work into Jira issues**, follow this file **in addition to** [ai-agent-ace-workflow.md](./ai-agent-ace-workflow.md).

---

## 2. Phase A — Understand the request

1. **Restate the goal** in your own words: outcome, constraints, non-goals, and definition of done.
2. **Clarify ambiguities** with the user **before** creating Jira issues, unless the omission is harmless and explicitly called out as an assumption in the plan.
3. **Identify boundaries**: environments (local/dev/stg/prod), security or compliance constraints, and whether the change is **breaking** for APIs or consumers.

Do **not** start substantial repo edits in this phase beyond what is needed to **research** (read code, configs, existing docs).

---

## 3. Phase B — Map the ACE estate

1. **Enumerate impacted applications** using the project service catalog and repo layout (see [architecture/service-catalog.md](../architecture/service-catalog.md) and [repositories.md](../repositories.md) in ace-manual). Typical units include: `ace-stack-backend`, `ace-dashboard-frontend`, `ace-db-gateway`, `ace-infra`, bots, `ace-jira-integration`, `ace-ops-scheduler`, `ace-commands-api`, `ace-configuration`, `local-env`, and any other `ace-*` repo in scope.
2. **Per app**, record:
   - **What** must change (feature, fix, config, migration, chart, etc.).
   - **Where** (paths, modules, Terraform/K8s resources, env vars).
   - **Dependencies** on other apps (API contracts, events, shared secrets, deploy order).
3. **Cross-check** existing patterns: reuse conventions from each repo’s `docs/` and code; do not invent new patterns without documenting why in the Jira text.

Output of this phase is **internal coherence**: you can explain how the pieces fit together before anything is filed in Jira.

---

## 4. Phase C — Create Jira issues (plan as work items)

1. **Create issues in Jira** that **fully cover** the plan. Prefer a **parent** (Epic or equivalent) plus **children** (Stories/Tasks) **or** a small set of linked issues with clear **order and dependencies**.
2. **Each issue** must be **actionable**: title, description, acceptance criteria, affected repo/service names, and pointers to relevant docs or code areas when useful.
3. **Traceability**: The parent or a top-level issue should summarize the **end-to-end objective**; children should map to **concrete deliverables** (e.g. one issue per service or per deployable unit when sensible).
4. **Labels / fields** (recommended): use team-agreed labels such as `ace-plan` or `ace-execution` so reviewers can filter plan vs execution work; set **components** or **fix versions** per team practice.
5. **Do not** treat chat alone as the system of record for the plan once this workflow is in use: **Jira is the approval artifact**.

**Tooling**: Create issues via whatever is available and authorized in the environment (Jira REST API, Atlassian UI, automation, MCP). If creation is **blocked** (permissions, missing project key), stop, report the blocker, and do not pretend the plan was recorded in Jira.

### STOP after Phase C (hard gate)

**Important: do not start coding for the planned scope until Phase D is complete.**

| Do **not** (before Phase D approval) | Do **instead** |
|--------------------------------------|----------------|
| Write implementation code for the scoped work | Create Jira Epic/parent and Stories/tasks (or linked issues) that fully cover the plan |
| Create feature branches for that scope | Summarize Jira keys and links in the conversation |
| Open PRs for that scope | Tell the user planning is in Jira and you are **awaiting approval** to proceed |
| Assume chat approval replaces Jira | Wait for explicit human acceptance in Jira (and thread confirmation if your team requires it) |

**Example approval phrasing** (adjust to team norms): a comment on the parent Epic such as `Approved — ACE may proceed`, or transitioning issues to a **ready for implementation** status **plus** a short confirmation in the thread naming which keys to execute.

---

## 5. Phase D — Human acceptance in Jira

1. **Stop** after the issues are created (and linked). **Summarize** in the conversation: Jira keys, links, and what each issue covers.
2. The **human** reviews the breakdown in Jira (descriptions, scope, ordering). They may edit issues, re-scope, or reject part of the plan.
3. **Execution must not begin** until the human gives **explicit go-ahead** aligned with Jira, for example:
   - A **comment** on the parent Epic (or agreed anchor issue) such as “Approved — ACE may proceed,” **or**
   - Transitioning issues to a team-agreed status meaning **ready for implementation** (e.g. Ready / In Progress per team workflow), **combined with** a short confirmation in the thread naming the Epic or issue keys to execute.

If the human changes the Jira scope, **re-read** the updated issues and treat them as the new source of truth.

---

## Violation consequences (why the gate exists)

If you implement **before** Jira acceptance:

- **Rework risk**: The human may change scope, split issues, or reject part of the plan; your code may be discarded or require heavy rework.
- **Loss of early review**: Stakeholders lose the chance to correct direction **before** implementation cost is spent.
- **Process break**: The team cannot rely on Jira as the plan and approval record; traceability from ticket to PR suffers.

If you realize you started Phase E too early: **stop**, notify the user, align Jira to what was done or roll back per team practice, and **do not** open PRs for unapproved scope without explicit instruction.

---

## 6. Phase E — Execute per issue

1. Work **issue by issue** (or in an order agreed in Jira). For each issue:
   - Follow [ai-agent-ace-workflow.md](./ai-agent-ace-workflow.md): feature branches, push discipline, secrets, local validation.
   - Implement **only** what that issue describes; if gaps appear, **add or update Jira** (or ask the human) instead of silently expanding scope.
2. **Update issue status** and optionally **comment** with PR links, commit SHAs, or validation notes per team norms.
3. When **all** agreed issues for the plan are done, provide a **consolidated summary** (repos touched, PR URLs, tests run).

Opening PRs still follows [pr-rules.md](./pr-rules.md): typically open toward **`development`** after the implementation for each stream is ready and validated, unless the user or Jira specifies otherwise.

---

## 7. Relationship to PR timing

- [ai-agent-ace-workflow.md](./ai-agent-ace-workflow.md) says to open PRs after user confirmation of the **implementation**. Under **this** workflow, there are **two** gates:
  1. **Jira acceptance** of the **plan** (before major implementation).
  2. **User confirmation** of the **implementation** before PR (or per team habit, PR first then merge after review—follow project norms).

---

## 8. Agents — summary checklist

| Step | Action |
|------|--------|
| A | Understand and restate the request; resolve ambiguities. |
| B | Map all impacted ACE apps/repos; what/where/dependencies. |
| C | Create Jira issue tree (parent + children or linked issues); Jira is the plan record. |
| D | Pause; human accepts in Jira (+ thread confirmation if required). |
| E | Implement per issue; branches, tests, docs; PRs per [pr-rules.md](./pr-rules.md). |

---

## Summary

**Plan in Jira first, execute after human acceptance, one issue at a time.** This keeps large ACE changes reviewable in the same tool stakeholders already use and prevents the agent from coding ahead of an agreed breakdown.
