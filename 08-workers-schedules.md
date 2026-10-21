# 8. Worker Schedules and Execution Semantics

## 8.1 Worker Architecture

Two scheduled .NET worker services are required:

- DailyReconciliationWorker
- AdvisoryFeeCalculationWorker

## 8.2 DailyReconciliationWorker

### Responsibilities
- Read internal transaction, cash, and position state.
- Compare with synthetic custodian records.
- Detect missing, stale, duplicate, or inconsistent records.
- Persist reconciliation breaks with source evidence and resolution history.

### Schedule
- Run daily after custodian import completion.
- Configurable business-date parameter and timezone.
- Retry on transient failure with bounded backoff.

### Controls
- Graceful shutdown support.
- Concurrency protections using unique execution locks per business date.
- Health metrics and dead-letter handling for queue-dependent consumers.

## 8.3 AdvisoryFeeCalculationWorker

### Responsibilities
- Evaluate eligible accounts for a defined business date.
- Use the correct fee schedule version.
- Produce deterministic result metadata and posting references.
- Support approval and subsequent posting workflows.

### Schedule
- Run after portfolio and cash state snapshots are finalized.
- Use explicit business date and schedule-version validation.
- Maintain execution state for safe recovery and restart.

### Controls
- Process each account once per execution window.
- Persist execution state and allow idempotent reruns.
- Halt gracefully if data quality is insufficient.

## 8.4 Shared Worker Requirements

- Support configurable schedules and environment overrides.
- Restrict retries to bounded attempt counts.
- Expose readiness/liveness metrics.
- Include correlation ID, business date, and workflow version in logs.
- Consume versioned Kafka topics through worker-owned consumers; persist processing/deduplication state before committing offsets.
- Build and deploy each worker as an independent container image while retaining the shared backend domain and persistence boundary.

## 8.5 Acceptance Criteria

The workers are acceptable when:

- scheduled logic is decoupled from HTTP request handling;
- recoverability and idempotency are explicit;
- duplicate processing does not create duplicate accounting effects;
- all state transitions and metrics are observable.
