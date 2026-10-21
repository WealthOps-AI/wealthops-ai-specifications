# 10. Custodian Integration and Reconciliation Design

## 10.1 Integration Strategy

The modern backend must integrate with a synthetic custodian adapter through a replaceable interface abstraction. This allows local mock data, future public-data integration, and a clear compatibility boundary for the legacy .NET Framework adapter.

## 10.2 Provider Interfaces

- `ICustodianFileImportService`
- `ISecurityReferenceDataProvider`
- `IPaymentStatusSimulator`
- `INotificationSender`

Each interface includes:
- authentication configuration
- timeout and retry policy
- rate limit handling
- error mapping to domain errors
- contract tests

## 10.3 Legacy Adapter Boundary

The legacy .NET Framework 4.8.1 adapter is separately hosted and interacts with the backend via a documented compatibility contract. It does not share database tables directly with the modern app.

The modern adapter/API boundary is routed through Kong for HTTP policy and observability. Events emitted by the modern backend flow through Kafka and the transactional outbox. The legacy host may publish or consume only through a versioned HTTP contract or a separately approved Kafka-compatible client; it must not bypass the boundary by writing directly to the database.

## 10.4 Reconciliation Design

### Input Sources
- Internal account balances, positions, and transaction history.
- Synthetic custodian cash and position snapshots.
- Trade and settlement metadata.

### Break Types
- missing internal record
- missing custodian record
- duplicate record
- stale record
- invalid quantity or cash mismatch
- account mapping mismatch

### Resolution Workflow
- Break is created with full evidence.
- Analyst investigates and records context.
- Resolution is approved or rejected.
- Audit trail preserved.

## 10.5 Acceptance Criteria

The integration and reconciliation design is acceptable when:

- no external integration is required for local development;
- all integrations are replaceable and mockable;
- reconciliation break evidence is durable and reviewable;
- financial integrity is maintained even under provider failures.
