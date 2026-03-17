# Dependencies and integrations

## External dependencies (runtime)

| Dependency | Purpose |
|------------|---------|
| **ace-stack-backend** | Single backend for the frontend. All API calls go to this service. Two base URLs are used: **VITE_API_URL** (admin and user APIs, e.g. `/api/admin/users`, `/api/admin/projects`) and **VITE_BACKEND_URL** (secrets, documentations, chat, integrations, e.g. `/api/secrets`, `/api/chat/send`). Both must point to the same backend in each environment (different ports or paths as configured). |

The frontend does not call ace-db-gateway, ace-configuration, or other ACE services directly; it always goes through ace-stack-backend.

## Main libraries (package.json)

| Package | Use |
|---------|-----|
| **react**, **react-dom** | UI framework. |
| **react-router-dom** | Routing and protected routes. |
| **vite** | Build tool and dev server. |
| **axios** | HTTP client for useApi (VITE_API_URL); interceptors for JWT and 401 logout. |
| **fetch** | Used by DashboardBackend and some hooks (VITE_BACKEND_URL). |
| **jwt-decode** | Decode JWT for validation/expiry. |
| **react-hook-form**, **@hookform/resolvers**, **zod** | Forms and validation. |
| **@radix-ui/** (accordion, dialog, dropdown, tabs, etc.) | Accessible UI primitives; shadcn/ui is built on these. |
| **tailwindcss**, **class-variance-authority**, **clsx**, **tailwind-merge** | Styling and component variants. |
| **lucide-react** | Icons. |
| **socket.io-client** | Real-time (e.g. assistant/chat). |
| **react-markdown**, **remark-gfm**, **mermaid** | Markdown and diagrams in UI. |
| **recharts** | Charts. |
| **next-themes** | Theme (e.g. dark/light). |
| **react-google-recaptcha** | reCAPTCHA on login/sign-up when configured. |
| **sonner** | Toasts. |

Dev: **@vitejs/plugin-react-swc**, **typescript**, **eslint**, **@playwright/test**, **lovable-tagger** (development only).

## Who uses ace-dashboard-frontend

| Consumer | How |
|----------|-----|
| **End users** | Browser; login and use dashboard, assistant, documentation, commands, secrets, settings. |
| **Admins** | Admin panel and management pages (users, clients, projects, channels, permissions, user types, configurations, external linkages, Slack channels, monitored resources when enabled). |
| **Super users** | ACE AWS (Knowledge Base, KB deletion blacklist) and ACE Jira (project links) sections. |

Operationally: the app is built and served as static assets; in EKS it is typically served by Nginx (or similar) from a Docker image built in ace-infra. Users and admins access it via the configured frontend URL (e.g. dev-dashboard.ace.ezops.cloud).

## Integration pattern (architecture)

- **Frontend → Backend only**: Every API request from the dashboard goes to ace-stack-backend. The backend then talks to ace-db-gateway, ace-configuration, or other services as needed.
- **Auth**: Login and token issuance are handled by the backend; the frontend stores the JWT and sends it on each request. No direct frontend–DB or frontend–gateway integration.
- **URLs**: All backend base URLs must be set via env (VITE_API_URL, VITE_BACKEND_URL, etc.); no hardcoded backend hosts. See [environment-and-configuration.md](./environment-and-configuration.md).

## Requirements

- **Node.js**: Version per repo and Dockerfile (e.g. Node 18+); see ace-dashboard-frontend and ace-infra.
- **Browser**: Modern browser with ES module and fetch support.
- **Network**: Frontend must be able to reach ace-stack-backend at the configured API and backend URLs (CORS configured on backend for the frontend origin).
