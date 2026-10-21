# 4. Architecture and Deployment Diagrams

## 4.1 Architecture Summary

The proposed active system is divided into six independently versioned repositories, each with a distinct ownership boundary and deployment responsibilities.

- wealthops-ai-web: React + TypeScript UI and client-side validation.
- wealthops-ai-be: .NET 10 ASP.NET Core modular monolith, EF Core persistence, migrations, seed tooling, and two worker services.
- wealthops-ai-legacy-adapter: separately hosted .NET Framework 4.8.1 custodian adapter.
- wealthops-ai-pipelines: containerized Airflow DAGs and data-transformation workflows.
- wealthops-ai-platform: Docker/Helm/Kubernetes packaging, Terraform, AWS infrastructure, secrets, and observability configuration.
- wealthops-ai-contracts: independently versioned OpenAPI and Kafka event schemas, compatibility rules, and generated-client guidance.

## 4.2 High-Level Runtime Topology

```text
Local: Docker Compose                         Production: AWS EKS + AWS managed services
---------------------                         -----------------------------------------
Browser --HTTPS--> Nginx --> .NET 10 API       Browser/UI --> Kong --> .NET 10 API
                         |                                                |
                         +--> EF Core --> PostgreSQL                      +--> EF Core --> RDS PostgreSQL
                         +--> Outbox --> Kafka                            +--> Outbox --> MSK Kafka
                         +--> Airflow REST trigger                        +--> Airflow REST trigger
                                      |                                                 |
Kafka topics <--> .NET workers        |                       MSK topics <--> .NET workers
Airflow containers --> local S3       |                       Airflow pods --> S3
LocalStack (selected AWS APIs)         |                       Secrets Manager / CloudWatch

.NET 4.8.1 adapter: Windows-compatible local host/container and separately hosted Windows-compatible
deployment; do not assume it can run in Linux containers/nodes.

All supported workloads --> OpenTelemetry Collector --> local console/collector or CloudWatch/Datadog
```

## 4.3 Deployment Strategy

### Local Development
- Docker Compose runs the application and dependency stack locally: an Nginx HTTPS edge, Kafka (single-node KRaft development mode), PostgreSQL, Airflow, UI, API, workers, and mock integrations. Nginx reverse-proxies the web UI and browser API/OpenAPI traffic to internal containers; the backend is not published directly to the host. HTTPS covers browser-facing application traffic; internal Compose service traffic remains on the private network.
- Optional LocalStack may emulate selected AWS APIs needed for local flows (initially S3 and Secrets Manager); local filesystem/environment test doubles remain the no-cost fallback. It is not a full replica of AWS or EKS.
- A documented bootstrap command applies EF Core migrations, loads versioned synthetic fixtures idempotently, and runs smoke checks.
- Compose profiles separate a fast dependency/test stack from the full local application stack.
- Local Compose validates containers, configuration, service wiring, and workflows; production Kubernetes manifests/Helm releases are validated separately in a Kubernetes test/deployment environment.

### Cloud Deployment
- AWS EKS for containerized APIs, workers, Kong, Airflow, and supporting workloads, with Terraform-managed infrastructure and Helm-managed Kubernetes releases.
- Amazon RDS for PostgreSQL, Amazon MSK for Kafka, S3 for object storage, AWS Secrets Manager for deployed secrets, and CloudWatch for baseline telemetry.
- Kong Gateway is the application API gateway; no direct browser-to-service or controller-to-broker coupling.
- Start with Kong Gateway's open-source capabilities; use of commercial plugins or subscriptions requires a separate cost/security decision.
- External AWS managed services are not run inside the production cluster. Local containers/emulators provide developer workflows, not production-equivalent service behavior.

### Legacy Adapter
- Separate hosting from the modern backend.
- .NET Framework 4.8.1 runs in a Windows-compatible container/host; only schedule it onto Windows EKS nodes after image and integration compatibility is proven. It is not part of a Linux-only Compose deployment if the host cannot run Windows containers.
- Strict contract boundary and no direct database sharing.

## 4.4 Domain Ownership

- Backend owns domain logic, ledger posting, authorization, and business state.
- Frontend owns presentation, validation UX, and status display.
- Pipelines own ingestion and analytics; they do not post financial journals.
- Platform owns runtime configuration, observability, and deployment automation.

## 4.5 Architecture Principles

- Business state changes occur in the backend transactional boundary.
- Async workflows use durable events and outbox patterns.
- Kong is the planned production API boundary; local Compose terminates browser HTTPS at Nginx and proxies to the API over its private network. Kafka is the asynchronous event backbone.
- Database schema changes flow through reviewed EF Core migrations, never ad-hoc startup DDL.
- Financial correctness outranks UI convenience.
- No direct database access from the UI.
- Airflow is an analytics and orchestration platform, not a ledger system.

## 4.6 Acceptance Criteria

Architecture is acceptable when:

- repository boundaries are explicit and non-overlapping;
- domain responsibilities are clear;
- the modern and legacy runtime constraints are documented;
- infrastructure and observability are traceable to the business requirements.
