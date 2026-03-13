# local-env

Local development configuration for ACE: docker-compose and configs.

## Purpose

- Run ACE services locally (backend, db-gateway, Redis, PostgreSQL, frontend, etc.).
- Single place for compose files and local env references; used by developers before pushing to shared environments.

## Documentation

- **This folder**: Central reference; setup and usage are documented under **environments/**.
- **Local setup**: [../../environments/local.md](../../environments/local.md) – prerequisites, docker-compose, startup order, troubleshooting.

## Related

- [Architecture service catalog](../../architecture/service-catalog.md)
- [Environments](../../environments/)
