# GitFlow Rules

This document defines the GitFlow branching strategy for ACE projects.

---

## Branch Types

### Main Branches
- `main` / `master`: Production-ready code
- `development`: Integration branch for features

### Supporting Branches
- `feature/*`: New feature development
- `bugfix/*`: Bug fixes
- `hotfix/*`: Urgent production fixes
- `release/*`: Release preparation

---

## Branch Lifecycle

### Feature Branches
```
development → feature/ACE-123-feature → development
```

1. Create from `development`
2. Implement feature
3. Open PR to `development`
4. Merge after approval
5. Delete branch

### Bugfix Branches
```
development → bugfix/ACE-123-fix → development
```

1. Create from `development`
2. Fix the bug
3. Open PR to `development`
4. Merge after approval
5. Delete branch

### Hotfix Branches
```
main → hotfix/ACE-123-urgent → main + development
```

1. Create from `main`
2. Implement fix
3. Open PR to `main`
4. Merge to `main` immediately
5. Cherry-pick or merge to `development`
6. Delete branch

---

## Protected Branches

The following branches are protected:
- `main` / `master`
- `development`

Protection settings:
- Require PR reviews
- Require status checks to pass
- No force pushes
- No branch deletion

---

## Merge Strategies

### Feature Merges
- Use "Squash and merge" for feature branches
- Or "Merge commit" if clean history is important

### Hotfix Merges
- Use "Fast-forward" when possible
- Or "Merge commit" to preserve history

---

## Related Documents

- [development-rules.md](./development-rules.md)
- [pr-rules.md](./pr-rules.md)
