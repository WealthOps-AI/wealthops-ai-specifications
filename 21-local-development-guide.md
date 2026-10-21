# 21. Local Startup, Synthetic Data Loading, and Operations Guide

## 21.1 Purpose and Implementation Status

This guide describes the implemented local Compose foundation and identifies planned extensions. Local development uses Docker Compose; production targets Kubernetes on AWS EKS. Compose does not emulate Kubernetes control-plane behavior or all managed AWS service behavior.

## 21.2 Local Stack

The current Compose definition provides:

- PostgreSQL, a one-shot EF Core migrator/seeder, .NET 10 API, an Nginx HTTPS edge serving the React/Vite web app and proxying API/OpenAPI routes, in the default profile.
- Optional standalone Kafka (KRaft mode) and Airflow containers in the `full` profile. Neither is wired to application event processing or pipelines yet.

The legacy .NET Framework adapter, LocalStack, workers, and local telemetry stack are not yet included in Compose. The .NET Framework 4.8.1 adapter requires a Windows-compatible container/host and must never be a prerequisite for the core local platform.

## 21.3 Prerequisites

- Docker Desktop for macOS/Windows or a compatible Docker Engine with Docker Compose v2.
- Git and a terminal. Windows setup uses PowerShell; Linux/macOS setup uses Bash and `curl`.
- Sufficient local memory/disk for PostgreSQL, Kafka, and Airflow containers.
- No AWS account or paid service is required for local development.

## 21.4 First-Run Workflow

Use the platform-owned, operating-system-specific [Windows setup guide](https://github.com/WealthOps-AI/wealthops-ai-platform/blob/master/local/GETTING-STARTED-WINDOWS.md) or [Linux/macOS setup guide](https://github.com/WealthOps-AI/wealthops-ai-platform/blob/master/local/GETTING-STARTED-LINUX-MACOS.md). The [organization profile](https://github.com/WealthOps-AI/.github/blob/master/profile/README.md) points contributors to the same instructions.

The setup scripts and Compose configuration live in `wealthops-ai-platform/local`. The default startup starts PostgreSQL, runs migrations and the Development seed in the one-shot migrator, then starts the API and Nginx edge, which serves the web app and proxies API/OpenAPI traffic. Optional Kafka and Airflow containers are started by the full profile and are not integrated with application event processing or pipelines.

The local `.env` file is git-ignored; do not commit environment-specific values.

The smoke test checks that HTTP redirects to HTTPS, the locally trusted certificate is accepted, an unauthenticated API request is rejected, and a Development-token-authenticated request succeeds through Nginx. Compose health checks separately verify PostgreSQL and API readiness.

Migration and seed scripts are explicit, repeatable operations. They must not rely on application startup side effects.

## 21.5 Database Migrations and Seed Data

- Use EF Core with Npgsql for the modern backend.
- Store committed migrations in the backend repository and apply them through a one-shot DbMigrator container/job.
- Local startup begins with an empty PostgreSQL volume; migrations create the schema before application services are marked ready.
- Production deploys run a controlled Kubernetes migration Job before the application rollout. API/worker startup must not auto-apply production migrations.
- Current seed fixtures are deterministic, synthetic, and idempotent. Only explicitly enabled Development environments may seed them.
- Seeded records must include balanced ledger examples only when the full journal and audit metadata are valid; seed tooling must not create real-like external effects.
- Dataset version and seed execution tracking remain future work.
- Provide a clear distinction between `migrate`, `seed`, and any future `reset` command. Reset must require explicit confirmation because it deletes local data.

## 21.6 Expected Local Validation

Current automated local checks verify:

- PostgreSQL and API health checks pass before the dependent Nginx edge starts.
- Nginx proxies authenticated API requests over the private Compose network, and the backend rejects unauthenticated requests.
- Synthetic advisor, household, client, and account records are present.

Migration-version assertions, securities/positions/transactions, Kafka outbox checks, Airflow DAG checks, and financial side-effect assertions are not implemented yet.

## 21.7 Common Operations

The current Compose workflow supports:

- view status/logs with `docker compose ps` and `docker compose logs -f <service>`;
- restart one service without reseeding;
- rerun the idempotent synthetic seed through the platform-specific `load-sample-data` script;
- stop services while preserving volumes.

Migration selection, fixture export/import, a confirmation-gated reset tool, and expanded operational troubleshooting remain future work.

Do not use `docker compose down --volumes` as the normal stop command; it deletes local database and broker state.

## 21.8 Production Contrast

| Concern | Local | Production |
|---|---|---|
| Workload runtime | Docker Compose containers | Kubernetes on AWS EKS |
| Database | PostgreSQL container/volume | AWS RDS PostgreSQL |
| Event streaming | Single-node Kafka container | AWS MSK |
| Object storage/secrets | Optional LocalStack subset or local test doubles | S3 and AWS Secrets Manager with workload identity |
| Ingress | Nginx HTTPS edge container | Kong on EKS, with any AWS edge service documented separately |
| Pipeline runtime | Airflow containers | Airflow containers/tasks on EKS |
| Deployment | Compose profiles and local scripts | Terraform infrastructure, Helm/Kubernetes releases, CI/CD |

Local success does not prove Kubernetes rollout behavior, IAM integration, AWS networking, or managed-service availability. Those are validated in a non-production EKS deployment stage.

## 21.9 Current Status and Remaining Acceptance Criteria

- Implemented: a developer can build and start the core local stack without host-installed .NET, Node.js, PostgreSQL, Kafka, or Airflow (Docker is required).
- Implemented: a clean PostgreSQL volume is migrated and seeded through the one-shot Development migrator.
- Implemented: repeated synthetic seeding is idempotent; no financial posting workflows exist yet.
- Implemented: local configuration uses synthetic values and no paid AWS services.
- Pending: production migration job, Kubernetes deployment, AWS integration, and EKS rollout validation.
