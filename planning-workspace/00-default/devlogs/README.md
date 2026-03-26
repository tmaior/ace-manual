# Development Logs (Devlogs)

This folder contains development logs that capture the journey of building features, solving problems, and making decisions throughout the development process.

## What is a Devlog?

A **devlog** (development log) is a chronological record of development activities, decisions, and insights. Unlike formal documentation that describes the final state, devlogs capture the *process* - the reasoning, alternatives considered, problems encountered, and solutions discovered.

Devlogs serve as:
- A historical record of development decisions
- A knowledge base for future reference
- A way to share insights with other agents and developers
- Documentation of the "why" behind technical choices

## When to Create Devlog Entries

Create a devlog entry in the following situations:

### Feature Completed
When a feature is finished and working correctly:
- Document what was built
- Explain the approach taken
- Note any interesting implementation details

### Error Encountered
When you encounter an error, bug, or unexpected behavior:
- Document the exact error message
- Describe what was happening when it occurred
- Note the circumstances that triggered it

### Solution Found
When you fix an error or resolve a problem:
- Document the root cause
- Explain the solution applied
- Note why this solution was chosen over alternatives

### Decision Made
When you make a significant technical decision:
- Document the options considered
- Explain the trade-offs
- State the final choice and reasoning

### Complex Logic Implemented
When implementing non-trivial logic or algorithms:
- Document the approach
- Explain the reasoning
- Note any alternatives that were considered

### Suggestion or Idea
When you have an idea or suggestion for future work:
- Document the suggestion
- Explain the potential benefit
- Note any caveats or concerns

## What to Document in Devlogs

### Document These Items:
- **Complex logic**: Any non-trivial implementation logic
- **Suggestions**: Ideas for future improvements or optimizations
- **Solutions**: How problems were actually solved
- **Choices and reasons**: Decisions made and the reasoning behind them
- **Errors and fixes**: Problems encountered and how they were resolved
- **Alternative approaches**: What was considered but not chosen

### Why Documentation Matters:
- Enables other agents to understand the reasoning behind decisions
- Provides context for future maintenance and development
- Captures institutional knowledge
- Helps avoid re-solving problems that have already been solved

## Devlog Entry Structure

Each devlog entry should follow a consistent structure:

```markdown
# Devlog Entry: [Title]

**Date/Time:** YYYY-MM-DD HH:MM
**Type:** [feature|fix|error|solution|decision|other]

## Description
[What happened or what was being worked on]

## Details
[Detailed explanation of the situation]

## Solution/Result
[How the issue was resolved or what was accomplished]

## Why This Solution
[Explanation of why this approach was chosen over alternatives]

## Related
[Links to related files, issues, or previous entries]
```

## Best Practices

### Do:
- Write entries in the past tense for completed items
- Write entries in present tense for current situations
- Be specific with dates and version numbers
- Include code snippets when relevant
- Reference related files and documents
- Explain your reasoning thoroughly

### Don't:
- Leave entries incomplete (finish what you start)
- Document trivial, obvious things
- Speculate without evidence
- Make entries too long (split if needed)
- Use vague descriptions

## Creating Entries

Use the `devlog-entry.md` template as a starting point for each new entry. Copy the template, fill in the details, and save with a descriptive filename that includes the date.

Example filename: `2026-03-26-feature-authentication-implementation.md`

## Related Documentation

- **[../README.md](../README.md)** - Planning workspace overview
- **[../RULES.md](../RULES.md)** - Planning workspace rules
- **[./devlog-entry.md](./devlog-entry.md)** - Template for new entries
