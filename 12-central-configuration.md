# 12. Central Configuration Contract

## 12.1 Scope

This configuration contract spans the frontend, backend, workers, Airflow pipelines, and infrastructure repositories. It governs runtime settings, environment-specific values, and secrets management.

## 12.2 Configuration Principles

- Fail fast on missing required settings.
- Validate configuration objects at startup.
- Use environment variables for local development.
- Use AWS Secrets Manager and workload identity for deployed secrets and credentials.
- Separate application config from secret material.
- No single all-encompassing credentials file.

## 12.3 Required Configuration Areas

- environment identity
- service name and version
- database connection strings
- Kafka bootstrap endpoints, topic/schema settings, consumer groups, and retry/dead-letter policies
- Kong routes, upstreams, and gateway policy configuration (non-secret configuration)
- worker schedule configuration
- Airflow REST URL and credentials
- external provider endpoints
- telemetry exporter settings
- feature flags
- timeout and retry values
- operational alert thresholds

## 12.4 Precedence Rules

1. Runtime environment variables
2. Local .env or secure local secret store
3. AWS Secrets Manager for cloud deployments
4. Default build-time fallback values only when explicitly safe

## 12.5 Secret Handling

- Secrets are never committed to the repository.
- Secrets are loaded through OS environment variables or secret manager integration.
- Each service must only receive the secrets it needs.

## 12.6 Acceptance Criteria

The configuration contract is acceptable when:

- all mandatory values are validated before service startup;
- secrets are not mixed with general config;
- environment-specific overrides are documented;
- local development remains self-contained and synthetic.
