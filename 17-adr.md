# 17. Architecture Decision Records

## ADR 01: Use a Modular Monolith for the Modern Business API

Decision: keep modern business logic in a modular monolith rather than a microservice mesh for the initial release.

Rationale: financial correctness and transactional consistency are more important than early decomposition. The architecture can still evolve into separate services when justified by scale or team boundaries.

## ADR 02: Keep the Ledger in the Backend

Decision: the .NET backend owns all authoritative journal posting and financial accounting state.

Rationale: the ledger is the source of truth and must not be partially controlled by UI or pipeline logic.

## ADR 03: Separate the Legacy Adapter Host

Decision: the .NET Framework 4.8.1 custodian adapter remains a separately hosted component with a formal contract boundary.

Rationale: runtime incompatibility and legacy dependencies make direct integration with the modern backend too risky.

## ADR 04: Use Kafka with an Outbox and Idempotent Consumers

Decision: message publication to Kafka occurs through a durable transactional outbox, and consumers validate idempotency before applying state changes.

Rationale: this supports at-least-once delivery without risking duplicate financial effects.

## ADR 05: Use EF Core and Explicit PostgreSQL Migrations

Decision: use EF Core with Npgsql for modern backend persistence. Commit reviewed migrations and apply them as an explicit release step. Seed only deterministic synthetic data in local/test environments.

Rationale: the ORM supports productive, type-safe persistence while migrations and explicit deployment control protect financial schema evolution.

## ADR 06: Use Kong for HTTP Gateway and Kafka for Events

Decision: use Kong Gateway for HTTP ingress and API policies, and Kafka for asynchronous domain/operational events. Do not use the gateway as a Kafka proxy.

Rationale: HTTP edge concerns and event streaming have distinct protocols and operational models.

## ADR 07: Treat Airflow as Analytics-Oriented, Not Ledger-Oriented

Decision: Airflow may orchestrate ingestion and reporting but never posts authoritative accounting journals.

Rationale: data quality workflows should support operational analytics and reconciliation without bypassing the domain ledger rules.

## ADR 08: Docker Compose Locally, Kubernetes in Production

Decision: use Docker Compose as the local development and integration runtime. Use AWS EKS/Kubernetes for production. Reuse container images, configuration contracts, and health checks, but do not claim Compose replicates Kubernetes behavior. Use LocalStack optionally for selected AWS APIs and provide local test doubles where a no-cost workflow is needed.

Rationale: Compose gives a straightforward local multi-container workflow, while production Kubernetes-specific behavior is validated in dedicated deployment checks rather than hidden behind a local cluster.

## ADR 09: Synthetic Data Only

Decision: all providers and datasets remain fictional and safe for local and cloud demonstration.

Rationale: the platform must teach domain fidelity without financial or privacy risk.
