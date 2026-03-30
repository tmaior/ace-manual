# Development Request Rules for ACE

---

## ⚠️ MANDATORY COMPLIANCE WARNING

**Before any development work, you MUST follow the workflow in this document.**

This is NOT optional. If you skip these rules:
- Your implementation may be discarded
- You will violate mandatory ACE rules
- The user loses ability to review plans early

**Remember**: Jira is the system of record. Chat alone is NOT sufficient.

---

## Overview

This document defines the rules for requesting and implementing development work on ACE.

## When This Applies

This workflow applies to:
- New features
- Bug fixes
- Refactoring
- Documentation updates
- Infrastructure changes

## Exemptions

You can opt out of this workflow ONLY if:
1. The user explicitly requests to skip the Jira workflow
2. You state the exception in your response
3. The exception is documented in the PR

## Minimum Reading

Before starting any development task, you MUST read:

1. **[AI-AGENT-QUICKSTART.md](./features/rules/AI-AGENT-QUICKSTART.md)** — Quick start guide
2. **[main-rules.md](./features/rules/main-rules.md)** — Core principles
3. **[jira-led-development-planning.md](./features/rules/jira-led-development-planning.md)** — Planning workflow

## Request Flow

1. User submits request
2. AI agent acknowledges and plans
3. Create Jira issues (Epic + Stories)
4. Wait for human acceptance
5. Implement per Story
6. Open PR after user confirmation

## System of Record

| Item | System of Record |
|------|-----------------|
| Development plans | Jira |
| Decisions | Jira comments |
| Progress | Jira status |
| Chat | NOT official |

---

## Related Documents

- [features/rules/jira-led-development-planning.md](./features/rules/jira-led-development-planning.md)
- [features/rules/development-rules.md](./features/rules/development-rules.md)
- [features/rules/pr-rules.md](./features/rules/pr-rules.md)
