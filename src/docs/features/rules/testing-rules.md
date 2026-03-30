# Testing Rules

These rules govern testing practices for ACE development.

---

## Testing Requirements

### Minimum Coverage
- Unit tests for all new functions
- Integration tests for API endpoints
- E2E tests for critical flows

### Test Types

#### Unit Tests
- Test individual functions
- Mock external dependencies
- Fast execution
- High coverage

#### Integration Tests
- Test component interactions
- Use real database
- Test API endpoints
- Validate responses

#### E2E Tests
- Test complete flows
- Use real browser when needed
- Validate user experience
- Cover happy path and errors

---

## Test Structure

### Naming Convention
```
<function-name>.test.ts
<function-name>.spec.ts
```

### Test File Location
```
src/
├── components/
│   └── component.test.ts
├── services/
│   └── service.test.ts
└── __tests__/
    └── integration.test.ts
```

---

## Test Writing Guidelines

### AAA Pattern
```typescript
test('should do something', () => {
  // Arrange
  const input = 'test';
  
  // Act
  const result = myFunction(input);
  
  // Assert
  expect(result).toBe('expected');
});
```

### Test Naming
- Be descriptive
- State expected behavior
- Include input values when relevant

### Test Data
- Use realistic test data
- Avoid magic numbers
- Use factories when possible

---

## Running Tests

### Local Testing
```bash
# Run all tests
npm test

# Run with coverage
npm run test:cov

# Run specific file
npm test -- file.test.ts
```

### CI Testing
- All tests must pass
- Coverage must not decrease
- No skipped tests in CI

---

## Test Maintenance

- Update tests when changing code
- Remove obsolete tests
- Keep tests fast
- Review test coverage regularly

---

## Related Documents

- [development-rules.md](./development-rules.md)
- [pr-rules.md](./pr-rules.md)
