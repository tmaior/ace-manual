# ACE Documentation

This directory is the **central documentation hub** for the ACE system. It is intended to cover everything about the system in one place, cross-repository.

## Purpose

- **Single place** for system-wide docs (architecture, environments, rules, services)
- **Consistent structure** so anyone can find setup, conventions, and service details
- **Living docs** updated with every relevant change (per project and workflow rules)

## Directory Layout

In **ace-manual**, the following tree lives under **`src/docs/features/`** (repository path: `ace-manual/src/docs/features/`).

```
docs/
├── index.md          # This index and quick links
├── START_HERE.md     # Entry point for new readers
├── README.md         # This file - overview and usage
├── development-request-rules-for-ace.md  # ACE: development requests; links minimum set (incl. Jira)
├── REPO_RULES.md     # Rules for index, START_HERE, README in every directory
├── repositories.md   # GitHub URLs and clone links for all ACE repositories
├── rules/            # Conventions, Gitflow, standards, security (agents on dev: AI-AGENT-QUICKSTART.md first)
├── architecture/    # System design, diagrams, ADRs
├── environments/     # Local, staging, production; configs; deployment
├── infrastructure/   # ace-infra repo, AWS, Terraform, Kubernetes, deployment
└── services/         # Per-service docs (APIs, setup, debugging)
```

Every documentation directory (including subdirectories) should contain **index.md**, **START_HERE.md**, and **README.md**. What each file is for and how to keep them in sync is described in [REPO_RULES.md](./REPO_RULES.md).

## How to Use

- **New to the project**: Start with [START_HERE.md](./START_HERE.md), then [architecture/](./architecture/) and [environments/](./environments/). To clone repos, use [repositories.md](./repositories.md).
- **Implementing features**: Start with [rules/AI-AGENT-QUICKSTART.md](./rules/AI-AGENT-QUICKSTART.md), then [development-request-rules-for-ace.md](./development-request-rules-for-ace.md) (mandatory reading list, including Jira-led gating), then [rules/](./rules/) for conventions and [services/](./services/) for the service you are changing.
- **Deploying or debugging**: Use [environments/](./environments/) and the relevant [services/](./services/) doc.
- **Infrastructure (ace-infra, Terraform, AWS, K8s)**: Use [infrastructure/](./infrastructure/).

## Relation to other docs

- **This docs root** (in ace-manual: `src/docs/features/`): Cross-repo, system-wide documentation. Top-level sections: rules/, architecture/, environments/, infrastructure/, services/.
- **`<service>/docs/`** (e.g. `ace-db-gateway/docs/`): Service-specific API docs, setup, and guides.

Keep both in sync: when you change behavior or APIs, update the relevant service doc and, if it affects the whole system, the appropriate file under this docs root.

## Contributing to Docs

- Use **kebab-case** for file names (e.g. `api-endpoints.md`, `debugging-guide.md`).
- All technical documentation must be in **English** (see [rules/main-rules.md](./rules/main-rules.md)); user-facing UI copy may use another language when required by the product.
- Update docs in the same PR that changes behavior or APIs.
- Prefer short, clear sections and links over long single files.

