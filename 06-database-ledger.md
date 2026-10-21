# 6. Database Schema, Ledger Invariants, and Illustrative Journals

## 6.1 Database Platform

- PostgreSQL for operational data and accounting records.
- Entity Framework Core is the ORM for the .NET 10 backend, using the Npgsql provider.
- EF Core migrations are committed, ordered, reviewed, and applied as an explicit release/deployment step. Production services do not automatically mutate schema during startup.
- Each bounded module owns its tables/schema and migration decisions within the modular monolith; the initial release uses one PostgreSQL database and one coordinated backend migration history unless module ownership requires a separately documented history.
- A local/test bootstrap command applies migrations before loading deterministic synthetic seed data. Seed execution is idempotent and disabled for deployed production environments.
- Decimal/Numeric for all monetary amounts.
- Explicit currency code and rounding policy in every financial table.

## 6.2 Database Domains

- Identity and master data tables
- Account and household tables
- Position and transaction tables
- Ledger and journal tables
- Fee schedule and billing tables
- Reconciliation and pipeline metadata tables

## 6.3 Ledger Model

### Chart of Accounts
- Assets
- Liabilities
- Equity
- Revenue
- Expenses
- Gains and losses

### Journal Header
- Id
- BusinessReference
- JournalType
- PostingStatus
- EffectiveDate
- Currency
- CreatedBy
- CreatedAt
- ApprovedBy
- ApprovedAt

### Journal Line
- Id
- JournalId
- AccountCode
- DebitAmount
- CreditAmount
- Description
- ReferenceId
- MetadataJson

### Invariants
- TotalDebits = TotalCredits for every posted journal.
- Journal lines cannot be updated after posting; corrections require reverse or adjusting journals.
- A journal is persisted atomically as a single transaction.
- No partially posted journal is ever committed.
- Journal source references and approval state must be preserved.

## 6.4 Accounting Rules

- Separate cash and security position records from the ledger.
- Ledger postings must agree with transfer, fee, and trade events.
- Business dates and posting timestamps are tracked separately.
- Period closing prohibits posting to closed periods without explicit override.
- Reversals and adjustments require an approved journal entry with traceability to original postings.

## 6.5 Illustrative Journal Examples

### 1. Advisory Fee Posting

- Debit: Advisory Fee Expense
- Credit: Accrued Fee Liability

### 2. Trade Execution Cash Impact

- Debit: Brokerage Cash Asset
- Credit: Client Cash Liability or payable account

### 3. Reversal of a Fee Entry

- Debit: Accrued Fee Liability
- Credit: Advisory Fee Expense

All examples must include business reference ids and authorization metadata.

## 6.6 Data Quality Requirements

- Unique constraint on business references.
- Idempotency keys required for retry-safe financial posting.
- Business references must be immutable after creation.

## 6.7 Acceptance Criteria

The ledger design is acceptable when:

- journal balancing is enforced at the database and domain layers;
- no silent modification of posted entries is possible;
- financial workflows produce auditable postings with external references;
- reconciliation and fee posts are logically separated from operational records.
- schema migrations can be applied and verified from an empty local database;
- seed data can be reloaded without duplicating records or producing financial postings.
