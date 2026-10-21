# 16. Milestone Plan and Learning Roadmap

## 16.1 Phased Delivery Plan and Status

### Milestone 1: Business and Architecture Specification — baseline complete
- finalize domain glossary and business requirements
- complete approval gate
- confirm architecture and repository boundaries
- confirm Compose-local/EKS-production, Kong, Kafka/MSK, EF Core migration, and repository topology decisions

### Milestone 2: Core Domain and Persistence — foundation complete; domain schema pending
- implemented: initial advisor, household, client, account, and audit entities, PostgreSQL schema, EF Core migration, and deterministic Development seed tooling
- pending: portfolio/trading, transfer, fee, journal, ledger-invariant, and reconciliation schemas and domain rules
- pending: validate transaction boundaries, concurrency, and financial approvals

### Milestone 3: API, UI, and Integration Contracts — foundation complete; feature/API coverage pending
- implemented: advisor, household, client, account, dashboard, and audit APIs; initial workstation views, API client, and theme configuration
- pending: production web OIDC, remaining API groups, mock provider, and versioned HTTP/event contracts

### Milestone 4: Messaging and Workers — pending
- implement Kafka outbox publisher and idempotent consumers
- deploy reconciliation and fee workers
- validate retry and recovery flows

### Milestone 5: Airflow and Reconciliation Analytics — pending
- implement ingestion and reference DAGs
- connect Airflow tasks and .NET pipeline triggers/status monitoring
- validate operational summaries and deployment through Compose locally and Kubernetes in production

### Milestone 6: Production Platform, Observability, Governance, and Recovery — pending
- implement Terraform and EKS/Helm deployment with AWS identity and managed services
- complete CI/CD, telemetry, alert dashboards, runbooks, and incident response
- harden security and audit controls

## 16.2 Learning Themes

- dependency injection and SOLID design
- domain modeling and aggregate boundaries
- REST API design and validation
- EF Core and transaction isolation
- concurrency control and idempotency
- messaging and outbox-pattern reliability
- observability and distributed tracing
- Airflow orchestration and data quality
- AWS EKS/Kubernetes and gateway operations
- security, auditing, and AWS deployment

### Future-only: Temporal Evaluation
- evaluate whether selected long-running application workflows benefit from Temporal;
- do not migrate Airflow batch/data DAGs by default;
- create a compatibility and coexistence decision before any code or infrastructure work.

### Deferred: Assistant Artifact Repositories
- separately design GitHub Copilot and Claude skill/agent artifact repositories and distribution model;
- no marketplace artifacts are created as part of the current specification update.

## 16.3 Acceptance Criteria

The milestone plan is acceptable when:

- each phase has testable outcomes;
- learning objectives align with implementation milestones;
- the final delivery remains understandable to an engineer who must maintain it later.
