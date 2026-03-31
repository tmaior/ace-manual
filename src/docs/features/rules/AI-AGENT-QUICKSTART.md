# AI agent quick start (read first for development)

**Audience**: ACE and any automated agent using this repository as the project documentation knowledge base.

**Purpose**: This is the **highest-visibility entry point** for **development work**. Read it **before** planning, creating Jira issues, or writing code. It does not replace the linked rule files; it **routes** you to them in the correct order.

---

## When this file applies

Use this path whenever the user asks for **implementation**, **fixes**, **features**, **refactors**, **infra changes**, or anything that will touch ACE codebases or result in commits and PRs. For **questions only** (no code change intended), see [main-rules.md](./main-rules.md) and answer from docs; you do not need the full development gate sequence.

---

## Three gates before you ship code (non-negotiable for development)

1. **Jira plan accepted** — The breakdown is in Jira; the human has explicitly approved proceeding (see [jira-led-development-planning.md](./jira-led-development-planning.md), Phase D). **Do not start bulk implementation before this.**
2. **User confirms implementation** — After coding, the user (or process) confirms the implementation is ready before you treat the PR as the final handoff step (see [ai-agent-ace-workflow.md](./ai-agent-ace-workflow.md) and [pr-rules.md](./pr-rules.md)).
3. **PR review** — Follow project norms: PR toward the correct base branch, description and tests as required.

Gate 1 is the one agents most often skip; **skipping it is a workflow violation** unless the user explicitly opts out for that task (documented in [development-request-rules-for-ace.md](../development-request-rules-for-ace.md)).

---

## Read order (minimum set for any development request)

Complete **all** of the following **before** substantive implementation (research-only reads of code and docs in Phases A–B are allowed; see [jira-led-development-planning.md](./jira-led-development-planning.md) for the hard stop).

| Order | Document | Why |
|-------|----------|-----|
| 1 | [main-rules.md](./main-rules.md) | Non-negotiable principles; English; document changes; do not invent. |
| 2 | [development-request-rules-for-ace.md](../development-request-rules-for-ace.md) | Official entry that binds the minimum set and Jira-led default. |
| 3 | [jira-led-development-planning.md](./jira-led-development-planning.md) | Plan → Jira → **wait** → execute. |
| 4 | [ai-agent-ace-workflow.md](./ai-agent-ace-workflow.md) | Branches, push, secrets, local validation, PR timing. |
| 5 | [development-rules.md](./development-rules.md) | Testing, branches from `development`, service `docs/`. |
| 6 | [gitflow-rules.md](./gitflow-rules.md) | Branch naming and protected-branch behavior. |

**Shortcuts for orientation** (not substitutes for reading the files):

- [golden-path-flowchart.md](./golden-path-flowchart.md) — Decision tree: when to stop and wait for Jira.
- [FIRST-TIME-DEVELOPER.md](./FIRST-TIME-DEVELOPER.md) — Checklist for first-time compliance.
- [SYSTEM-PROMPT-REMINDER.md](./SYSTEM-PROMPT-REMINDER.md) — Short text suitable for pasting into agent or project instructions.

---

## System of record

**Chat is not the system of record for an approved plan.** Once you use Jira-led workflow, **Jira** holds the plan and approval artifact. Summarize keys and links in the thread, but **do not** implement the full plan until Phase D is complete in Jira (and any thread confirmation your team requires).

---

## Summary

| Step | Action |
|------|--------|
| 1 | Classify: development request vs explain-only. |
| 2 | If development: read this file, then the minimum set table above. |
| 3 | Follow Phases A–C in [jira-led-development-planning.md](./jira-led-development-planning.md); **stop** at Phase D until approved. |
| 4 | Phase E: implement per issue; follow [ai-agent-ace-workflow.md](./ai-agent-ace-workflow.md) and PR rules. |

For the full rules directory map, see [START_HERE.md](./START_HERE.md).
