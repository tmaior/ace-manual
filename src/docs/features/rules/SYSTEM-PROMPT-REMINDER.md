# System prompt reminder (for AI agents)

**Purpose**: Short block you can paste into **project instructions**, **agent system prompts**, or **IDE rules** so ACE-class agents are reminded of Jira-led gating without opening the full docs tree.

**Full detail**: [development-request-rules-for-ace.md](../development-request-rules-for-ace.md), [jira-led-development-planning.md](./jira-led-development-planning.md), [AI-AGENT-QUICKSTART.md](./AI-AGENT-QUICKSTART.md).

---

## Reminder text (copy from below this line)

```
ACE / project docs development workflow (mandatory):

1. Before implementing ANY feature, bugfix, or code/infra change in ACE repositories:
   - Read development-request-rules-for-ace.md (docs root) and the minimum rule set it lists.
   - Follow Jira-led planning: jira-led-development-planning.md — Plan → Jira issues → WAIT for human acceptance → Execute per issue.

2. NEVER start bulk implementation before Jira Phase D is complete:
   - Do not create feature branches for the main scope, do not open PRs for that scope, and do not treat chat as the approval record when Jira-led workflow applies.
   - After filing Jira issues, STOP and tell the user you are awaiting approval (e.g. comment on Epic: "Approved — ACE may proceed" or team-equivalent).

3. Chat is not the system of record for the approved plan when using this workflow; Jira is.

4. Exception: If the user explicitly opts out of Jira for a one-off task, state the exception in the response and follow their instruction for that task only.

5. Also follow ai-agent-ace-workflow.md for sandboxes (push, secrets, local validation, PR after user confirmation).
```

---

## Optional one-line addition

```
For ACE development: read rules/AI-AGENT-QUICKSTART.md first, then obey Jira wait gate before coding.
```
