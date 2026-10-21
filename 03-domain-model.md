# 3. Domain Model and Lifecycle State Machines

## 3.1 Aggregate Overview

Core aggregates:

- AdvisorAggregate
- HouseholdAggregate
- ClientAggregate
- AccountAggregate
- PortfolioAggregate
- TradeAggregate
- TransferRequestAggregate
- FeeScheduleAggregate
- JournalAggregate
- ReconciliationBreakAggregate

## 3.2 Core Entities

### Advisor
- Id
- Name
- Firm affiliation
- Status
- Permissions

### Client
- Id
- HouseholdId
- KYC status
- Contact details
- Status

### Household
- Id
- Household name
- Members
- Servicing advisor

### Account
- Id
- ClientId
- HouseholdId
- Account type
- Custodian account reference
- Status
- Opened date
- Closed date

### Security
- Id
- Ticker or symbol
- Asset class
- Currency
- Price source
- Active flag

### Position
- AccountId
- SecurityId
- Quantity
- Market value
- Cost basis
- Last updated timestamp

### Transaction
- Id
- AccountId
- Transaction type
- Direction
- Amount
- Status
- External source
- Business date

### Journal
- Id
- Business reference
- Journal type
- Currency
- Effective date
- Status
- Posted timestamp

### JournalLine
- JournalId
- AccountId
- DebitAmount
- CreditAmount
- Currency
- Description

### FeeSchedule
- Id
- Version
- EffectiveFrom
- EffectiveTo
- Account eligibility rules
- Valuation formula
- Rounding policy

### ReconciliationBreak
- Id
- AccountId
- Source type
- Internal record id
- External record id
- Break type
- Severity
- Status

## 3.3 State Machines

### Account Lifecycle
- Draft
- Submitted
- Approved
- Active
- Suspended
- Closed
- Archived

Transitions must record actor, date, and reason.

### Transfer Request Lifecycle
- Draft
- Submitted
- Approved
- PendingSettlement
- Settled
- Rejected
- Reversed

### Fee Calculation Lifecycle
- Pending
- Eligible
- Calculated
- Approved
- Posted
- Reversed

### Reconciliation Break Lifecycle
- Identified
- Investigating
- Resolved
- Accepted
- Reopened

## 3.4 Business Rules

- An account cannot be closed until user-visible obligations are completed.
- A transfer request cannot settle without a valid approval chain.
- Fee calculations must reference a schedule version and execution record.
- A journal balance cannot be posted unless debits equal credits.
- Reconciliation break resolution must preserve evidence.

## 3.5 Persistence and ORM Boundary

- The modern backend uses EF Core with the Npgsql provider for PostgreSQL persistence.
- Domain rules remain in domain/application services; EF Core entities and database rows are not public API contracts.
- EF Core migrations are owned and versioned by the backend. Each migration is reviewed for forward deployment and recovery implications.
- Synthetic seed records are deterministic, versioned, repeatable, and restricted to local/test environments.
- Ledger persistence uses explicit transactional posting and database constraints; ORM usage does not replace domain validation or financial invariants.

## 3.6 Acceptance Criteria

The domain model is acceptable when:

- every primary workflow has an explicit lifecycle;
- ownership boundaries between account records, ledger records, and operational records are clear;
- financial state transitions are traceable;
- no state machine allows silent updates.
