# 5. API Contracts for Four API Groups

## 5.1 Shared API Conventions

- Base path: `/api/v1`
- JSON payloads, UTF-8 encoding.
- Consistent error envelope.
- Use correlation ID in request and response headers.
- Pagination via `page`, `pageSize`, and `sort` query parameters.
- Idempotency key support for POST and PATCH operations with financial effect.
- Public HTTP traffic is routed through Kong Gateway; resource-level authorization is still enforced by the backend.
- Kafka is not exposed as an HTTP API and is not called directly by UI/controller request handlers.

## 5.2 Error Contract

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "The request is invalid.",
    "details": [
      { "field": "accountId", "reason": "Required" }
    ],
    "correlationId": "c1d6f3b1-..."
  }
}
```

## 5.3 API A: Advisor and Account API

The following is the implemented foundation. Other API groups below remain planned contracts.

### Endpoints
- `GET /api/v1/advisors`
- `GET /api/v1/advisors/{id}`
- `POST /api/v1/advisors`
- `GET /api/v1/households`
- `GET /api/v1/accounts`
- `GET /api/v1/accounts/{id}`
- `POST /api/v1/accounts`
- `PATCH /api/v1/accounts/{id}/status`
- `GET /api/v1/households/{id}`
- `POST /api/v1/households`
- `GET /api/v1/clients`
- `GET /api/v1/clients/{id}`
- `POST /api/v1/clients`
- `GET /api/v1/dashboard/summary`
- `GET /api/v1/audit/search`
- `POST /api/v1/auth/dev-token` (Development only)

These implemented endpoints require a bearer token except for the Development-only token endpoint. Reads and writes use role policies; status transitions that require approval are audited.

### Key Contracts
- Advisor creation requires firm reference and user identity.
- Account creation requires household, client, custodian reference, and account type.
- Onboarding approval is explicit and auditable.
- Every status change records the actor and reason.

## 5.4 API B: Portfolio and Transaction API

### Endpoints
- `GET /api/v1/securities`
- `GET /api/v1/accounts/{id}/positions`
- `GET /api/v1/transactions`
- `POST /api/v1/trades`
- `PATCH /api/v1/trades/{id}/status`
- `POST /api/v1/transactions/imports`

### Key Contracts
- Security reference data must include symbol, asset class, market price, and currency.
- Trade status values include Draft, Submitted, Approved, Executed, Settled, Rejected, Canceled.
- Import operations must preserve source file metadata and reject invalid rows with reasons.

## 5.5 API C: Money Movement and Billing API

### Endpoints
- `POST /api/v1/transfers`
- `GET /api/v1/transfers/{id}`
- `PATCH /api/v1/transfers/{id}/approval`
- `POST /api/v1/fees/calculate`
- `GET /api/v1/fees/{id}`
- `POST /api/v1/billing/adjustments`

### Key Contracts
- Transfer requests must be versioned and include approval metadata.
- Fee calculation output includes schedule version, execution id, inputs, and amount.
- Approval and reversal operations require authorized user identity and reason code.

## 5.6 API D: Operations and Reconciliation API

### Endpoints
- `GET /api/v1/reconciliation/breaks`
- `POST /api/v1/reconciliation/breaks/{id}/resolve`
- `GET /api/v1/pipelines/runs`
- `POST /api/v1/pipelines/runs`
- `GET /api/v1/pipelines/runs/{id}/status`
- `GET /api/v1/audit/search`

### Key Contracts
- Reconciliation breaks must include source evidence and severity.
- Pipeline run requests return HTTP 202 with an execution reference.
- Audit search supports filters by entity type, date range, and actor.

## 5.7 Authentication and Authorization

- Use OAuth2/OIDC or equivalent enterprise identity service.
- Roles include Advisor, ClientService, OperationsAnalyst, ComplianceReviewer, BackOffice, Admin, Reader.
- Sensitive endpoints require maker-checker flow and explicit separation of duties.

## 5.8 Acceptance Criteria

- Every endpoint has clear responsibility and stable resource naming.
- All financial effect endpoints support idempotency and audit metadata.
- Error semantics are consistent across API groups.
- Operational endpoints clearly distinguish accepted, in-progress, and failed states.
