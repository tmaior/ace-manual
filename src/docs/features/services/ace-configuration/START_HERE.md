# Start Here – ace-configuration

**ace-configuration** is the ACE service that owns the configuration database schema and seed data. It runs migrations and seeders (in EKS as a Job) and provides Sequelize models; other services access configuration data via **ace-db-gateway** or by importing these models, not by calling an ace-configuration HTTP API.

## Contents

- **[README.md](./README.md)** – Overview of this folder, purpose, and links to all docs.
- **[index.md](./index.md)** – Simple list of contents of this directory.
- **[overview.md](./overview.md)** – What the app is, purpose, high-level architecture, main components, and what you can do with it.
- **[dependencies-and-integrations.md](./dependencies-and-integrations.md)** – External dependencies (PostgreSQL), main libraries, and who uses ace-configuration (DB Gateway, backend, bots, ops-scheduler).
- **[features-and-capabilities.md](./features-and-capabilities.md)** – Schema ownership, migrations, seeders, runtime modes (Job vs server), health check, no config CRUD API.
- **[environment-and-configuration.md](./environment-and-configuration.md)** – Required and optional env vars, per-env behavior, secrets.
- **[build-deploy-and-cicd.md](./build-deploy-and-cicd.md)** – Local build, Docker image (ace-infra), Kubernetes Job deploy, GitHub Actions (dev, demo, prod).
- **[database-and-migrations.md](./database-and-migrations.md)** – Tables summary, migrations, seeders, connection config, links to repo schema docs.

## Where to find more

- **Repository**: `ace-configuration/` (sibling to ace-manual). Code, migrations, seeders, and repo-specific docs (e.g. `docs/architecture/`, `docs/setup/`).
- **Architecture**: [../../architecture/service-catalog.md](../../architecture/service-catalog.md), [../../architecture/data-flow.md](../../architecture/data-flow.md), [../../architecture/diagrams.md](../../architecture/diagrams.md).
- **Environments**: [../../environments/](../../environments/) for local, demo, and production setup.
