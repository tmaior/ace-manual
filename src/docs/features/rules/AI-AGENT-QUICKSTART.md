# AI Agent Quick Start - READ THIS FIRST

**IMPORTANT**: Before any development task, you MUST read this document and follow the workflow below.

---

## 🚨 The 3 Gates Before Code

You CANNOT write any code until ALL gates are passed:

| Gate | Description | When Complete |
|------|-------------|---------------|
| **Gate 1** | Jira plan accepted | Human approves Epic/Stories in Jira |
| **Gate 2** | User confirms implementation | User reviews and approves your plan |
| **Gate 3** | PR review | After implementation, before merge |

---

## Golden Path Flowchart

```
User asks for a feature/fix
           ↓
    [GATE 1] Create Jira Issues (Epic + Stories)
           ↓
    Wait for approval in Jira
    (comment like "Approved — ACE may proceed")
           ↓
    [GATE 2] User confirms you may start
           ↓
    Implement per Story
           ↓
    Share results with user
           ↓
    [GATE 3] Open PR after user confirmation
           ↓
    Done!
```

---

## 🚫 What NOT to Do

- ❌ Don't write code immediately
- ❌ Don't create branches before Jira acceptance
- ❌ Don't open PRs before user confirms
- ❌ Don't skip reading the rules
- ❌ Don't treat chat as system of record

## ✅ What TO Do

- ✓ Read rules first (this file is a start)
- ✓ Create Jira issues before implementation
- ✓ Wait for explicit approval
- ✓ Use feature branches
- ✓ Push regularly to remote
- ✓ Update Jira with progress

---

## Required Reading (in order)

1. [main-rules.md](./main-rules.md) — Non-negotiable principles
2. [jira-led-development-planning.md](./jira-led-development-planning.md) — **Plan → Jira → Wait → Execute**
3. [ai-agent-ace-workflow.md](./ai-agent-ace-workflow.md) — Feature branches, push, validation
4. [development-rules.md](./development-rules.md) — Testing, branches from `development`
5. [gitflow-rules.md](./gitflow-rules.md) — Branch naming conventions

Then as needed: `pr-rules.md`, `security-rules.md`, `testing-rules.md`

---

## Quick Checklist

Before starting ANY development task:

- [ ] Read this file (AI-AGENT-QUICKSTART.md)
- [ ] Read [main-rules.md](./main-rules.md)
- [ ] Read [jira-led-development-planning.md](./jira-led-development-planning.md)
- [ ] Understand: Jira is the system of record, not chat
- [ ] Create Jira issues (Epic + Stories)
- [ ] Wait for approval in Jira
- [ ] Only THEN implement

---

## Violation Consequences

If you implement before Jira acceptance:
- Work may be discarded if plan changes
- User loses ability to review and redirect early
- Breaks audit trail in Jira
- Violates mandatory ACE rules

---

## Need Help?

- Full rules: See [START_HERE.md](./START_HERE.md)
- First-time setup: See [FIRST-TIME-DEVELOPER.md](./FIRST-TIME-DEVELOPER.md)
- Questions? Ask the user before guessing.
