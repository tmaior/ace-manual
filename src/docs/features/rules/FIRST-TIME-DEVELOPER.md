# First-time developer and agent checklist

**Audience**: Humans onboarding to ACE and AI agents that must prove they have loaded the right rules before acting.

Use this as a **self-check** before you claim you are following project process.

---

## Before your first development task

- [ ] Read [main-rules.md](./main-rules.md) (English for code/docs, document everything, correct doc placement, sync index/README/START_HERE, no invention).
- [ ] Read [development-request-rules-for-ace.md](../development-request-rules-for-ace.md) (when development rules apply; minimum reading set).
- [ ] Read [jira-led-development-planning.md](./jira-led-development-planning.md) (Phases A–E; **stop after Phase C** until Phase D approval).
- [ ] Read [AI-AGENT-QUICKSTART.md](./AI-AGENT-QUICKSTART.md) (three gates; read order).
- [ ] Understand: **chat is not the system of record for an approved plan** when Jira-led workflow applies — **Jira** is.

---

## If you are an AI agent in a sandbox

- [ ] Read [ai-agent-ace-workflow.md](./ai-agent-ace-workflow.md) (feature branches, `git push`, secrets, local-env/smoke tests, URLs, PR after user confirmation).
- [ ] Read [gitflow-rules.md](./gitflow-rules.md) and [development-rules.md](./development-rules.md) before you branch or merge.

---

## Before opening a PR

- [ ] [pr-rules.md](./pr-rules.md) (template, base branch, `PR.md`, `gh` if used).
- [ ] [documentation-rules.md](./documentation-rules.md) (docs updated in same change).
- [ ] [testing-rules.md](./testing-rules.md) if the team expects documented validation steps.

---

## Quick verification questions

Answer **yes** before coding a multi-step development request (unless the user opted out of Jira for this task):

1. Have I created (or confirmed existing) Jira issues that cover the full plan?
2. Has the human **explicitly** approved proceeding in Jira (or equivalent team process)?
3. Am I implementing **only** what the approved issues describe?

If any answer is **no**, **do not** start bulk implementation; return to Phase C or D in [jira-led-development-planning.md](./jira-led-development-planning.md).
