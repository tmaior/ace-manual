# Security Rules

These rules govern security practices for ACE development.

---

## General Principles

### 1. Never Expose Secrets
- Never commit API keys
- Never commit passwords
- Never commit tokens
- Use environment variables

### 2. Validate All Inputs
- Validate user input
- Sanitize data before use
- Use parameterized queries
- Handle edge cases

### 3. Principle of Least Privilege
- Request minimum permissions
- Use read-only when possible
- Rotate credentials regularly

---

## Authentication

- Use strong password policies
- Implement MFA when possible
- Use secure session management
- Implement proper logout

## Authorization

- Check permissions before actions
- Implement role-based access
- Validate ownership of resources
- Log authorization decisions

---

## Data Protection

### At Rest
- Encrypt sensitive data
- Use secure storage
- Implement key rotation

### In Transit
- Use HTTPS everywhere
- Validate certificates
- Implement HSTS

---

## Error Handling

- Don't expose stack traces
- Log errors securely
- Return generic messages to users
- Alert on security events

---

## Compliance

- Follow OWASP guidelines
- Meet regulatory requirements
- Document security decisions
- Conduct security reviews

---

## Related Documents

- [application-requirements.md](./application-requirements.md)
- [testing-rules.md](./testing-rules.md)
