# 20. Repository Topology and Proposed Structures

## 20.1 Recommendation

Use six active, independently versioned repositories. This is enough separation to model real ownership and release boundaries without splitting the initial modular backend into premature microservices.

| Repository | Owns | Does not own |
|---|---|---|
| `wealthops-web` | React advisor/operations UI, browser tests, API client | Business rules, accounting, secrets |
| `wealthops-backend` | .NET 10 modular monolith, EF Core persistence, migrations, seed tool, ledger, API, two .NET workers | Airflow DAGs, infrastructure |
| `wealthops-legacy-adapter` | .NET Framework 4.8.1/Web API 2 custodian adapter and Windows-compatible packaging | Modern backend database or ledger |
| `wealthops-pipelines` | Containerized Airflow, Python DAGs/modules, ingestion and analytics | Authoritative ledger posting |
| `wealthops-platform` | Local Docker Compose, Terraform, production Kubernetes/Helm, Kong config, AWS deployment, observability/runbooks | Domain implementation |
| `wealthops-contracts` | Versioned OpenAPI, Kafka event and file/data schemas, compatibility policy | Runtime business behavior |

### Why isolate the legacy adapter?

The .NET Framework 4.8.1 target has distinct Windows hosting, build tooling, package compatibility, and release concerns. A separate repository gives it an independently testable compatibility boundary and avoids making the modern backend pipeline depend on Windows build infrastructure.

### Why keep both workers in the backend repository?

The reconciliation and fee workers share domain rules, ledger boundaries, database models, and release coordination with the backend. They can be separate images and deployments without being separate source repositories or services with independent data ownership.

## 20.2 Directory Structure

The proposed detailed trees for the active repositories are in [repository CI and structure](./14-repository-ci.md). Standard top-level conventions for every active repository:

```text
README.md
docs/
src/ or service-specific source directory
tests/
Dockerfile(s)
.github/workflows/
configuration templates without credentials
```

Each repository has its own version, tests, local setup, configuration documentation, CI/CD, and deployment instructions. Deployment ownership is centralized in `wealthops-platform`; source owners publish immutable container images and release metadata.

## 20.3 Contracts and Release Coordination

- Contracts are published as immutable tagged releases with compatibility checks.
- Backend, web, pipelines, and legacy adapter pin a released contract version.
- Breaking API/event changes use explicit major versions and a consumer migration window.
- JSON Schema or an equivalent schema format is used for Kafka events and file contracts; OpenAPI describes HTTP APIs.
- Contract publishing does not require consumers to share source code or participate in one monolithic build.
- Schema registry selection (AWS Glue Schema Registry versus a Kafka-compatible registry) is an open deployment decision; local development must have a validation path without requiring paid services.

## 20.4 Deferred Artifact/Workflow Repositories

These are not active repositories and are not part of the initial implementation:

- `wealthops-copilot`: future home for GitHub Copilot custom agents, skills, prompts, instructions, and related artifacts.
- `wealthops-claude`: future home for Claude-compatible skills/plugins and related artifacts.
- `wealthops-temporal`: possible future workflow implementation only after a Temporal suitability decision.

The Copilot and Claude repositories should be designed as discoverable/versioned artifact catalogs, not assumed to be identical marketplaces. Their exact conventions, installation/distribution experience, trust and review controls, and cross-tool reuse strategy will be discussed separately. This plan does not create their contents.

During that design, assess whether portable skill content (for example, a documented `SKILL.md` plus supporting files) can be shared, while keeping tool-specific agent, plugin, prompt, and marketplace manifests in their respective repositories. Do not assume that a Claude marketplace package is directly installable by Copilot or vice versa; validate the then-current supported distribution mechanisms before setting repository conventions.

## 20.5 Acceptance Criteria

- Each business capability has one authoritative owning repository.
- The legacy adapter has a Windows-capable independent build and release path.
- Shared contracts are versioned and independently consumable.
- The local Compose environment can bring up the agreed application stack from sibling checkouts or pinned images.
- No additional microservice or repository is introduced without an ownership, data, deployment, and operational reason.
- Deferred Copilot, Claude, and Temporal work remains outside initial delivery.
