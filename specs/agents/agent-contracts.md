# Agent Contracts

## Common Input

```json
{
  "task_id": "uuid",
  "task_type": "rca.generate_hypotheses",
  "incident_id": "uuid",
  "incident_version": 12,
  "requested_by": "orchestrator",
  "deadline": "timestamp",
  "budgets": {
    "max_elapsed_seconds": 120,
    "max_tool_calls": 10,
    "max_iterations": 4
  },
  "allowed_skill_versions": ["telemetry-query:1.2.0"],
  "evidence_ids": ["e1", "e2"],
  "policy_context": {
    "environment": "production",
    "data_classes": ["internal"]
  }
}
```

## Common Output

```json
{
  "task_id": "uuid",
  "status": "completed|inconclusive|failed",
  "result_schema_version": "1.0",
  "proposed_incident_patch": [],
  "evidence_ids": [],
  "warnings": [],
  "quality": {
    "confidence": 0.0,
    "evidence_coverage": 0.0
  },
  "agent_version": "agent-name:1.0.0",
  "trace_id": "string"
}
```

The orchestrator rejects output that is malformed, references inaccessible evidence, exceeds authority, or targets a stale incident version.

## Correlation Agent

### Inputs

- Candidate events and alerts.
- Time-valid topology.
- Existing open situations.
- Correlation window and service policy.

### Outputs

- Proposed situation memberships.
- Membership reason codes.
- Confidence per membership.
- Suggested merge or split operations.
- Signals that should remain independent.

### Prohibited behavior

- Suppressing source records.
- Closing incidents.
- Changing severity without an explicit severity proposal.

## Context Enrichment Agent

### Inputs

- Incident scope.
- Approved source precedence.
- Allowed read-only skills.

### Outputs

- Context assertions with source, freshness, and confidence.
- Conflicts and missing context.
- Recommended additional retrieval tasks.

### Prohibited behavior

- Resolving source conflicts without recording alternatives.
- Returning secrets or access-restricted content.

## RCA Agent

### Inputs

- Incident and enriched context.
- Evidence graph.
- Similar incidents.
- Source-quality state.

### Outputs

- Ranked hypotheses.
- Supporting and contradicting evidence.
- Assumptions and diagnostic tests.
- Explicit inconclusive result when warranted.

### Quality gates

- Every hypothesis references evidence.
- At least one alternative hypothesis is considered for high-severity incidents unless only one is technically possible and documented.
- Confirmed cause requires a deterministic verification rule or operator confirmation.

## Remediation Planning Agent

### Inputs

- Approved or probable hypothesis.
- Current service state and risk context.
- Registered action catalog.
- Applicable policy constraints.

### Outputs

- One or more remediation plan candidates.
- Risk, blast radius, dependencies, preconditions, verification, and rollback.
- A no-safe-plan outcome when appropriate.

### Prohibited behavior

- Calling mutating tools.
- Inventing an unregistered action.
- Treating confidence as approval.

## Verification Agent

### Inputs

- Approved plan and action results.
- Pre-action baseline.
- Independent health queries.
- Stabilization and rollback rules.

### Outputs

- Verified success, failure, regression, or inconclusive status.
- Evidence and metric comparison.
- Rollback recommendation or escalation.

### Independence

Verification should use separate queries and, where practical, separate code paths from execution. It must not rely solely on executor-supplied summaries.

## Skill Contract

Every skill registry entry MUST define:

- Name, owner, version, lifecycle status, and description.
- Input and output JSON schemas.
- Read-only or mutating classification.
- Target systems and environments.
- Required workload identity and permissions.
- Rate limits, timeout, retries, idempotency behavior, and side effects.
- Data classifications and logging rules.
- Preconditions, expected errors, and safe failure behavior.
- Test suite, evaluation evidence, and last validation date.
