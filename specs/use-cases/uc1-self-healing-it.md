# UC1 — Self-Healing IT Systems (Customer Digital and Application Estate)

Status: Target use case for the first release
Domain: Energy retail (market-agnostic; regulatory hooks left pluggable)

## Summary

Detect, diagnose, and safely remediate degradations across the energy retailer's customer-facing IT estate, using New Relic as the primary observability source and Databricks as the intelligence, orchestration, and audit plane.

## Business context

The customer digital estate is the retailer's front door: check usage, pay bills, submit meter reads, report outages, and switch tariff. Demand is spiky — cold snaps, price-cap change dates, and bill-shock periods create surges exactly when reliability matters most. Degradation in those windows means failed payments, complaint spikes, contact-centre overload, regulatory service-standard exposure, churn, and revenue leakage. The estate is heterogeneous (cloud microservices, a third-party payment provider, CRM, meter data management, integration middleware), so the value is correlating scattered signal quickly and acting reversibly.

## Actors

| Actor | Role |
|---|---|
| SRE / platform engineer | Primary operator; reviews RCA, approves and executes |
| Digital / app product owner | Owns the customer-facing service and its context |
| Payment operations | Owns payment-success impact |
| Incident & change manager | Risk, approvals, maintenance windows, rollback evidence |
| Contact-centre operations | Downstream impact awareness |
| Platform administrator | Configures autonomy and policy per service |
| Auditor | Reconstructs decisions and actions |

## Systems and data sources

- **New Relic (primary):** APM, infrastructure metrics, synthetic monitors (view-bill, login, make-payment), browser/RUM, alerts and incidents, entity/service maps, deployment markers, NRQL/NerdGraph.
- **Cloud provider:** resource health, autoscaling state, configuration, audit logs, metrics.
- **GitLab:** pipelines, deployments, commits/merges, protected rollback.
- **ServiceNow:** incident/change records, CMDB service mapping, approvals.
- **Databricks:** system tables (compute/jobs), Unity Catalog data-quality monitors for data dependencies.
- **Confluence:** runbooks. **Payment-provider status:** external dependency signal.

## Preconditions

- Services mapped to canonical service/asset identities with owners and criticality; New Relic entities linked to those identities.
- Action skills registered (GitLab rollback, scale-out, restart, failover, feature-flag/status-banner) with a risk tier and autonomy level per service.
- Read-only investigation identity distinct from the mutating execution identity.

## Trigger

A cold-snap surge drives customers into the app. A deploy 30 minutes earlier shipped a connection-pool misconfiguration on the account/billing API. New Relic opens an alert (error rate and latency), the view-bill and make-payment synthetics fail, RUM shows dashboard load failures, payment success rate drops, and ServiceNow tickets climb.

## Main flow

1. **Detect** — a verified New Relic alert/incident webhook becomes a canonical `alert.opened`, joined with infrastructure metrics and a ticket-volume spike; a situation is created.
2. **Promote** — customer-facing, payment-impacting, and peak-demand criticality meet the promotion criteria (`FR-COR-006`); an incident opens, links the New Relic and ServiceNow incidents, and preserves the situation id.
3. **Enrich** — attach service owner, criticality, the GitLab deploy (commit, author, diff summary captured as an immutable snapshot), maintenance windows, runbook, and prior surge-related incidents with outcomes.
4. **Correlate and disambiguate** — separate a bad deploy from a pure capacity surge: error onset aligns to the deploy marker, not the traffic ramp; payment failures are downstream of the account API.
5. **Diagnose (RCA)** — ranked, evidence-backed hypotheses: deploy regression (probable) > surge amplification (contributing) > payment-provider degradation (alternative), each with supporting/contradicting evidence and a confirming test.
6. **Plan** — candidates from the registry: Plan A, GitLab protected rollback (reversible, blast radius = the service, verification and rollback triggers defined); Plan B, scale-out plus a pre-approved pool-size bump.
7. **Policy and approval** — deterministic policy decision; rollback of a payment-adjacent service requires A3 approval, and because it is payment-impacting (R2) a second approver (`NFR-AUT-003`). The approval binds approver, plan version, scope, and expiry.
8. **Execute** — the action gateway validates the approval token and a fresh precondition (the bad deploy is still the active version), uses a short-lived least-privilege identity, applies idempotency, runs the rollback, records the full request/response, and emits origin-tagged action events (`FR-ACT-009`).
9. **Verify** — from independent evidence: New Relic golden signals recover, synthetics pass, payment success rate returns to baseline, and ticket inflow subsides across a stabilization period. Success is judged from service evidence, not the rollback API response.
10. **Close** — record confirmed cause, actions, outcome, residual risk, and follow-up; produce a draft runbook update that is not auto-published (`FR-KNW-003`).

## Alternate and failure flows

- Rollback fails → escalate (`RollingBack → Escalated`), page on-call.
- Local recovery but downstream still degraded → escalate or add a plan.
- New Relic degraded/unavailable → mark the evidence source degraded, block autonomous action that relies on it, require a human.
- Unknown-result on a scale action (timeout) → reconcile actual capacity before any retry (`FR-ACT-008`).
- Post-action transient blip → suppressed by the post-action window, not re-diagnosed (`FR-ACT-009`).

## Actions, autonomy, and risk

| Action | Risk tier | Initial autonomy |
|---|---|---|
| Observe / correlate / RCA | R0 | A0–A1 |
| Rollback deploy, failover, restart (customer-facing) | R1–R2 | A3 (approve-to-run) |
| Retry idempotent internal job; restart a narrow stateless worker; bounded capacity bump | R0–R1 | A4 candidate (later phase, after the promotion gate) |
| Destructive / irreversible infrastructure change | R4 | Prohibited |

## Out of scope

Irreversible/destructive infrastructure changes; changes to the payment provider's systems; autonomous customer communications; schema migrations.

## Success metrics

MTTA / MTTD / MTTR during peak events; percentage of incidents auto-correlated; top-1/top-3 RCA acceptance; percentage of plans approved unmodified; verified-remediation rate; payment-success recovery time; contact-centre deflection; duplicate-alert reduction; unsafe-action prevention (zero).

## Demo narrative

A peak-event countdown with payments failing; the platform assembles New Relic and GitLab evidence in seconds, presents an evidence-backed RCA, takes one approval, fires the rollback, and shows golden signals plus the payment KPI recovering on the incident timeline with a complete audit trail.

## Databricks mapping

Bronze/Silver/Gold on Delta and Unity Catalog; New Relic/cloud/GitLab/ServiceNow connectors via the event bridge; agents on Agent Framework and Model Serving; orchestration and UI on Databricks Apps; MLflow 3 for the platform's own agents; AI Search for runbooks; data-quality monitors for evidence-source health; system tables for Databricks-side compute evidence.

## Requirements traceability

`FR-TEL-*` (incl. `FR-TEL-007`), `FR-COR-*` (incl. `FR-COR-006`), `FR-CTX-*`, `FR-RCA-*`, `FR-PLN-*`, `FR-ACT-*` (incl. `008`, `009`), `FR-VER-*` (incl. `006`); `NFR-AVL-006/007`, `NFR-AUT-003`, `NFR-AUD-005`. Orchestration detail in [agent-orchestration](../agents/agent-orchestration.md#uc1--self-healing-it-systems-orchestration).
