# ace-dashboard-frontend – Overview

## What it is

**ace-dashboard-frontend** is the user-facing React/Vite single-page application (SPA) for the ACE (Automation, Control & Enablement) system. It provides the dashboard and management UI through which users and admins interact with the platform. All data and business logic are provided by **ace-stack-backend**; the frontend consumes backend APIs using JWT authentication and does not connect directly to databases or other ACE services.

## Purpose

- **User-facing dashboard**: Home, assistant (chat), documentation, commands, secrets, data sources, integrations, and settings.
- **Admin flows**: Admin panel with user, client, project, channel, permission, user-type, and configuration management; external linkages; Slack channel management; monitored resources (when feature-flagged); ACE AWS (Knowledge Base, KB deletion blacklist) and ACE Jira (project links) for super users.
- **Authentication**: Login (including OAuth provider callback), sign-up, verify, recover/reset password; JWT stored and sent on API requests; private and super-user routes.
- **API consumer**: Calls **ace-stack-backend** only. Base URLs come from environment variables (`VITE_API_URL`, `VITE_BACKEND_URL`); no direct calls to ace-db-gateway or other services.

## High-level architecture

- **Stack**: React 18, TypeScript, Vite 5, React Router 6. UI: Radix UI, Tailwind CSS, shadcn/ui-style components; forms (react-hook-form, zod); HTTP (axios, fetch); real-time (socket.io-client); markdown (react-markdown, mermaid).
- **Build output**: Static assets (HTML, JS, CSS). Served by a web server (e.g. Nginx in Docker in EKS); dev server runs on port 8080.
- **Auth flow**: User logs in via backend; frontend receives JWT and stores it (e.g. localStorage); `AuthContext` and `PrivateRoute`/`SuperUserRoute` protect routes; axios interceptor attaches `Authorization: Bearer <token>` and redirects to login on 401.
- **Backend usage**: Two main API entry points—**VITE_API_URL** (admin/users, clients, projects, channels, etc. via axios/useApi) and **VITE_BACKEND_URL** (secrets, documentations, chat, integrations via fetch/DashboardBackend and specific hooks). Both point to ace-stack-backend (different ports or paths depending on environment).

## Main components

| Component | Role |
|-----------|------|
| **src/App.tsx** | React Router setup; public and private routes; SuperUserRoute for ACE AWS/Jira and client-management; 404. |
| **src/context/AuthContext.tsx** | Auth state, login/logout, token; used by PrivateRoute and API clients. |
| **src/context/LoggedUserOptions.tsx** | Logged-in user options (e.g. selected project) used across pages. |
| **src/lib/DashboardBackend.ts** | Class-based API client (fetch) for backend URLs using VITE_BACKEND_URL; secrets, documentations, projects, etc. |
| **src/hooks/use-api.ts** | Axios instance with base URL from VITE_API_URL; token injection and 401 logout; user/admin/client/project APIs. |
| **src/components/PrivateRoute.tsx** | Wraps routes that require authenticated user. |
| **src/components/SuperUserRoute.tsx** | Wraps routes restricted to super users. |
| **src/pages/** | Page components for each route (Index, Login, Assistant, AdminPanel, UserManagement, etc.). |
| **vite.config.ts** | Vite config; dev server port 8080; allowedHosts for local and ACE domains; path alias `@/` → `src/`. |

## What you can do with this app

- Run the dev server (`yarn dev`), build for production (`yarn build`), or preview build (`yarn preview`).
- Implement new pages and routes following existing patterns (see repo `docs/development/`).
- Add or change environment variables (VITE_*) and document them in this manual and in the repo.
- Run lint (`yarn lint`) and Playwright E2E tests (`yarn test`); see repo `docs/development/testing.md`.

## Related documentation

- [Dependencies and integrations](./dependencies-and-integrations.md)
- [Features and capabilities](./features-and-capabilities.md)
- [Environment and configuration](./environment-and-configuration.md)
- [Build, deploy and CI/CD](./build-deploy-and-cicd.md)
- Repository docs: `ace-dashboard-frontend/docs/` (setup, development, API, architecture).
- Architecture: [service-catalog](../../architecture/service-catalog.md), [data-flow](../../architecture/data-flow.md), [diagrams](../../architecture/diagrams.md).
