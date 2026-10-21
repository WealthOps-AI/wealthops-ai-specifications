# 2. Functional and Non-Functional Requirements

## 2.1 Functional Requirements

### API A: Advisor and Account API
- Create, update, and search advisors, clients, households, and accounts.
- Enable onboarding validation, approval, and account status changes.
- Record external custodian identifiers and account metadata.
- Support audit retrieval and authorization history.

### API B: Portfolio and Transaction API
- Manage securities reference data.
- Maintain positional records and valuation snapshots.
- Create, validate, and monitor simulated trades.
- Track settlement, import, and correction states.

### API C: Money Movement and Billing API
- Create transfer requests and fee calculations.
- Support approval, settlement, rejection, and reversal states.
- Persist fee schedule versions and billing metadata.
- Post accounting references and adjustment workflows.

### API D: Operations and Reconciliation API
- Import custodian data and compare against internal records.
- Generate and resolve reconciliation breaks.
- Expose pipeline execution and operational summary data.
- Support audit and search features for exception review.

## 2.2 Cross-Cutting Requirements

- All financial calculations use decimal arithmetic and controlled rounding.
- Every user-facing action records actor, timestamp, and reason.
- Every asynchronous workflow has durable metadata and an execution state.
- Every outbound event is versioned and idempotent consumer-safe.
- Every ledger effect is traceable to a source workflow.

## 2.3 Non-Functional Requirements

### Security
- Authentication and role-based access control.
- Maker-checker approval for sensitive financial actions.
- Secrets in AWS Secrets Manager for deployed environments.
- Redaction of sensitive data in logs and traces.

### Reliability
- Durable persistence for all business state.
- Idempotent processing for transfers, billing, and pipeline operations.
- Retry with bounded backoff and no silent financial mutation.

### Performance
- APIs should respond in sub-second to low-volume local operations.
- Batch and reconciliation jobs should be asynchronous and horizontally scalable if needed.
- Long-running execution requests should return HTTP 202 for accepted asynchronous work.

### Observability
- Structured logs, distributed traces, and metrics.
- Export to CloudWatch and Datadog via OpenTelemetry-compatible settings.
- Health checks, operational alerts, and dashboard-ready summaries.

### Maintainability
- Separate business logic from infrastructure concerns.
- Versioned API and event contracts.
- Clear domain boundaries and explicit transaction boundaries.
- Use Entity Framework Core as the modern backend ORM, with reviewed, version-controlled PostgreSQL migrations.
- Keep migrations and repeatable, synthetic-only seed data in the backend repository; seed data must never run in production.

### Platform and delivery
- Publish all modern .NET, React, Airflow, and supporting service images as containers.
- Use Kong Gateway for north-south API routing and policy enforcement.
- Use Kafka for asynchronous domain and operational events, with a transactional outbox and idempotent consumers.
- Deploy to production on AWS EKS using Kubernetes/Helm; run the full local development stack with Docker Compose and container images.
- Use AWS managed services in deployed environments, including RDS PostgreSQL, MSK, S3, Secrets Manager, and CloudWatch, with local containerized equivalents or test doubles as documented.
- Run the .NET Framework 4.8.1 adapter in a Windows-compatible container/host; it is not assumed to run in a Linux Kubernetes node.

## 2.4 Acceptance Criteria

The solution is acceptable when:

- each capability is tied to a named business requirement;
- financial effects are isolated behind authorized transaction boundaries;
- all asynchronous workflows have durable status and replay metadata;
- observability covers API, ledger, and worker failures;
- security and audit controls are documented and auditable.
- a clean local checkout can be started, migrated, seeded, and smoke-tested using the documented bootstrap guide.
