# Jira-Led Development Planning

---

## ⚠️ IMPORTANT: DO NOT START CODING UNTIL PHASE D IS COMPLETE

### 🚫 What NOT to do:
- ❌ Don't write any code
- ❌ Don't create branches (except for research)
- ❌ Don't open PRs
- ❌ Don't run implementation commands in sandbox

### ✅ What TO do:
- ✓ Create Jira Epic
- ✓ Create Stories/Tasks
- ✓ Wait for approval comment like: "Approved — ACE may proceed"
- ✓ Only then execute

### ⚠️ VIOLATION CONSEQUENCES
If you implement before Jira acceptance:
- Work may be discarded if plan changes
- User loses ability to review and redirect early
- Breaks audit trail in Jira
- This is a MANDATORY rule - violations are not acceptable

---

## Overview

This document defines the workflow for planning development work using Jira as the system of record. All development requests must go through this process before implementation begins.

## Workflow Phases

### Phase A: Understand the Request
- Read the user's request carefully
- Ask clarifying questions if needed
- Ensure you understand the goal and constraints

### Phase B: Research and Analysis
- Research relevant documentation
- Identify affected components
- Analyze dependencies and risks

### Phase C: Create Jira Issues
Create the following Jira issues:

1. **Epic**: High-level feature or change
2. **Stories**: Specific deliverable items
3. **Tasks**: Technical implementation tasks

Each issue should include:
- Clear title
- Detailed description
- Acceptance criteria
- Estimated effort
- Priority

> ⚠️ **STOP HERE** — Do NOT proceed to Phase E until Phase D (Human Acceptance) is complete.

### Phase D: Human Acceptance
- Wait for the user to review the Jira plan
- User must explicitly approve with a comment like: "Approved — ACE may proceed"
- Address any feedback or changes requested
- Do NOT proceed until approval is received

### Phase E: Implementation
Only after Phase D is complete:
1. Create feature branches from `development`
2. Implement per Story
3. Push regularly to remote
4. Update Jira with progress
5. Open PR when ready

### Phase F: Verification and Merge
- Run tests locally
- Ensure CI passes
- Request review
- Merge after approval

---

## Important Principles

1. **Jira is the System of Record**: All plans, decisions, and progress must be tracked in Jira
2. **Chat is NOT Sufficient**: Don't treat Slack, Teams, or chat as official documentation
3. **Wait for Explicit Approval**: Don't assume permission - wait for the user to confirm

---

## Exception Handling

If the user explicitly requests to skip Jira workflow:
1. State the exception in your response
2. Document the decision in the PR
3. Acknowledge the deviation from standard process
