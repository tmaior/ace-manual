# Authentication

ace-db-gateway protects almost all API routes with authentication. Only the following are public:

- `POST /login`
- `POST /login/external`
- `GET /check/*` (user-project, debug-project, user-id, email-registered, provider-registered)
- `POST /signup/provider`

All routes under `/api` (and `GET /auth/refresh-token`) require a valid credential.

## JWT authentication

### Obtaining a token

- **Email/password**: `POST /login` with body `{ "email", "password" }`. On success, response includes `token`, `userInfo`, `tokenIAT`, `tokenEXP`.
- **External provider**: `POST /login/external` with body `{ "providerType", "providerId" }` (e.g. Slack). User must exist in `UsersProviders` and have `allow: true`. Response includes `token`, `user`, `tokenIAT`, `tokenEXP`.

Token is signed with `JWT_SECRET`; expiration from `SESSION_EXPIRATION_SECONDS` (default `12h`). Payload includes `email`, `user` (id), `hierarchy`, `clientId`, `userTypeId`, `name`.

### Using the token

Send the token in the request header:

```http
Authorization: Bearer <token>
```

The middleware `Auth.validator` verifies the JWT and attaches `req.user` (decoded payload). Invalid or missing token returns `401` with `{ "error": "Invalid token" }` or similar.

### Refresh token

- **Endpoint**: `GET /auth/refresh-token`
- **Auth**: Requires valid JWT (Bearer).
- **Response**: New token and user info (`access_token`, `userInfo`, `tokenIAT`, `tokenEXP`).

## Internal service tokens

Services that call the DB gateway without a user context can use fixed tokens. The validator accepts any of:

- `BACKEND_DB_GATEWAY_TOKEN`
- `COMMANDS_API_DB_GATEWAY_TOKEN`
- `SCHEDULLER_DB_GATEWAY_TOKEN`

(With fallback defaults in code if the env var is not set; production should set them explicitly.)

Send the token as Bearer:

```http
Authorization: Bearer <value of e.g. BACKEND_DB_GATEWAY_TOKEN>
```

If the header value matches one of these tokens, the request is allowed and no JWT payload is required. Used by ace-stack-backend, ace-commands-api, and ace-ops-scheduler.

## Session storage

After successful login or external login, the gateway stores session data in Redis (see `controllers/Session.js`). Redis connection uses `REDIS_HOST`, `REDIS_PORT`; key expiration uses `SESSION_EXPIRATION_SECONDS` and `SESSION_SECRET` for signing.

## Summary

| Method | Endpoint / scope | Credential |
|--------|------------------|------------|
| POST   | `/login`         | None (body: email, password) |
| POST   | `/login/external`| None (body: providerType, providerId) |
| GET    | `/auth/refresh-token` | JWT Bearer |
| GET    | `/check/*`       | None |
| POST   | `/signup/provider` | None |
| All    | `/api/*`         | JWT Bearer **or** internal service token (Bearer) |
