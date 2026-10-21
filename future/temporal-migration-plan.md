# Future Temporal Evaluation and Migration Plan

## Status

Planning only. Temporal is not selected for the initial architecture, no Temporal repository is active, and no source code, container, or infrastructure is to be developed until a separate approval.

## Current Workflow Ownership

- Apache Airflow remains the batch/data orchestration engine for custodian ingestion, reference-data processing, and reconciliation analytics.
- .NET workers own the initial scheduled reconciliation and fee-calculation workflows.
- PostgreSQL and the .NET backend remain authoritative for operational records and the ledger.
- Kafka remains the event backbone.

## Evaluation Hypothesis

Temporal may be worth evaluating for selected long-running, stateful application workflows that need durable timers, human approval waits, compensation, or recovery across service restarts. Examples to assess later include transfer approval lifecycles, pipeline-trigger coordination, and multi-step operational remediation. This is not a decision to move Airflow DAGs wholesale.

## Future Decision Gates

Before implementation, compare:

1. Airflow's strengths for scheduled batch/data DAGs against Temporal's workflow durability model.
2. Workflow state ownership, persistence, retention, replay/versioning behavior, and operator tooling.
3. Integration boundaries with Kafka and the existing .NET workers.
4. Local Docker Compose and production EKS deployment/operations cost.
5. Migration/coexistence strategy, recovery procedures, and acceptance tests.

## Possible Coexistence Boundary

If later approved, keep Airflow for data-oriented DAGs and consider Temporal only for specifically selected durable application workflows. Any workflow moved must retain a single authoritative state owner, stable idempotency keys, audit correlation, and tested replay/compensation behavior. Avoid dual-running the same financial side effect.

## Deferred Until Approval

- Temporal server/worker deployment and Helm values.
- New Temporal repository or source tree.
- Workflow migration implementation.
- New dependencies, schemas, or runtime configuration.
