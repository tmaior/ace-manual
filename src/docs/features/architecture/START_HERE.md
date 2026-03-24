# Start Here – Architecture

This directory contains the system architecture documentation for ACE. Below is each file and subdirectory with a short description.

---

## Files

**[README.md](./README.md)**  
Overview of this directory: what it is for, what documents it contains, and how to use it. Links to parent docs and to the main entry points (overview, service catalog, data flow).

**[overview.md](./overview.md)**  
Single source of truth for what ACE is and how it is structured: high-level diagram, list of services with one-line purpose, and main boundaries (frontend, backend, db-gateway, bots, infra). Start here for the big picture.

**[service-catalog.md](./service-catalog.md)**  
Catalog of every ACE service: name, repository, tech stack, main responsibility, default port, key env vars, and dependencies. Use it to know which service to touch and how it fits in.

**[data-flow.md](./data-flow.md)**  
How data and requests move through the system: auth flow (login, JWT), request flows (user → frontend → backend → db-gateway → DB), sequence diagrams, and error/timeout handling.

**[integrations.md](./integrations.md)**  
How services integrate: base URLs (env-based), how the backend calls db-gateway and other services, JWT propagation, API contracts, and idempotency/retries where relevant.

**[deployment.md](./deployment.md)**  
Where and how ACE runs: local (docker-compose), staging, production, Kubernetes/EKS namespaces, AWS (region, accounts, main services), and links to infra and environment docs.

**[security.md](./security.md)**  
Security model for agents and developers: JWT issuance and validation, roles/permissions, where secrets live and how they are injected, input validation, and what not to do (no secrets in code or logs).

**[diagrams.md](./diagrams.md)**  
Central place for architecture diagrams: component diagram, deployment diagram, and optional sequence diagrams (e.g. Mermaid). Complements overview.md and data-flow.md.

---

## Subdirectories

**[adr/](./adr/)**  
Architecture Decision Records: why certain technical choices were made. Contains a template for new ADRs and the list of existing decisions.

---

*For project rules (Gitflow, development, PR, documentation), see [../rules/](../rules/).*
