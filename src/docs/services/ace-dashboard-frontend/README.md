# ace-dashboard-frontend

React/Vite frontend for the ACE (Automation, Control & Enablement) system.

## Purpose

- User-facing dashboard and management UI (home, assistant, documentation, commands, secrets, integrations, settings).
- Admin panel: users, clients, projects, channels, permissions, user types, configurations, external linkages, Slack channel management, monitored resources (when enabled).
- Super-user sections: ACE AWS (Knowledge Base, KB deletion blacklist) and ACE Jira (project links).
- Calls **ace-stack-backend** only; base URLs from env (`VITE_API_URL`, `VITE_BACKEND_URL`). Tech stack: React 18, TypeScript, Vite 5, React Router 6, Radix UI, Tailwind, axios/fetch, JWT auth.

## Documentation in this folder

| Document | Description |
|----------|-------------|
| [START_HERE.md](./START_HERE.md) | Entry point and short description of each file. |
| [index.md](./index.md) | Simple list of contents. |
| [overview.md](./overview.md) | What the app is, purpose, architecture, main components. |
| [dependencies-and-integrations.md](./dependencies-and-integrations.md) | Backend dependency, libraries, consumers, integration pattern. |
| [features-and-capabilities.md](./features-and-capabilities.md) | Routes, auth, API clients, feature areas, feature flags. |
| [environment-and-configuration.md](./environment-and-configuration.md) | VITE_* env vars, per-env behavior, secrets. |
| [build-deploy-and-cicd.md](./build-deploy-and-cicd.md) | Build, Docker, Kubernetes, GitHub Actions. |

## How to use

- **New to the service**: Start with [START_HERE.md](./START_HERE.md), then [overview.md](./overview.md).
- **Setting up locally**: See [environment-and-configuration.md](./environment-and-configuration.md) and [build-deploy-and-cicd.md](./build-deploy-and-cicd.md); detailed setup in the repo `ace-dashboard-frontend/docs/`.
- **Deploying**: See [build-deploy-and-cicd.md](./build-deploy-and-cicd.md) and [../../environments/](../../environments/).

## Related

- **Repository docs**: `ace-dashboard-frontend/docs/` (setup, development, API, architecture).
- [Architecture service catalog](../../architecture/service-catalog.md)
- [Environments](../../environments/) for local and deployed setup
