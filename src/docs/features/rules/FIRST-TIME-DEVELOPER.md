# First-Time Developer Checklist

**Purpose**: This checklist ensures you (human or AI agent) have read all required documentation before starting development work on ACE.

**Status**: Complete ALL items before implementing any feature, fix, or code change.

---

## Core Rules (Mandatory for All)

- [ ] Read [main-rules.md](./main-rules.md)
  - English for code and docs
  - Document every change
  - Never invent — read first

- [ ] Read [development-request-rules-for-ace.md](../development-request-rules-for-ace.md)
  - This is the entry point for development
  - Contains the minimum reading set

- [ ] Read [jira-led-development-planning.md](./jira-led-development-planning.md)
  - **This is CRITICAL**
  - Plan in Jira first
  - Wait for acceptance before coding

---

## Execution Rules (Before Implementation)

- [ ] Read [ai-agent-ace-workflow.md](./ai-agent-ace-workflow.md)
  - Feature branches
  - Push to remote regularly
  - Use project secrets

- [ ] Read [development-rules.md](./development-rules.md)
  - Branch from `development`
  - Local testing

- [ ] Read [gitflow-rules.md](./gitflow-rules.md)
  - Branch naming: `feature/`, `bugfix/`, `hotfix/`
  - Protected branches

---

## PR Rules (Before Opening PRs)

- [ ] Read [pr-rules.md](./pr-rules.md)
  - PR templates
  - Base branches
  - Body structure

- [ ] Read [testing-rules.md](./testing-rules.md)
  - Verification requirements
  - What to test

---

## Key Understanding Check

**Can you answer these questions?**

1. What are the 3 gates before code?
   - Answer: Gate 1 (Jira acceptance), Gate 2 (User confirms), Gate 3 (PR review)

2. What is the system of record for development plans?
   - Answer: Jira (not chat/Slack/Teams)

3. When should you stop and wait?
   - Answer: After creating Jira issues (Phase C), before implementation (Phase D)

4. What should you do if something is unclear?
   - Answer: Read documentation first, then ask the user. Never guess.

---

## Important Notes

- Chat is NOT a system of record — Jira is
- You can opt out of Jira workflow only if the user explicitly requests it
- If you opt out, state the exception in your response
- First-time reading may take 15-30 minutes — this is expected

---

## Related Documents

- [AI-AGENT-QUICKSTART.md](./AI-AGENT-QUICKSTART.md) — Quick reference flowchart
- [START_HERE.md](./START_HERE.md) — Full list of rules
- [README.md](./README.md) — Overview of rules directory
