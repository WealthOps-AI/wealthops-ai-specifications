# 9. Airflow DAG Specifications and Data Contracts

## 9.1 Repository Responsibility

The `wealthops-pipelines` repository owns Airflow DAGs and orchestration logic. Airflow, schedulers, and task executors are containerized. On Kubernetes, task execution uses isolated pods/container images. Airflow does not post ledger journals or modify authoritative financial records.

## 9.2 Initial DAGs

### 1. custodian_data_ingestion

Purpose:
- read synthetic custodian files for cash, positions, and transactions;
- validate schema and required fields;
- deduplicate and normalize records;
- route valid records to staging and invalid rows to rejection reasons.

Inputs:
- file metadata
- source file name and version
- submission timestamp

Outputs:
- normalized transaction records
- rejected record reasons
- staging status summary

Controls:
- retry rules, timeout limits, and manual backfill support;
- idempotent file processing using file identity and checksum.

### 2. reference_data_processing

Purpose:
- normalize security identifiers and market reference data;
- validate price, currency, and instrument metadata;
- capture stale or missing reference data.

Outputs:
- reference-quality scorecards
- stale data alerts
- processing metadata

### 3. reconciliation_analytics

Purpose:
- aggregate operational metrics from reconciliation breaks;
- generate dashboard-ready breakdowns;
- publish summaries to downstream notification and monitoring systems.

Outputs:
- unresolved-break counts
- daily operational summaries
- alert payloads

## 9.3 Airflow Contract with .NET Backend

- .NET backend creates a durable pipeline execution record.
- It validates the requested DAG and parameters.
- It submits the trigger to Airflow via authenticated REST API.
- It persists DAG-run ID and current status.
- It exposes async retrieval for status and logs.

## 9.4 Idempotency and Recovery

- DAG run must be idempotent by business date and source file identity.
- Failed or partial pipeline outputs are treated as staging-only and never authoritative.
- Safe replays require re-validation before publication to operational systems.
- Local development and EKS use container images; local object storage is backed by an AWS-compatible S3 emulator and deployed environments use S3.
- Airflow continues to own batch/data-pipeline DAGs in the initial architecture. A possible Temporal adoption for selected durable application workflows is future-only and is described separately; this is not an approved Airflow replacement or a current implementation task.

## 9.5 Acceptance Criteria

The Airflow design is acceptable when:

- DAG responsibilities are isolated from ledger posting;
- file identity, retries, and backfills are well-defined;
- partial outputs are not mistaken for trusted financial data;
- backend integration is durable and safe.
