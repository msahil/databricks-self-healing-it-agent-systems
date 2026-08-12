# Implementation Roadmap

## Phase 0: Discovery and Controls

### Goals

- Confirm use cases, systems of record, business impact, data residency, and ownership.
- Inventory telemetry, APIs, MCP servers, identities, actions, and existing runbooks.
- Approve canonical service, event, incident, evidence, and action models.
- Define initial safety policy, autonomy levels, and evaluation baselines.

### Exit criteria

- Both target use cases (UC1 and UC2) have named owners and measurable baselines.
- Source and action access are approved.
- RACI, threat model, and business impact analysis are reviewed.
- No unresolved ambiguity exists about the incident and change systems of record.

## Phase 1: Observe

### Deliverables

- Bronze and Silver ingestion for selected sources.
- Service and asset identity mapping.
- Incident timeline user experience.
- Unity Catalog governance and data quality monitoring.
- Data profiling for critical telemetry tables.
- Audit and operational dashboards.

### Autonomy

`A0 Observe` only.

## Phase 2: Assist

### Deliverables

- Correlation and context-enrichment agents.
- Approved knowledge ingestion and AI Search indexes.
- RCA agent in shadow and recommendation mode.
- MLflow tracing, evaluation datasets, and operator feedback.
- ServiceNow incident linking and updates.

### Autonomy

`A1 Assist`.

## Phase 3: Prepare and Approve

### Deliverables

- Remediation planning and verification agents.
- Versioned skill and action registry.
- Policy and approval engine.
- Action gateway in non-production.
- GitLab, Databricks, and selected cloud action integrations.
- Game-day and rollback validation.

### Autonomy

`A2 Prepare`, then `A3 Approve-to-run` for qualified actions.

## Phase 4: Bounded Autonomy

### Candidate actions

- Retry an idempotent failed workload.
- Pause a repeatedly failing schedule.
- Restart a narrowly scoped stateless component.
- Roll back an approved deployment through an existing protected workflow.
- Apply a temporary, bounded capacity adjustment.

### Entry criteria

- Autonomy promotion gate is satisfied per action class.
- Shadow and approved execution results meet thresholds.
- Emergency-stop, rollback, and reconciliation tests pass.
- Service owner and governance approval is time bounded.

### Autonomy

`A4 Bounded autonomous` only for explicitly qualified actions.

## Phase 5: Scale

- Add services, regions, sources, and action classes through repeatable onboarding.
- Introduce portfolio-wide dependency and incident analytics.
- Improve models using reviewed feedback and evaluation datasets.
- Optimize cost, latency, and deterministic processing.
- Reassess—not assume—the need for broader autonomy.

## First Vertical Slice

The recommended first implementation is [UC1 — Self-Healing IT Systems](../use-cases/uc1-self-healing-it.md), specifically a deploy-related degradation of a customer-facing service:

1. Ingest New Relic alerts, APM and infrastructure metrics, synthetics, deployment markers, and ServiceNow tickets.
2. Normalize service, owner, environment, and dependency identities; link New Relic entities to canonical services.
3. Correlate the degradation, downstream impact, and the recent deployment into one situation.
4. Enrich with the GitLab deploy, configuration, historical incidents, and runbooks.
5. Rank root-cause hypotheses.
6. Recommend rollback, scale-out, restart, or escalation.
7. Execute only an approved, reversible deploy rollback in the first action release.
8. Verify recovery from independent evidence: golden signals, synthetics, and the payment-success KPI.

[UC2 — Agent Observability and Security](../use-cases/uc2-agent-observability-security.md) follows once the tracing, evaluation, and policy-gated action controls from the slice are in place, reusing the same spine against the customer GenAI assistant.

## Delivery Workstreams

- Product and operating model.
- Data and integration engineering.
- Databricks platform and governance.
- Agent and retrieval engineering.
- Front-end and orchestration services.
- Security and action controls.
- Evaluation, quality, and reliability.
- Change management and operator adoption.

## Open Decisions

- Authoritative service topology and ownership source.
- Required Lakewatch interfaces and availability by target region.
- Event bridge technology and enterprise standards.
- Durable orchestration implementation for long-running remediation.
- BPMN requirement versus declarative state-machine workflows.
- Front-end hosting and authentication pattern.
- Supported model providers and data-processing boundaries.
- Final RPO, RTO, SLO, and retention requirements.
- Initial service, region, and action classes.
