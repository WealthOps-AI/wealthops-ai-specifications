# 15. Testing Strategy and Acceptance Criteria

## 15.1 Test Layers

- Unit tests for domain logic and validation.
- Integration tests for PostgreSQL/EF Core migrations, Kafka, Kong routing, and backend services.
- Contract tests for API and message schemas.
- End-to-end tests for the advisor workstation and key operational flows.
- Concurrency and idempotency tests for financial operations.
- Recovery tests for failure scenarios and outage simulation.

## 15.2 Required Financial Tests

- Unbalanced journal rejection.
- Atomic journal posting.
- Duplicate posting prevention.
- Reversal and adjustment integrity.
- Reproducible fee calculations.
- Concurrent transfer or posting requests.
- Reconciliation break creation and resolution.
- Closed-period and invalid-state rejection.

## 15.3 Required Failure Scenarios

- external provider outage
- database outage
- Kafka outage and consumer lag/rebalance
- Kong unavailable, misrouted upstream, and rejected authorization
- duplicate message delivery
- Airflow submission timeout
- partially completed pipeline
- invalid or duplicate custodian file
- observability exporter outage
- failed deployment health check

For each scenario, the expected state, symptoms, and recovery steps must be documented.

## 15.4 Acceptance Criteria

The testing strategy is acceptable when:

- key financial invariants are validated repeatedly;
- failure scenarios are explicit and recoverable;
- no replay or duplication path causes ledger corruption;
- operational recovery steps are documented and demonstrable.
