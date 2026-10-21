# 18. Assumptions, Open Questions, and Deferred Requirements

## 18.1 Assumptions

- The initial system can operate as a modular monolith with future extraction opportunities.
- Settlement and transfers are modeled as domain workflow states rather than real external payment settlement.
- Each account has a synthetic custodian mapping but not a live external integration.
- Fee calculation policies are educational and intentionally fictional.
- EF Core with Npgsql is the ORM for the modern .NET backend.
- Kong is the HTTP API gateway; Kafka/MSK is the sole event backbone.
- Docker Compose is the local runtime; AWS EKS/Kubernetes is the production target.
- Claude/Copilot artifact marketplace design and Temporal implementation are deferred.

## 18.2 Open Questions

- Should all account types share one chart-of-accounts structure or should specialized subledgers be introduced?
- Are all balances and securities denominated in a single base currency or does the architecture need multi-currency semantics from day one?
- Which approval workflow is required for account closure versus transfer exceptions?
- How much of the legacy adapter contract should be formalized before implementation begins?
- Which Compose profiles should be included in the default local quick-start and which services should remain optional?
- Which AWS APIs are essential for local emulation, beyond S3 and Secrets Manager, and can they be covered by no-cost local test doubles?
- Should Kong Gateway itself be deployed through Compose locally and on EKS, or should an AWS edge service precede it in deployed environments?
- Which specific long-running workflows, if any, should later be evaluated for Temporal while Airflow remains responsible for batch pipelines?
- What distribution, review, versioning, and trust model should the future GitHub Copilot and Claude artifact repositories use?

## 18.3 Deferred Requirements

- Real external payments and banking operations.
- Real-time market pricing from production data providers.
- Regulated reporting and audit retention beyond the simulation environment.
- Additional integration providers beyond the initially mocked set.
- Temporal code/infrastructure and all Claude/Copilot marketplace artifacts until separately designed and approved.

## 18.4 Approval Gate Notes

These assumptions and open questions remain intentionally visible so the design can be reviewed before implementation. Any material change should be reflected in the traceability matrix and architecture decision records.
