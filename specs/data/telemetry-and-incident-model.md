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

## Identity and Topology

Every source must map to canonical asset and service identities. Mapping confidence and provenance are stored. Unknown or ambiguous mappings cannot receive autonomous actions. Service dependencies are time-versioned so historical incident replay uses the topology valid at incident time.

## Retention

Retention is defined by data class and jurisdiction. Raw payload retention may differ from normalized facts, traces, prompts, and audit records. Legal hold and security-investigation requirements override ordinary deletion schedules through approved governance processes.
