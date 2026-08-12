# Telemetry and Incident Model

## Data Layers

### Bronze

Immutable source payloads and receipt metadata. Bronze is the forensic record and replay source. Parsing failures remain queryable rather than being discarded.

### Silver

Normalized entities and relationships:

- `asset`
- `service`
- `service_dependency`
- `identity`
- `telemetry_event`
- `alert`
- `security_finding`
- `change`
- `deployment`
- `maintenance_window`
- `knowledge_document`

Service-health signals (UC1):

- `synthetic_check_result`
- `service_level_signal`

Monitored-agent telemetry (UC2):

- `monitored_agent`
- `agent_interaction`
- `agent_tool_call`
- `agent_evaluation_result`
- `agent_finding`

### Gold

Operational intelligence and outcomes:

- `situation`
- `situation_member`
- `incident`
- `incident_event`
- `evidence`
- `hypothesis`
- `remediation_plan`
- `action_execution`
- `approval`
- `verification_result`
- `operator_feedback`
- `agent_run`
- `incident_outcome`

## Canonical Event Envelope

```json
{
  "event_id": "uuid",
  "event_type": "deployment.completed",
  "event_version": "1.0",
  "source_system": "gitlab",
  "source_event_id": "string",
  "tenant_id": "string",
  "event_time": "timestamp",
  "ingested_at": "timestamp",
  "subject": {
    "type": "service",
    "id": "service-id"
  },
  "environment": "production",
  "severity": "info",
  "trace_context": {},
  "classification": ["internal"],
  "payload_ref": "catalog.schema.bronze_table:record-key",
  "integrity": {
    "hash": "sha256",
    "signature_status": "verified|unverified|failed"
  }
}
```

## Canonical Incident Object

The incident object is versioned. Agents return proposed patches; the orchestrator validates and persists them.

```json
{
  "incident_id": "uuid",
  "version": 12,
  "status": "diagnosing",
  "title": "string",
  "summary": "string",
  "severity": "sev2",
  "service_ids": ["service-id"],
  "environment": "production",
  "owner": "team-id",
  "started_at": "timestamp",
  "detected_at": "timestamp",
  "evidence_ids": ["evidence-id"],
  "hypothesis_ids": ["hypothesis-id"],
  "active_plan_id": null,
  "external_records": [
    {"system": "servicenow", "type": "incident", "id": "INC001"}
  ],
  "risk": {
    "business_impact": "high",
    "data_impact": "none-known",
    "security_impact": "unknown"
  },
  "created_by": "orchestrator",
  "created_at": "timestamp",
  "updated_at": "timestamp"
}
```

## Evidence Model

Each evidence item MUST include:

- Stable evidence identifier.
- Evidence type and source system.
- Source record reference or immutable snapshot reference.
- Retrieval query or tool call identifier.
- Event time, retrieval time, and freshness classification.
- Integrity status and content hash where available.
- Access classification and permitted audiences.
- Human-readable summary generated separately from the raw evidence.
- Relationship to the supported or contradicted claim.

Agents must not cite a temporary prompt position or free-form URL as the sole evidence reference.

For mutable sources (for example a ServiceNow ticket or a live cloud configuration), evidence MUST capture an immutable content snapshot with a content hash at retrieval time. A live reference alone is insufficient, because the underlying record can change after the evidence is cited and silently break audit reconstruction (Review RT-013).

## Hypothesis Model

```json
{
  "hypothesis_id": "uuid",
  "incident_id": "uuid",
  "rank": 1,
  "statement": "A deployment introduced an invalid connection pool setting.",
  "confidence": 0.74,
  "cause_type": "probable",
  "supporting_evidence_ids": ["e1", "e2"],
  "contradicting_evidence_ids": ["e3"],
  "assumptions": ["Traffic routing remained unchanged"],
  "tests": ["Compare pool saturation before and after deployment"],
  "agent_version": "rca-agent:1.0.0"
}
```

Confidence is an analytical estimate, not authorization. Thresholds must be calibrated per use case and may not be compared blindly across agents.

