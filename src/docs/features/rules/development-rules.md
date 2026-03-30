# Development Rules

These rules govern the implementation phase of development.

---

## Branch Strategy

### Base Branch
- All feature development MUST branch from `development`
- Never branch directly from `main` or `master`
- Hotfixes may branch from release tags

### Branch Naming
```
feature/<jira-issue>-description
bugfix/<jira-issue>-description
hotfix/<jira-issue>-description
release/<version>
```

### Examples
```
feature/ACE-123-user-authentication
bugfix/ACE-456-fix-login-error
hotfix/ACE-789-security-patch
```

---

## Local Development

### Setup
1. Clone the repository
2. Install dependencies
3. Copy `.env.example` to `.env`
4. Configure environment variables
5. Run database migrations
6. Start development server

### Testing
- Run tests locally before pushing
- Ensure all tests pass
- Add tests for new functionality
- Update tests for changed functionality

### Code Style
- Follow existing code style
- Use linters if configured
- Format code before committing
- Run pre-commit hooks if available

---

## Commit Guidelines

### Commit Message Format
```
<type>(<scope>): <subject>

<body>

<footer>
```

### Types
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation
- `refactor`: Code refactoring
- `test`: Adding tests
- `chore`: Maintenance

### Example
```
feat(auth): add JWT token refresh

- Implement token refresh endpoint
- Update auth middleware to handle refresh
- Add tests for refresh flow

Closes ACE-123
```

---

## Code Review

- Request review from team members
- Address feedback promptly
- Don't push directly to protected branches
- Squash commits before merge if needed

---

## Related Documents

- [gitflow-rules.md](./gitflow-rules.md)
- [pr-rules.md](./pr-rules.md)
- [testing-rules.md](./testing-rules.md)
