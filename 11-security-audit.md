# 11. Security, Authorization, and Audit Matrix

## 11.1 Security Architecture

- Authentication using standard enterprise identity integration.
- Role-based authorization with least-privilege mapping.
- Separate permissions for read, write, approve, reverse, and administrative action.
- Audited access for sensitive financial actions.
- Kong Gateway enforces the agreed north-south routing, TLS, request limits, and authentication integration; the backend remains responsible for resource-level authorization.
- Kafka uses TLS and least-privilege per-service identities/ACLs in deployed environments; local credentials are synthetic only.
- Kubernetes workloads use scoped service accounts and AWS IAM roles for service accounts (or the approved EKS Pod Identity mechanism) rather than shared cloud credentials.

## 11.2 Sensitive Financial Actions

Maker-checker approval is required for:
- transfer approval or reversal
- advisory fee approval and posting
- ledger correction or reversal
- account closure and status override
- pipeline re-run after failed financial result publication

## 11.3 Audit Fields

Every financial record must capture:
- actor identity
- timestamp
- reason code or business purpose
- source workflow or event id
- correlation id
- prior state and resulting state

## 11.4 Audit Matrix

| Action | Actor | Approval Path | Audit Record | Protection |
|---|---|---|---|---|
| Account creation | Advisor/Admin | Role-based | Yes | Immutable history |
| Transfer request | Client/Advisor | Maker-checker | Yes | Secure state transitions |
| Fee calculation | Worker | Approval before posting | Yes | Reproducible execution metadata |
| Reconciliation resolution | Ops Analyst | Verified workflow | Yes | Source evidence preserved |
| Journal posting | Backend service | Domain validation | Yes | Immutable ledger |

## 11.5 Security Review Expectations

- Synthetic data only; no real customer personal data.
- Secrets not committed to repo.
- Redact tokens and credentials from logs.
- Preserve tamper-evidence controls for audit and ledger data in deployed environments.

## 11.6 Acceptance Criteria

The security and audit design is acceptable when:

- sensitive actions require explicit approval;
- every financial action is auditable;
- logs and traces cannot expose secrets;
- a future production deployment path includes additional legal and privacy review obligations.