## Remediation Plan Model

A remediation plan MUST contain:

- Plan and incident version.
- Goal and expected service effect.
- Target resources and environment.
- Ordered action graph and dependencies.
- Preconditions and freshness limits.
- Risk tier and calculated blast radius.
- Approval requirement and eligible approver roles.
- Expected action duration and timeout.
- Verification queries and stabilization period.
- Rollback actions and rollback triggers.
- Safe-stop behavior when rollback is unavailable.
- Expiry time and invalidation conditions.

## UC1 Service-Health Signals

The generic `telemetry_event`, `alert`, `change`, and `deployment` entities carry most UC1 signal, with two additions and an explicit New Relic mapping.

- `synthetic_check_result` — monitor id, journey (view-bill, login, make-payment), status, latency, failure reason, region, run time. Sourced from New Relic synthetics.
- `service_level_signal` — service id, signal type (golden signal: error rate, latency, throughput, saturation; or business KPI such as payment-success rate), value, window, source, and baseline reference.
- **New Relic mapping:** New Relic entities map to canonical `asset` and `service` identities with mapping confidence and provenance; entity relationships enrich `service_dependency`; deployment markers map to `deployment`; New Relic alerts and incidents map to `alert`. Only events with `signature_status = verified` justify a mutating action (`FR-TEL-007`).

## Agent Observability and Security Model (UC2)

These entities describe the monitored customer-facing GenAI assistant. They are distinct from `agent_run`, which records the platform's own reasoning agents.

- `monitored_agent` — registry-linked identity: agent id, agent version, prompt version, policy version, tool allowlist, data classifications, evaluation-baseline reference, owner.
- `agent_interaction` — interaction/session id, monitored-agent version, authenticated customer scope, input classification (untrusted), retrieved-context references, output reference, latency, token cost, and MLflow trace id.
- `agent_tool_call` — interaction id, tool (skill) and version, argument classification, policy decision (allow/deny with reason), executed flag, idempotency key, and target scope.
- `agent_evaluation_result` — evaluation-dataset version, metric (groundedness, correctness, hallucination, refusal, cost, containment), value, monitored-agent version, and judge-model version.
- `agent_finding` — finding type (`prompt_injection`, `jailbreak`, `pii_leak`, `anomalous_tool_call`, `cost_anomaly`, `quality_drift`), severity, detector and version, session/interaction references, evidence references, and disposition.

An `agent_finding` (security) or a `quality_drift` signal enters the incident spine as an `alert` or `security_finding` feeding correlation; from there UC2 reuses `situation`, `incident`, `evidence`, `hypothesis`, `remediation_plan`, `verification_result`, and `incident_outcome` unchanged. Raw prompts, retrieved content, and tool arguments carry data classification and are minimized or redacted before prompt construction and before appearing in traces (`NFR-SEC-003`, `NFR-SEC-010`).

## Identity and Topology

Every source must map to canonical asset and service identities. Mapping confidence and provenance are stored. Unknown or ambiguous mappings cannot receive autonomous actions. Service dependencies are time-versioned so historical incident replay uses the topology valid at incident time.

## Time and Clock Skew

Correlation and evidence ordering depend on timestamps from heterogeneous sources. The maximum tolerated clock skew per source MUST be documented, and correlation windows MUST widen to account for it (`NFR-AUD-005`). Events whose timestamps fall outside the tolerated skew are flagged and de-weighted rather than trusted for tight time-based correlation.

## Retention

Retention is defined by data class and jurisdiction. Raw payload retention may differ from normalized facts, traces, prompts, and audit records. Legal hold and security-investigation requirements override ordinary deletion schedules through approved governance processes.

Data-subject erasure obligations conflict directly with append-only audit immutability. This conflict MUST be reconciled through crypto-erasure or tokenization of personal fields — erasing the encryption key or token while preserving the immutable audit record (`NFR-SEC-010`). The data-subject-erasure process must be defined before production.
