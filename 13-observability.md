# 13. Observability, Alerts, and Runbooks

## 13.1 Observability Goals

The platform must provide structured logs, traces, and metrics across APIs, workers, messaging, database access, ledger posting, and Airflow execution. The goal is operational clarity without exposing sensitive data.

## 13.2 Required Signals

- Structured log entries with service name, version, environment, correlation ID, trace ID, and span ID.
- Metrics for API latency, errors, Kafka consumer lag, worker success/failure, and dead-letter counts.
- Distributed traces across frontend, backend, worker, and external integration boundaries.
- Health checks for application, database, queue, and external provider dependencies.

## 13.3 Export Paths

- CloudWatch and optional Datadog via OpenTelemetry-compatible exporters or collectors; EKS workloads export through a collector.
- Legacy .NET Framework adapter gets a separate instrumentation strategy due to runtime differences.

## 13.4 Operational Alerts

- API failures and 5xx rates
- Database outage or slow queries
- Kafka consumer lag, broker health, publish failures, and dead-letter topic growth
- Kong request rates, upstream latency, authorization failures, and gateway errors
- Worker failure and retry exhaustion
- Pipeline failure and quality degradation
- Ledger posting errors
- Unresolved reconciliation break spikes

## 13.5 Sensitive Data Handling

- no tokens, credentials, or confidential payloads in logs
- redact or hash PII and sensitive customer data
- limit debug data in production traces

## 13.6 Runbook Requirements

- restore service health steps
- rollback or re-run guidance
- queue replay and dead-letter recovery
- ledger incident response
- Airflow failure and partial-output handling

## 13.7 Acceptance Criteria

The observability design is acceptable when:

- operators can identify the failing service and workflow quickly;
- traces and metrics cover key financial workflows;
- alerts are mapped to business-impacting operational failures;
- legacy instrumentation is explicitly documented.
