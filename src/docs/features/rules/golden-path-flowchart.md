# Golden path: development requests (decision tree)

**Audience**: ACE and human contributors. Use this page to see **when to stop** and wait for Jira before coding.

**Related**: [jira-led-development-planning.md](./jira-led-development-planning.md), [AI-AGENT-QUICKSTART.md](./AI-AGENT-QUICKSTART.md), [development-request-rules-for-ace.md](../development-request-rules-for-ace.md).

---

## Flowchart

```mermaid
flowchart TD
  Start([User message received])
  Q1{"Development request?<br/>implement / fix / feature /<br/>infra / refactor / PR-bound work"}
  Q1 -->|No| Info["Answer from docs.<br/>Follow main-rules:<br/>no invention, read or ask."]
  Q1 -->|Yes| Read["Read development-request-rules-for-ace.md<br/>and minimum rule set<br/>including jira-led-development-planning"]
  Read --> A[Phase A: Understand request]
  A --> B["Phase B: Map ACE estate<br/>repos and dependencies"]
  B --> C["Phase C: Create Jira<br/>Epic/parent + Stories/tasks<br/>or linked issues"]
  C --> Stop([STOP: await Phase D approval])
  Stop --> D{"Phase D: Human accepted<br/>in Jira?<br/>comment or agreed status"}
  D -->|No| Wait["Do not implement.<br/>Message user:<br/>Planning in Jira complete;<br/>awaiting approval to proceed."]
  Wait --> D
  D -->|Yes| E["Phase E: Execute per issue<br/>branches, tests, docs, PRs"]
  E --> End([Done per issue scope])
```

---

## Plain-language sequence

1. **New feature or change that touches code or infra?** If **no**, respond from documentation; no Jira gate.
2. If **yes**: read the development entry doc and Jira-led rules **first**.
3. **Plan** in conversation only as needed to clarify; **record** the real plan in **Jira** (parent + children or linked issues).
4. **After creating issues**: **stop**. Tell the user the Jira keys and that you are **waiting for approval**.
5. **Only after** explicit go-ahead (Jira + team habit): branches, implementation, validation, PRs **per issue**.

---

## What “development request” means

Same scope as [development-request-rules-for-ace.md](../development-request-rules-for-ace.md): anything that changes behavior, APIs, schemas, migrations, configs affecting runtime, IaC/K8s for ACE, refactors with PR intent, etc. When in doubt, treat it as a development request and use Jira-led workflow unless the user explicitly opts out for that task.
