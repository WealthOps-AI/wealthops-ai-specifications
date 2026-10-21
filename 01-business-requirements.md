# 1. Business Requirements and Domain Glossary

## 1.1 Business Purpose

WealthOps is a fictional, simulation-first wealth-management and brokerage operations platform intended to model advisor servicing, onboarding, portfolio operations, billing, reconciliation, and operational controls. It is designed to teach enterprise software engineering with realistic financial-domain complexities while avoiding any real trading or money movement.

## 1.2 Business Goals

- Support advisor and client servicing from onboarding through account lifecycle management.
- Manage security reference data, holdings, transactions, and portfolio valuations.
- Handle simulated order and money-movement workflows with explicit approval states.
- Calculate advisory fees deterministically and preserve historical reproducibility.
- Post ledger entries only when debits and credits balance and the transaction is authorized.
- Reconcile internal records against synthetic custodian data and surface durable operational break records.
- Provide operational dashboards, pipeline visibility, and auditability for compliance and support teams.

## 1.3 Core Roles

- Advisor: manages client relationships and account activity.
- Client: owns or is associated with household accounts and financial goals.
- Household: a customer grouping for account management and shared servicing.
- Operations Analyst: resolves reconciliation breaks and operational exceptions.
- Compliance Reviewer: approves exceptions and verifies workflows.
- Back-office Processor: owns fee posting, transfers, and ledger corrections.
- Platform/Engineering Admin: manages infrastructure, configuration, and observability.

## 1.4 Primary Business Domains

### Advisor and Account Domain
- Account onboarding and approval.
- Household, advisor, and client relationships.
- External custodian reference mapping.
- Account status lifecycle and servicing actions.

### Portfolio and Transaction Domain
- Securities master data.
- Positions and valuations.
- Trade execution simulation and settlement status.
- Transaction imports and corrections.

### Money Movement and Billing Domain
- Transfer requests, approvals, and reversals.
- Fee schedule versions and billing calculations.
- Posting to accounting records.

### Operations and Reconciliation Domain
- Reconciliation break creation and resolution.
- Pipeline status and operational summaries.
- Exception queue and historical records.

## 1.5 Business In-Scope Processes

- Advisor and client onboarding.
- Custodian account import and matching.
- Trade and cash transaction ingestion.
- Portfolio valuation and exception review.
- Transfer initiation and maker-checker approval.
- Advisory fee calculation and posting.
- Journal posting and ledger balancing.
- Reconciliation with synthetic custodian records.
- Operational dashboards and pipeline visibility.

## 1.6 Business Out of Scope

- Real trading or live financial settlement.
- Real banking, credit, or custodial connectivity.
- Production customer data or regulated financial operations.
- Real financial legal advice or actual fee schedules.

## 1.7 Domain Glossary

- Account: a financial account under an advisor or household.
- Household: a logical grouping of clients and accounts.
- Security: a financial instrument (equity, fund, fixed income, cash equivalent).
- Position: current quantity and valuation of a security held in an account.
- Cash Balance: cash available or in transit for the account.
- Trade: intent to buy or sell a security.
- Execution: a validated trade instruction processed in the simulation.
- Settlement: the transfer of ownership and cash effect after execution.
- Transfer Request: an instruction to move cash or securities between accounts or providers.
- Fee Schedule: versioned policy for advisor fee calculation.
- Journal: a balanced accounting record with debit and credit entries.
- Ledger: the authoritative accounting record for all posted financial effects.
- Reconciliation Break: a difference between internal records and the synthetic custodian record set.
- Pipeline Execution: an Airflow-run metadata record that tracks asynchronous processing.

## 1.8 Financial Integrity Principles

- No partial posting of a journal.
- No hidden modification of posted ledger records.
- Reversals or adjustments require explicit journal entries.
- Duplicate operations must not create duplicate accounting effects.
- All fee calculations must be deterministic and version-aware.

## 1.9 Key Constraints

- Synthetic data only.
- Public-domain business context only.
- Use fictional organizations and institutions.
- Every financial workflow must be traceable to a business reference and user action.

## 1.10 Acceptance Criteria

The business requirements are complete when:

- the major account, trade, fee, and reconciliation workflows are identified;
- financial ownership and accountability are clear;
- all required domain concepts are defined;
- synthetic-only boundaries are explicit;
- all downstream architecture decisions are traceable to these requirements.
