# Start Here – ace-dashboard-frontend

**ace-dashboard-frontend** is the user-facing React/Vite UI for the ACE system. It calls ace-stack-backend APIs (via VITE_API_URL and VITE_BACKEND_URL), handles authentication (JWT), and provides dashboard, assistant, admin panel, and super-user flows (ACE AWS, ACE Jira).

## Contents

- **[README.md](./README.md)** – Overview of this folder, purpose, and links to all docs.
- **[index.md](./index.md)** – Simple list of contents of this directory.
- **[overview.md](./overview.md)** – What the app is, purpose, high-level architecture, main components, and what you can do with it.
- **[dependencies-and-integrations.md](./dependencies-and-integrations.md)** – External dependency (ace-stack-backend), main libraries, who uses the frontend, and integration pattern.
- **[features-and-capabilities.md](./features-and-capabilities.md)** – Routing, auth, API clients, feature areas, feature flags, and UI/tooling.
- **[environment-and-configuration.md](./environment-and-configuration.md)** – Required and optional VITE_* env vars, per-env behavior, and secrets.
- **[build-deploy-and-cicd.md](./build-deploy-and-cicd.md)** – Local build and run, Docker image (ace-infra), Kubernetes deploy, and GitHub Actions (dev, demo, stg, prod).

## Where to find more

- **Repository**: `ace-dashboard-frontend/` (sibling to ace-manual). Code, setup, development, API, and architecture docs in the repo's `docs/` folder.
- **Architecture**: [../../architecture/service-catalog.md](../../architecture/service-catalog.md), [../../architecture/data-flow.md](../../architecture/data-flow.md), [../../architecture/diagrams.md](../../architecture/diagrams.md).
- **Environments**: [../../environments/](../../environments/) for local, demo, and production setup.
