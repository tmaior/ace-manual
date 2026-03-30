# Standardization Rules

These rules govern code standardization for ACE.

---

## General Standards

### Formatting
- Use consistent indentation
- Follow language style guides
- Use Prettier/ESLint when available
- Format on save

### Naming Conventions

#### Variables and Functions
```javascript
// Good
const userName = 'John';
function getUserById(id) {}

// Bad
const x = 'John';
function getUser(id) {}
```

#### Classes and Types
```typescript
// Good
class UserService {}
interface UserData {}
type UserStatus = 'active' | 'inactive';

// Bad
class userService {}
interface user_data {}
```

---

## TypeScript/JavaScript Standards

### Types
- Use explicit types for function parameters
- Use interfaces for object shapes
- Avoid `any` type
- Enable strict mode

### Imports
- Use named exports
- Group imports by type
- Use absolute paths when configured
- Remove unused imports

---

## Architecture Standards

### Modularity
- Keep functions small
- Single responsibility
- Low coupling
- High cohesion

### Error Handling
- Always handle errors
- Use typed errors
- Log appropriately
- Return meaningful messages

---

## Performance Standards

### General
- Avoid unnecessary re-renders
- Use efficient data structures
- Lazy load when possible
- Optimize queries

### Database
- Use indexes appropriately
- Avoid N+1 queries
- Use pagination
- Monitor query performance

---

## Related Documents

- [development-rules.md](./development-rules.md)
- [security-rules.md](./security-rules.md)
