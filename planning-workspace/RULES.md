# Rules for AI Agents - Planning Workspace

These rules govern how AI agents should work within the planning workspace to ensure consistent, organized, and effective documentation and task management.

---

## 1. Rules for Creating New Task Folders

### Folder Creation Process
- **ALWAYS** copy from `00-default/` template - never create from scratch
- **ALWAYS** use the next available two-digit number prefix
- **NEVER** skip numbers (maintain sequential order)
- **ALWAYS** use lowercase names with hyphens (e.g., `03-user-management`)

### Naming Convention
```
[NN]-[descriptive-name]/
```
- Two-digit number (01, 02, 03...) for ordering
- Hyphen separator
- Descriptive name in lowercase with hyphens for spaces
- No special characters or uppercase letters

### Examples
| Correct | Incorrect |
|---------|-----------|
| `01-authentication` | `1-authentication` |
| `02-api-design` | `02-API_design` |
| `03-database-schema` | `03 Database Schema` |

---

## 2. Rules for Documentation Flow and Order

### Sequential Documentation
AI agents MUST follow this order when documenting:

1. **First**: Read existing workspace documentation
2. **Second**: Understand current state and objectives
3. **Third**: Plan your documentation approach
4. **Fourth**: Create/update documentation following the workflow

### Documentation Workflow Order
```
Initial Planning → Architecture → Engineering → Task Management
```

### Reading Order
When entering any task folder, read in this order:
1. `README.md` - Overview and context
2. `objective.md` - What needs to be accomplished
3. `notes.md` - Current status and notes
4. Any other supporting files

---

## 3. Rules for Pre-Work Requirements

### Before Starting Any Task
- [ ] Read the workspace `README.md` completely
- [ ] Read the workspace `RULES.md` (this file)
- [ ] Review the parent workspace documentation
- [ ] Understand the overall project objectives
- [ ] Check for existing task folders to avoid duplication
- [ ] Identify dependencies with other tasks

### Before Creating Documentation
- [ ] Verify the documentation doesn't already exist
- [ ] Check if similar documentation exists that should be referenced
- [ ] Ensure you understand the context and purpose
- [ ] Identify which workflow phase applies

### Before Committing Changes
- [ ] Review all created/modified files for completeness
- [ ] Verify file formatting (Markdown linting)
- [ ] Ensure all links are correct
- [ ] Check that documentation follows templates

---

## 4. Rules for When to Document

### Document IMMEDIATELY When:
- Creating a new task or objective folder
- Completing a milestone or task
- Making a significant decision
- Discovering a dependency or constraint
- Resolving an open question
- Updating status or progress

### Document PROACTIVELY When:
- Starting a new phase of work
- Encountering blockers or issues
- Changing approach or methodology
- Adding new requirements or scope changes
- Learning something relevant for future work

### Document REGULARLY:
- Update `notes.md` with progress at each significant step
- Keep status current in all `README.md` files
- Record decisions when made
- Capture meeting notes or discussions

---

## 5. Rules for How to Document

### Formatting Standards

#### Markdown Formatting
- Use ATX-style headers (`#`, `##`, `###`)
- Use bullet lists for unordered items (`-` or `*`)
- Use numbered lists for sequences (`1.`, `2.`, `3.`)
- Use code blocks with language hints (``` ```)
- Use bold for emphasis (**text**)
- Use links for navigation `[text](url)`

#### File Structure
- Each task folder MUST contain:
  - `README.md` - Primary documentation
  - `objective.md` - Objective definition
  - `notes.md` - Working notes

#### Content Guidelines
- Write clear, concise descriptions
- Use active voice
- Include dates for significant changes
- Mark incomplete sections with `[ ]` or `[TODO]`
- Mark complete sections with `[x]` or `[DONE]`

### Documentation Template

Each task folder should follow this structure:

```markdown
# [Task Name]

## Overview
[Brief description of the task]

## Objective
[What needs to be accomplished - from objective.md]

## Status
- [ ] Not Started
- [x] In Progress
- [ ] Completed

## Dependencies
- [List any dependencies]

## Progress
[Regular updates on work progress]

## Notes
[Meeting notes, decisions, questions]
```

---

## 6. Rules About the 00-default Template Folder

### CRITICAL: DO NOT MODIFY
The `00-default/` folder is a **TEMPLATE** and **MUST NOT** be modified.

### Why 00-default Must Remain Unchanged
- It serves as the canonical reference for folder structure
- It ensures consistency across all task folders
- It provides a reliable starting point for new tasks
- Modifying it would invalidate existing references

### If Changes Are Needed
1. Document the proposed change
2. Create a new template folder (e.g., `00a-extended-default/`)
3. Use the new template for future tasks
4. Existing folders retain their original structure

