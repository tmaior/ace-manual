# Start Here – ace-jira-integration

**ace-jira-integration** is the ACE Atlassian Connect app for Jira. It receives Jira webhooks (issue_created, issue_updated, issue_deleted, comment_created), resolves ACE project links via ace-db-gateway, and forwards payloads to ace-ops-scheduler for the omnichannel pipeline.

## Contents

- **[README.md](./README.md)** – Overview of this folder, purpose, and links to all docs.
- **[index.md](./index.md)** – Simple list of contents of this directory.
- **[overview.md](./overview.md)** – What the app is, purpose, high-level architecture, main components, and what you can do with it.
- **[webhooks-and-routes.md](./webhooks-and-routes.md)** – Atlassian Connect descriptor, webhook endpoints, authentication, and test bypass routes.
- **[dependencies-and-integrations.md](./dependencies-and-integrations.md)** – Jira/Atlassian Connect, ace-db-gateway (project-jira-links), ace-ops-scheduler (payloads), and main libraries.
- **[features-and-capabilities.md](./features-and-capabilities.md)** – Webhook handling, assign-to-ACE filter, comment mention detection, descriptor resolution, logging.
- **[environment-and-configuration.md](./environment-and-configuration.md)** – Required and optional env vars, config.json, multi-env base URLs.
- **[build-deploy-and-cicd.md](./build-deploy-and-cicd.md)** – Local build and run, Docker image (ace-infra), Kubernetes deploy, GitHub Actions (dev, stg, prod).

## Where to find more

- **Repository**: `ace-jira-integration/` (sibling to ace-manual). Code, atlassian-connect.json, and repo-specific docs (e.g. `docs/` if present).
- **Architecture**: [../../architecture/service-catalog.md](../../architecture/service-catalog.md), [../../architecture/data-flow.md](../../architecture/data-flow.md).
- **Environments**: [../../environments/](../../environments/) for local, dev, stg, and production setup.
