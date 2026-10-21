# 7. Event Catalog and Kafka Topology

## 7.1 Event Principles

- Events are versioned and schema-backed.
- Consumers must be idempotent.
- Duplicate delivery must not create duplicate financial effects.
- Consumers should use durable state and replay-safe behavior.

## 7.2 Kafka Cluster and Topic Topology

- Kafka is the event backbone; services publish through application-level messaging abstractions and the transactional outbox rather than coupling HTTP controllers to a broker client.
- Topic names include domain, event name, and major schema version (for example, `wealthops.account.created.v1` and `wealthops.reconciliation.break-created.v1`).
- Partition keys use stable aggregate identifiers (account, transfer, journal, or execution ID) to preserve per-aggregate ordering where required.
- Production runs on AWS MSK; local development uses a containerized, single-node KRaft Kafka profile and does not imply production availability guarantees.
- Dead-letter topics retain poison events with original key, headers, failure classification, and correlation metadata. Retry topics or bounded application retry are specified per consumer.

## 7.3 Example Event Names

- `AccountCreated`
- `AccountStatusChanged`
- `TradeExecuted`
- `TransferRequested`
- `TransferApproved`
- `FeeCalculated`
- `JournalPosted`
- `ReconciliationBreakCreated`
- `PipelineRunFailed`

These are logical event names. Physical Kafka topic names follow the versioned naming rule in 7.2; Kafka topic and schema contracts define routing and compatibility.

## 7.4 Event Catalog

### Account Events
- `AccountCreated`
- `AccountStatusChanged`

### Portfolio Events
- `TradeSubmitted`
- `TradeExecuted`
- `PositionUpdated`

### Billing/Transfer Events
- `TransferRequested`
- `TransferApproved`
- `TransferSettled`
- `FeeCalculated`
- `BillingPosted`

### Reconciliation/Operations
- `ReconciliationBreakCreated`
- `BreakResolved`
- `PipelineRunStarted`
- `PipelineRunCompleted`
- `PipelineRunFailed`

## 7.5 Kafka Producer and Consumer Behavior

- Producers use durable topic configuration, schema validation, delivery acknowledgements, and idempotent-producer support where available.
- The transactional outbox is committed in the same PostgreSQL transaction as business state; a publisher relays it to Kafka and records delivery state.
- Consumers commit offsets only after durable processing. Consumer groups and replay behavior are documented per consumer.
- Retry attempts are bounded; poison events are moved to a versioned dead-letter topic with operator-visible diagnostics.
- Kong handles HTTP routing and API policies; it is not a Kafka proxy or substitute for broker authorization.

## 7.6 Duplicate Handling

- Event deduplication key stored in durable consumer state.
- Business effects must be protected by idempotency keys.
- Consumer side should re-check source business references before making posting or state changes.

## 7.7 Delivery Guarantees

The operational model is at-least-once from the application's perspective. Kafka producer idempotence and transactions do not make PostgreSQL-plus-Kafka business effects exactly once. Consumers use durable deduplication and domain idempotency constraints to make replay safe.

## 7.8 Acceptance Criteria

The event model is acceptable when:

- every financial effect is traceable to a domain event;
- duplicate message processing cannot duplicate ledger effects;
- dead-letter and replay procedures are defined;
- event contracts are versioned and documented.
