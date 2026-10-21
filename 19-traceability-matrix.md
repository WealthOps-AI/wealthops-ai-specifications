# 19. Traceability Matrix

| Business Requirement | Architecture Component | Test Coverage | Acceptance Criteria |
|---|---|---|---|
| Advisor and account servicing | API A, account aggregate, relational schema | Unit + integration | Account lifecycle and approvals are stable and auditable |
| Portfolio and valuations | API B, portfolio aggregate, pricing service | Unit + integration | Positions and valuation logic remain consistent |
| Transfer request management | API C, transfer aggregate, ledger posting | Unit + idempotency | Transfer states and approvals are durable |
| Advisory fee calculations | Fee schedule aggregate, worker, ledger entries | Unit + replay test | Fee outputs are versioned and reproducible |
| Reconciliation and exception handling | Reconciliation aggregate, worker, API D | Integration + failover | Breaks preserve evidence and can be resolved |
| Messaging reliability | Kafka, transactional outbox, event catalog | Contract + duplicate-delivery tests | No duplicate financial effects |
| Airflow analytics orchestration | DAG definitions and backend trigger service | Pipeline integration tests | Partial outputs remain non-authoritative |
| Security and auditability | Authentication/authorization, audit log model, approval workflows | Security tests | Sensitive actions require approval and traceability |
| Observability and recovery | OpenTelemetry, metrics, alerting, runbooks | Failure injection tests | Operators detect and recover from incidents |
| ORM persistence and schema evolution | EF Core/Npgsql mappings and backend migrations | Migration integration tests from empty and prior schema versions | Migrations are ordered, reviewed, repeatable, and safe to deploy |
| Synthetic local data | Backend seed tooling and versioned fixtures | Idempotent bootstrap and smoke tests | A clean local environment can be loaded without production credentials |
| API gateway protection | Kong routes/plugins plus backend authorization | Gateway routing and authorization contract tests | Requests reach only intended upstreams and backend authorization remains authoritative |
| Local and production platform | Docker Compose, containers, production Helm/EKS, AWS managed services | Compose startup/smoke tests and EKS deployment checks | Local services start through Compose; production runs on Kubernetes; differences are explicit |

This matrix is intended to remain updated as implementation proceeds. It is the primary cross-document consistency check for business requirements, technical design, and test validation.
