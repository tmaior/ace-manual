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
   Read [README.md](./README.md) for how this documentation is organized. For the role of index, START_HERE, and README in every directory, see [REPO_RULES.md](./REPO_RULES.md).

2. **Review architecture**  
   Go to [architecture/](./architecture/) to see how services connect and communicate.

3. **Set up your environment**  
   Use [environments/](./environments/) for local setup (e.g. docker-compose) and deployment targets.

4. **Follow project rules**  
   Check [rules/](./rules/) for Gitflow, code standards, security, and conventions.

5. **Work on a service**  
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
| Run or deploy the system | [environments/](./environments/) |
| Know coding/deploy rules | [rules/](./rules/) |
| Document or use a specific service | [services/](./services/) |
| Learn how index/README/START_HERE work in each folder | [REPO_RULES.md](./REPO_RULES.md) |

---

*Last updated: March 2025*
