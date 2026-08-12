# Solution Overview

## Objective

Reduce mean time to acknowledge, diagnose, remediate, and verify IT incidents by combining governed enterprise telemetry, Databricks lakehouse capabilities, specialized agents, deterministic orchestration, and policy-controlled automation.

The first release of this solution accelerator is scoped to an energy-retail context through two target use cases:

- **[UC1 — Self-Healing IT Systems](../use-cases/uc1-self-healing-it.md):** heal the customer-facing digital and application estate (New Relic-driven).
- **[UC2 — Agent Observability and Security](../use-cases/uc2-agent-observability-security.md):** observe and secure the customer-facing GenAI assistant (MLflow-driven).

Both use cases share one spine — telemetry, correlation, enrichment, RCA, policy-gated action, verification, and audit — pointed at services in UC1 and at the AI agents themselves in UC2.

## Target Outcomes

- Detect and group related signals before alert volume overwhelms operators.
- Produce evidence-backed incident summaries and ranked root-cause hypotheses.
- Recommend safe, context-aware remediation actions.
- Execute approved low-risk actions and verify their outcomes.
- Learn from operator decisions and completed incidents without silently changing production policy.
- Maintain a complete audit trail for data access, reasoning, approvals, tool calls, changes, and outcomes.

## Personas

| Persona | Primary need | Use case |
|---|---|---|
| Site reliability engineer | Fast evidence collection, diagnosis, and mitigation | UC1 |
| Application / digital product owner | Service-specific context and controlled remediation | UC1 |
| Payment & change operations | Payment-impact awareness, approvals, maintenance windows, rollback evidence | UC1 |
| AI/ML platform owner | Agent health, quality drift, versioning, and rollback | UC2 |
| AI governance & risk | Evaluation gates, autonomy policy, and safe behavior | UC2 |
| Security analyst | Agent-security findings linked to operational context | UC2 |
| Data protection / privacy officer | PII handling, erasure, and vulnerable-customer care | UC2 |
| Platform administrator | Reliable ingestion, governance, cost, and policy control | UC1 + UC2 |
| Auditor | Reconstructable decisions and actions | UC1 + UC2 |

## Target Use Cases

The first release is narrowed to two use cases, each specified in full under `specs/use-cases/`.

### UC1 — Self-Healing IT Systems

Detect, diagnose, and safely remediate degradations across the energy retailer's customer-facing IT estate (app/portal, billing/payments, CRM, meter data management, integration middleware, cloud). New Relic is the primary observability source; remediation examples include rolling back a bad deploy, scaling out, restarting a stateless component, or failing over — all reversible and policy-gated. Full specification: [uc1-self-healing-it](../use-cases/uc1-self-healing-it.md).

### UC2 — Agent Observability and Security

Observe the health and enforce the safety of the retailer's customer-facing GenAI assistant: trace and evaluate every interaction (MLflow 3), detect quality drift, and stop prompt injection, PII leakage, and unauthorized financial actions before they reach a customer. Full specification: [uc2-agent-observability-security](../use-cases/uc2-agent-observability-security.md).

The earlier exploratory use cases (failed Databricks workload, deployment degradation, cloud degradation, and security-finding triage) are folded into these two: workload/deployment/cloud remediation are incident classes within UC1, and security is refocused from generic IT findings onto the AI agents themselves in UC2.

## Operating Principles

1. **Evidence before conclusion:** Agents must cite retrievable evidence identifiers.
2. **Plans before actions:** Reasoning agents do not directly perform production changes.
3. **Policy before execution:** A deterministic policy decision precedes every mutating tool call.
4. **Verify every change:** Success is determined from post-action service evidence, not an API `200` response.
5. **Prefer reversibility:** Autonomous actions must have tested rollback or a documented safe-stop behavior.
6. **Bound context:** Agents receive the minimum data and tools required for the task.
7. **Separate recommendation from authority:** Model confidence never grants permission.
8. **Progressive autonomy:** Authority expands only from measured reliability and explicit approval.

## Success Measures

- Reduction in duplicate alerts reaching operators.
- Reduction in median time to acknowledge and diagnose.
- Percentage of hypotheses accepted by operators.
- Percentage of recommended plans approved without modification.
- Successful remediation and verification rate.
- Rollback, escalation, and unsafe-action prevention rates.
- Operator time saved per supported incident class.
- Cost per investigated and resolved incident.

## Constraints

- Enterprise systems remain authoritative for their owned records.
- ServiceNow remains the initial incident/change system of record unless superseded by an approved decision.
- Databricks is the governed analytical, intelligence, agent, and evidence platform.
- Integrations may expose API, MCP, A2A, event, or workflow interfaces, but all mutating access passes through the action gateway.
- Regional, residency, retention, and segregation requirements must be configurable by deployment.