### What 00-default Contains
```
00-default/
├── README.md    # Template for task documentation
├── objective.md # Template for objective definition  
├── notes.md     # Template for working notes
└── assets/      # Folder for supporting files
```

---

## 7. General Working Rules

### Git and Version Control
- Commit early and often with descriptive messages
- Use conventional commit format: `feat:`, `fix:`, `docs:`, `chore:`
- Keep commits focused and atomic
- Push changes regularly

### Collaboration
- Document decisions, not just outcomes
- Leave clear notes for next agent
- Flag unresolved items prominently
- Use `[TODO]` and `[OPEN]` markers

### Communication
- All significant information should be in documentation, not ephemeral
- Use documentation to "hand off" between sessions or agents
- Make status visible and up-to-date

---

## 8. Quick Reference Checklist

### Starting a New Task
- [ ] Copy `00-default/` to new folder with next number
- [ ] Rename folder appropriately
- [ ] Update `README.md` with task details
- [ ] Update `objective.md` with task objective
- [ ] Clear `notes.md` for fresh notes
- [ ] Add to workspace tracking

### Documenting Progress
- [ ] Update status in `README.md`
- [ ] Add notes to `notes.md`
- [ ] Link to any new artifacts
- [ ] Note any blockers or dependencies

### Completing a Task
- [ ] Verify all objectives met
- [ ] Update final status
- [ ] Document lessons learned
- [ ] Archive or link related work

---

## Summary

| Rule Category | Key Points |
|---------------|------------|
| Creating Folders | Always copy `00-default/`, use sequential numbering |
| Documentation Flow | Follow Initial → Architecture → Engineering → Tasks order |
| Pre-Work | Read all docs, understand context, check for duplicates |
| When to Document | Immediately for decisions, proactively for progress |
| How to Document | Follow templates, use Markdown, keep consistent |
| 00-default | NEVER modify - it's the canonical template |

---

## 9. Rules for Development Logs (Devlogs)

### Devlogs Purpose
Devlogs (development logs) capture the journey of development - the reasoning, decisions, problems, and solutions. They document the "why" behind technical choices, complementing formal documentation that describes the final state.

### Required Devlog Rules

#### Rule: Agents MUST Create Devlog Entries Throughout Development
- Devlogs are NOT optional - they are a required part of the documentation workflow
- Every significant development activity should result in a devlog entry
- Treat devlog entries as first-class documentation artifacts

#### Rule: Document Features When Finished
When a feature is completed:
- Create a devlog entry documenting what was built
- Explain the approach and implementation details
- Note any interesting decisions made during implementation
- Include any limitations or known issues

#### Rule: Document Errors When Encountered
When an error or bug is encountered:
- Immediately create a devlog entry documenting the error
- Include the exact error message and stack trace if available
- Describe the circumstances that triggered the error
- Note any relevant system state or inputs

#### Rule: Document Solutions When Errors Are Fixed
When an error is resolved:
- Update the existing devlog entry (or create a new one)
- Document the root cause of the error
- Explain the solution applied
- Include any lessons learned

#### Rule: Document Complex Logic, Suggestions, and Solution Choices with Reasoning
For any significant technical decision:
- Document the problem or requirement being addressed
- List the alternatives that were considered
- Explain the trade-offs of each option
- State the final choice and the reasoning behind it

#### Rule: Devlogs Should Be Updated Constantly During Development
- Update devlogs at each significant step
- Don't wait until completion - log progress as it happens
- Add entries for partial solutions, dead ends, or pivots
- Capture both successes and failures

### Devlog Entry Types

| Type | When to Use | Content Focus |
|------|-------------|----------------|
| `feature` | Feature completed | What was built, how it works |
| `fix` | Bug fixed | Problem, root cause, solution |
| `error` | Error encountered | Error details, circumstances |
| `solution` | Solution found | How the problem was solved |
| `decision` | Decision made | Options, trade-offs, reasoning |
| `other` | Miscellaneous | Other relevant information |

### Devlog Entry Quality Standards

**Must Include:**
- Date and time
- Entry type
- Clear, descriptive title
- Detailed description of what occurred
- Technical details (code snippets, error messages)
- Reasoning and explanation

**Should Include:**
- Related files and documents
- Alternatives considered
- Expected vs. actual outcomes
- Action items or follow-ups

### Quick Devlog Checklist

When working on any task, verify devlog coverage:
- [ ] Started work? Log the approach being taken
- [ ] Made a decision? Log the options and reasoning
- [ ] Encountered an error? Log the details immediately
- [ ] Fixed an error? Log the solution and root cause
- [ ] Completed a feature? Log what was done
- [ ] Discovered something useful? Log it for future reference

---

*For questions or clarifications, refer to the workspace `README.md` or parent documentation.*
