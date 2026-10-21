# 14. Repository Trees and Independent CI/CD Design

## 14.1 Repository Boundaries

- `wealthops-ai-web`: UI, frontend tests, API client, routing, accessibility checks.
- `wealthops-ai-be`: ASP.NET Core API, domain modules, EF Core persistence/migrations, deterministic seed tool, workers, and ledger logic.
- `wealthops-ai-legacy-adapter`: separately hosted .NET Framework 4.8.1 custodian adapter, Windows-compatible image/host, contract tests, and isolated release pipeline.
- `wealthops-ai-pipelines`: containerized Airflow DAGs, staging logic, quality rules, and analytics tasks.
- `wealthops-ai-platform`: Docker Compose local stack, production Helm/Kubernetes packaging, Terraform for AWS EKS and managed services, Kong configuration, monitoring, and deployment runbooks.
- `wealthops-ai-contracts`: versioned OpenAPI specifications, Kafka event schemas, compatibility policy, and generated-client guidance. Avoid compile-time cross-repository coupling; consumers pin released contract versions.
- `wealthops-ai-specifications`: business, architecture, development, security, and testing documentation for the workspace and its repositories.

## 14.2 Proposed Repository Trees

### `wealthops-ai-web`

```text
src/
  app/                 # router, shell, auth boundary
  features/            # advisor, accounts, portfolio, billing, operations
  shared/              # API client, UI components, formatting
tests/                 # unit, component, e2e
Dockerfile
README.md
```

### `wealthops-ai-be`

```text
src/
  WealthOps.Api/       # HTTP endpoints, auth, OpenAPI
  WealthOps.Domain/    # domain rules and aggregates
  WealthOps.Application/ # use cases and ports
  WealthOps.Infrastructure/ # EF Core/Npgsql, Kafka outbox, provider adapters
  WealthOps.Workers.Reconciliation/
  WealthOps.Workers.Fees/
  WealthOps.DbMigrator/ # explicit migration and seed commands
  seed-data/            # versioned synthetic datasets, never credentials
tests/                 # unit, integration, contract
Dockerfile
README.md
```

### `wealthops-ai-legacy-adapter`

```text
src/                   # .NET Framework 4.8.1 adapter and Web API 2 host
contracts/             # pinned HTTP/event contract references
tests/                 # unit and contract tests
Dockerfile.windows     # Windows container, subject to host/runtime compatibility
README.md
```

### `wealthops-ai-pipelines`

```text
dags/
src/                   # reusable, tested pipeline modules
contracts/             # pinned references to released shared data contracts
tests/                 # DAG import, task, data-quality tests
Dockerfile
README.md
```

### `wealthops-ai-platform`

```text
local/
  compose/             # compose files/profiles and service health checks
  scripts/             # Windows PowerShell and cross-platform bootstrap helpers
  config/              # non-secret local configuration templates
  localstack/          # local AWS API configuration
aws/
  terraform/           # EKS, RDS, MSK, S3, IAM, Secrets Manager, observability
  helm/                # environment values/release definitions
gateway/
  kong/                # declarative routes/plugins/upstreams
observability/
runbooks/
README.md
```

### `wealthops-ai-contracts`

```text
openapi/
events/                # versioned Kafka event schemas
data/                  # ingestion file/data contracts
compatibility/         # compatibility and release policy
README.md
```

### Future assistant artifact repositories (deferred)

- `wealthops-ai-copilot`: later design for GitHub Copilot agents, skills, prompts, and instructions, with versioning, review, and discovery.
- `wealthops-ai-claude`: later design for Claude-compatible skills/plugins and supporting artifacts.
- These are separate product/tooling artifact repositories, not runtime dependencies. Their format, publishing marketplace/discovery model, license review, and shared-artifact strategy need a separate decision. No such files are created in this specification phase.

### Future workflow migration (documentation only)

- `wealthops-ai-temporal` may be considered as a future repository only if the Temporal evaluation approves a distinct durable-workflow runtime.
- It is not part of the initial repository set, and this specification phase adds no Temporal source code or deployment.

## 14.3 Repository Count Recommendation

Use six product repositories: web, backend, legacy adapter, pipelines, platform, and contracts. Keep the independently versioned specifications repository as the documentation source for the workspace. This isolates the .NET Framework toolchain and Windows release lifecycle, keeps domain and ledger ownership together, gives data pipelines their independent Python/Airflow lifecycle, and prevents contract publishing from requiring consumers to build each other. Keep the platform repo as the local Compose and cloud deployment source of truth rather than adding another infrastructure repository.

Defer `wealthops-ai-copilot`, `wealthops-ai-claude`, and any `wealthops-ai-temporal` repository until their separate scope, publication/security model, and ownership are approved. Avoid creating empty implementation repositories now.

## 14.4 CI/CD Principles

- Each repository has independent tests and versioning.
- Shared API and event contracts are versioned and treated as compatibility boundaries.
- Pull requests must run linting, unit tests, and contract validation.
- Deployment gating includes health checks and smoke tests.

## 14.5 Local Setup

- Docker Compose for local development, integration testing, and synthetic data bootstrap.
- Production Kubernetes deployment definitions are managed separately and run in AWS EKS.
- Synthetic fixtures for transactions, positions, and pricing data.
- Optional mock provider endpoints to support offline development.
- All modern .NET services, UI, Airflow components, and dependencies are containerized. The .NET Framework adapter requires a Windows-compatible container or host.

## 14.6 Deployment Verification

- Smoke tests on deployed environment.
- Rollback plan for failed health checks.
- Data recovery and restore instructions for operational incidents.

## 14.7 Acceptance Criteria

The repository and delivery model is acceptable when:

- each repository has a clear ownership model;
- CI/CD does not assume cross-repo builds;
- local dev is runnable without production credentials;
- deployment recovery instructions are documented.
