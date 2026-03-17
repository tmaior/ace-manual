# Start Here - ACE Documentation

Welcome to the ACE (Automation, Control & Enablement) system documentation. Use this page as your entry point.

## What is ACE?

ACE is a distributed system of microservices that provides:

- **Dashboard** (React/Vite frontend) for management and visibility
- **Backend** (NestJS) for business logic and authentication (JWT)
- **DB Gateway** for centralized database access
- **Bot services** (Slack, security, operations) and supporting APIs
- **Infrastructure** (Terraform, Kubernetes, AWS) for deployment

## First Steps

1. **Understand the structure**  
   Read [README.md](./README.md) for how this documentation is organized. For the role of index, START_HERE, and README in every directory, see [REPO_RULES.md](./REPO_RULES.md). For GitHub links to each ACE repo, see [repositories.md](./repositories.md).

2. **Review architecture**  
   Go to [architecture/](./architecture/) to see how services connect and communicate.

3. **Set up your environment**  
   Use [environments/](./environments/) for local setup (e.g. docker-compose) and deployment targets. To bring up a local environment step-by-step (same as local-env), follow [environments/local-setup/](./environments/local-setup/) in order.

4. **Understand infrastructure**  
   Use [infrastructure/](./infrastructure/) for ace-infra, AWS, Terraform, and Kubernetes.

5. **Follow project rules**  
   Check [rules/](./rules/) for Gitflow, code standards, security, and conventions.

6. **Work on a service**  
   Use [services/](./services/) for per-service APIs, setup, and debugging.

## Key Conventions

- **Code and comments**: English  
- **User-facing communication**: Portuguese  
- **Branches**: kebab-case; PRs to production go from `development`  
- **Local development**: docker-compose required  
- **Documentation**: Must be updated with every change (PRs)

## Where to Find Things

| I want to… | Go to… |
|------------|--------|
| Onboard / understand the system | This file + [README.md](./README.md) |
| See high-level design | [architecture/](./architecture/) |
| Run or deploy the system | [environments/](./environments/); local step-by-step: [environments/local-setup/](./environments/local-setup/) |
| Understand infra (ace-infra, AWS, Terraform, K8s) | [infrastructure/](./infrastructure/) |
| Know coding/deploy rules | [rules/](./rules/) |
| Document or use a specific service | [services/](./services/) |
| Get clone URLs / links for each ACE repo | [repositories.md](./repositories.md) |
| Learn how index/README/START_HERE work in each folder | [REPO_RULES.md](./REPO_RULES.md) |

## For AI agents

- **Mandatory rules**: Read [rules/main-rules.md](./rules/main-rules.md) first. All code and technical documentation must be in English; document every change; keep index, README, and START_HERE in sync when adding, removing, or moving docs; do not invent—read docs and code, then ask if unclear.
- **Local environment**: To bring up a local ACE stack (same as local-env), follow [environments/local-setup/](./environments/local-setup/) in order (prerequisites → repos → config → env vars → startup → health checks). Do not skip steps or invent paths or ports.
- **Doc structure**: Every documentation directory must have **index.md**, **START_HERE.md**, and **README.md**; when creating a new folder, add all three; when changing contents, update the three in that folder. See [REPO_RULES.md](./REPO_RULES.md).
- **Where to document**: Central docs root (here: `src/docs/`) for system-wide topics; service `docs/` for that app. See [rules/documentation-rules.md](./rules/documentation-rules.md).
- **Technical depth**: Service docs under [services/](./services/) include overviews, dependencies, environment variables, APIs/routes, build/deploy, and queues/workers where applicable. Use them to understand behavior before changing code.

