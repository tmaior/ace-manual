# AI Agent ACE Workflow

This document describes the workflow for AI agents working on ACE development.

---

## Workflow Overview

### 1. Request Reception
- Receive user request
- Acknowledge receipt
- Begin documentation review

### 2. Documentation Review
- Read relevant documentation
- Understand existing patterns
- Identify affected components

### 3. Planning Phase
- Create Jira issues (Epic + Stories)
- Wait for approval in Jira
- Get user confirmation to proceed

### 4. Implementation Phase
- Create feature branch from `development`
- Implement per Story
- Push regularly to remote
- Update Jira with progress

### 5. Verification Phase
- Run tests locally
- Ensure CI passes
- Open PR when ready
- Wait for review approval

---

## Branch Management

### Creating Feature Branches
```bash
# Always branch from development
git checkout development
git pull origin development
git checkout -b feature/your-feature-name
```

### Pushing Changes
```bash
# Push regularly to maintain backup
git push origin feature/your-feature-name
```

### Keeping Branches Updated
```bash
# Rebase on latest development
git fetch origin
git rebase origin/development
```

---

## Code Quality Requirements

- Follow existing code patterns
- Include tests for new functionality
- Update documentation as needed
- Use meaningful commit messages

---

## Error Handling

- Log errors clearly
- Don't expose sensitive data
- Provide actionable error messages
- Update Jira with issue details

---

## Security Considerations

- Never commit secrets
- Use environment variables
- Validate all inputs
- Follow security rules

---

## Related Documents

- [development-rules.md](./development-rules.md)
- [gitflow-rules.md](./gitflow-rules.md)
- [pr-rules.md](./pr-rules.md)
- [security-rules.md](./security-rules.md)
