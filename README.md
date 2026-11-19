# CitiCore Configuration Repository

Externalized configuration for all CitiCore microservices, served by the
Spring Cloud Config Server from a Git backend.

This repository is the **single source of truth** for environment-specific
configuration. It contains no secrets — all credentials are referenced through
environment variables (`${VAR}`) that are resolved by the running container
from the deployment environment (`.env.local` locally, AWS Secrets Manager in
staging/production).

## Repository

- **Remote**: `https://github.com/Rabbaniinamdar/citicore-config` (to be created)
- **Default branch**: `main` (protected)
- **Config Server label**: `main`

## Structure

```
citicore-config/
├── README.md
├── .gitignore
├── .env.example
│
├── application.yml                  # Shared config — ALL services
├── application-local.yml            # Shared LOCAL overrides
├── application-test.yml             # Shared TEST overrides
├── application-staging.yml          # Shared STAGING overrides
├── application-prod.yml             # Shared PROD overrides
│
├── eureka-server.yml                # Eureka Server (platform)
├── eureka-server-local.yml
├── eureka-server-test.yml
├── eureka-server-staging.yml
├── eureka-server-prod.yml
│
├── api-gateway.yml                  # API Gateway
├── api-gateway-local.yml
├── api-gateway-test.yml
├── api-gateway-staging.yml
├── api-gateway-prod.yml
│
├── auth-service.yml
├── auth-service-local.yml
├── auth-service-test.yml
├── auth-service-staging.yml
├── auth-service-prod.yml
│
├── user-service.yml
├── user-service-local.yml
├── user-service-test.yml
├── user-service-staging.yml
├── user-service-prod.yml
│
├── account-service.yml
├── account-service-local.yml
├── account-service-test.yml
├── account-service-staging.yml
├── account-service-prod.yml
│
├── transaction-service.yml
├── transaction-service-local.yml
├── transaction-service-test.yml
├── transaction-service-staging.yml
├── transaction-service-prod.yml
│
└── notification-service.yml
    ├── notification-service-local.yml
    ├── notification-service-test.yml
    ├── notification-service-staging.yml
    └── notification-service-prod.yml
```

## Configuration Resolution Hierarchy

Spring Cloud Config resolves property sources in the following order — later
sources override earlier ones:

1. `application.yml` — shared defaults (lowest priority)
2. `application-{profile}.yml` — shared environment overrides
3. `{service}.yml` — service-specific overrides
4. `{service}-{profile}.yml` — service + environment overrides (highest priority)

Example for `auth-service` with the `prod` profile:

```
application.yml  →  application-prod.yml  →  auth-service.yml  →  auth-service-prod.yml
```

## Profiles

| Profile   | Purpose | Activated by |
|-----------|---------|--------------|
| `local`   | Local Docker Compose / IDE | `SPRING_PROFILES_ACTIVE=local` |
| `test`    | CI integration tests | `SPRING_PROFILES_ACTIVE=test` |
| `staging` | AWS staging | `SPRING_PROFILES_ACTIVE=staging` |
| `prod`    | AWS production | `SPRING_PROFILES_ACTIVE=prod` |

## Design Principles

1. YAML throughout.
2. Common, non-sensitive configuration lives in shared files.
3. Service-specific configuration lives in `{service}.yml`.
4. Environment-specific overrides live in `{service}-{profile}.yml` and
   `application-{profile}.yml`.
5. Profile files contain **only** the properties that genuinely differ — never
   a copy of the base configuration.
6. Consistent property naming (`camelCase` matching Spring Boot binding).
7. No secrets — use `${ENV_VAR}` placeholders only.
8. Every change is version controlled.
9. Every file is documented.

## Secrets Management

| Environment | Mechanism |
|-------------|-----------|
| Local | `.env.local` (gitignored) + `.env.example` template |
| Test | GitHub Actions secrets / Testcontainers |
| Staging | AWS Secrets Manager + IRSA |
| Production | AWS Secrets Manager + IRSA + KMS |

**Never commit:** database passwords, JWT signing keys, AWS credentials, Git
credentials, SMTP passwords, third-party API keys, private certificates.

## Config Server Connection

The Config Server is configured in `config-service/src/main/resources/` and
reads this repository from its Git backend:

```yaml
spring:
  cloud:
    config:
      server:
        git:
          uri: ${CONFIG_GIT_URI}
          default-label: ${CONFIG_GIT_LABEL:main}
          clone-on-start: true
          force-pull: true
          timeout: 10
          username: ${CONFIG_GIT_USERNAME}
          password: ${CONFIG_GIT_TOKEN}
```

## Validation

```bash
# Shared config
curl http://config-server:8888/application/prod

# Service + profile
curl http://config-server:8888/auth-service/prod
curl http://config-server:8888/account-service/local
curl http://config-server:8888/apigateway-service/prod
```

## Adding a New Microservice

1. Create `{service}.yml` with service-specific overrides (port, datasource,
   kafka, logging).
2. Create `{service}-local.yml`, `{service}-test.yml`,
   `{service}-staging.yml`, `{service}-prod.yml` with only the
   environment-specific differences.
3. Wire the service as a Config Client (see
   `documentation/engineering/configuration-management.md`).
4. Open a pull request — configuration changes require review."# citicore-config" 
