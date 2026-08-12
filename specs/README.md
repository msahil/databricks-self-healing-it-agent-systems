# Self-Healing IT and Agentic Systems Specifications

Status: First draft, hardened by independent review RT-001
Version: 0.2
Date: 2026-08-12

## Purpose

This specification set defines a Databricks-centered platform that turns enterprise telemetry into correlated incidents, evidence-backed root-cause hypotheses, governed remediation plans, verified actions, and reusable operational knowledge.

The design starts from the customer architecture shared on 2026-08-11. It preserves the proposed front end, integration and orchestration backend, event bridge, agent roles, registries, and enterprise-system integrations while adding the data contracts, safety boundaries, governance, evaluation, and delivery controls required for production use.

## Design Position

The system is an incident intelligence and automation platform, not an unconstrained autonomous agent. Deterministic workflow control surrounds probabilistic reasoning. Every material conclusion must have evidence, every action must be policy checked, and every automated change must be observable, idempotent, and reversible where technically possible.

## Reading Order

1. [Solution overview](architecture/00-solution-overview.md)
2. [Reference architecture](architecture/01-reference-architecture.md)
3. [Functional requirements](requirements/functional-requirements.md)
4. [Non-functional requirements](requirements/non-functional-requirements.md)
5. [Telemetry and incident model](data/telemetry-and-incident-model.md)
6. [Data quality and profiling](data/data-quality-and-profiling.md)
7. [Agent system](agents/agent-system.md)
8. [Agent contracts](agents/agent-contracts.md)
9. [Integration contracts](integrations/integration-contracts.md)
10. [Security, safety, and autonomy](governance/security-safety-autonomy.md)
11. [Evaluation, testing, and SLOs](operations/evaluation-testing-slos.md)
12. [Implementation roadmap](roadmap/implementation-roadmap.md)
13. [Architecture decisions](adrs/ADR-001-deterministic-orchestration.md)
14. [Independent red-team review RT-001](reviews/red-team-review-001.md)

## Scope

### In scope

- IT and platform telemetry ingestion and normalization.
- Security findings from Lakewatch-compatible interfaces.
- Incident detection, correlation, enrichment, diagnosis, and remediation.
- ServiceNow, Atlassian, GitLab, cloud, Databricks, and enterprise tool integration.
- Human-in-the-loop approvals and bounded autonomous remediation.
- Databricks-native governance, data quality monitoring, data profiling, agent evaluation, and operational analytics.
- Auditable incident and action histories.

### Out of scope for the first release

- Enterprise-wide autonomous remediation.
- Replacement of ServiceNow, existing observability tools, or security systems of record.
- Fully autonomous high-impact infrastructure changes.
- Generic conversational assistance unrelated to supported incidents.
- Training a foundation model from scratch.

## Terminology

- **Situation:** A set of potentially related operational signals before an incident is confirmed.
- **Incident:** A tracked degradation, interruption, security event, or policy violation requiring investigation or action.
- **Evidence:** An immutable reference to telemetry, topology, change, knowledge, or tool output supporting a claim.
- **Hypothesis:** A ranked, testable explanation for an incident.
- **Remediation plan:** An ordered set of proposed actions, preconditions, expected effects, risks, and rollback steps.
- **Skill:** A versioned capability exposed to an agent, normally backed by a governed API, MCP server, query, or workflow.
- **Autonomy level:** The maximum action authority allowed for a specific service and action class.
- **Data profiling:** The current Databricks term for profiling Delta tables under Unity Catalog data quality monitoring. “Lakehouse Monitoring” is retained only when referring to historical material.
- **Lakewatch:** Databricks security/SIEM capability used as a source and destination for security-oriented findings and investigations; it is not treated as the sole source of general IT observability.

## Specification Conventions

- Requirements use stable identifiers such as `FR-COR-001` or `NFR-SEC-001`.
- `MUST`, `SHOULD`, and `MAY` have their RFC 2119 meanings.
- Every implementation increment must identify the requirements it satisfies.
- Open assumptions must be resolved before the relevant production gate.

## Source Material

- Customer architecture: “From observability to self-healing IT with Databricks,” dated 2026-08-11.
- Databricks documentation for Agent Framework, Databricks Apps, Unity Catalog, system tables, AI Search, MLflow 3, data quality monitoring, and data profiling.
- Databricks public material describing Lakewatch.

## Review Status

This draft is suitable for architecture and product review. It is not yet an approved production design. Open decisions and red-team findings are tracked under `specs/adrs/` and `specs/reviews/`.

The first independent red-team pass ([RT-001](reviews/red-team-review-001.md)) has been completed. It raised 27 findings (nine Critical or High), applied the clearly-correct hardening to this baseline (new `FR-*` and `NFR-*` requirements, a corrected incident state machine, the Lumos integration contract, event-authenticity and loop-prevention controls, and evidence/erasure/clock-skew tightening), and recorded six Residual Open Decisions (RD-1..RD-6) for the customer and architecture board. These open decisions must be resolved before the relevant production gates.
