# PR Rules

These rules govern pull request creation and review.

---

## PR Creation

### Before Opening PR
- [ ] All tests pass locally
- [ ] Code is formatted correctly
- [ ] Documentation is updated
- [ ] Jira issue is referenced
- [ ] Branch is up to date with target

### PR Template
```markdown
## Description
<!-- What does this PR do? -->

## Type of Change
- [ ] Bug fix
- [ ] New feature
- [ ] Breaking change
- [ ] Documentation update

## Related Issues
<!-- Link to Jira issues -->

## Testing
<!-- How was this tested? -->

## Checklist
- [ ] Tests added/updated
- [ ] Documentation updated
- [ ] Code follows style guidelines
```

---

## PR Title Format

```
<type>(<scope>): <subject>

Types: feat, fix, docs, refactor, test, chore
```

### Examples
```
feat(auth): add password reset functionality
fix(api): correct response format for user endpoint
docs(readme): update installation instructions
```

---

## Review Requirements

### Minimum Reviews
- 1 approval for small changes
- 2 approvals for complex changes
- All CI checks must pass

### Review Checklist
- [ ] Code is readable
- [ ] Tests are adequate
- [ ] No security issues
- [ ] Documentation is complete
- [ ] Follows coding standards

---

## Merge Requirements

- All reviews approved
- All CI checks passing
- Branch is up to date
- No conflicts

---

## Related Documents

- [development-rules.md](./development-rules.md)
- [testing-rules.md](./testing-rules.md)
