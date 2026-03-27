# Architecture

System architecture, design decisions, and technical overview for ACE. This directory is the **single place** for high-level structure, service boundaries, data flow, deployment, and security so that agents and developers can work safely across repositories.

---

## Purpose

This directory holds documentation for:

- **High-level system design** and service boundaries (see [overview.md](./overview.md))
- **Service catalog**: every service, tech stack, ports, env vars, dependencies (see [service-catalog.md](./service-catalog.md))
- **Data flow** and communication between services: auth, request flows, sequences (see [data-flow.md](./data-flow.md))
- **Integrations**: how services call each other, URLs, JWT, contracts (see [integrations.md](./integrations.md))
- **Deployment**: where ACE runs (local, staging, production), K8s, AWS (see [deployment.md](./deployment.md))
- **Security model**: auth, secrets, validation (see [security.md](./security.md))
- **Architecture Decision Records (ADRs)** in [adr/](./adr/)
- **Diagrams**: component, deployment, sequence (see [diagrams.md](./diagrams.md))

---

## How to use

- **New to ACE or starting a task**: Read [START_HERE.md](./START_HERE.md), then [overview.md](./overview.md) and [service-catalog.md](./service-catalog.md).
- **Implementing a feature across services**: Use [data-flow.md](./data-flow.md) and [integrations.md](./integrations.md) to see how data and APIs connect.
- **Deploying or changing infra**: Use [deployment.md](./deployment.md) and [security.md](./security.md).
- **Understanding a past decision**: Check [adr/](./adr/).

---

## Relationship with other docs

- **Parent**: The documentation hub is [../README.md](../README.md) (on disk: `ace-manual/src/docs/features/`); architecture is one section.
- **Rules**: For Gitflow, development, PR, documentation, and security rules, see [../rules/](../rules/).
- **Environments**: For local setup and deployment targets, see [../environments/](../environments/).
- **Services**: For per-service APIs and setup, see [../services/](../services/).

