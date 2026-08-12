# Evaluation, Testing, and SLOs

## Evaluation Strategy

MLflow 3 traces and evaluation assets should record agent inputs, outputs, retrieval, tool calls, latency, token use, errors, feedback, and version metadata. Evaluation datasets are versioned and segmented by use case, severity, service, environment, and failure mode.

## Core Metrics

### Correlation

- Pairwise and cluster precision and recall.
- Duplicate-alert reduction.
- Incorrect merge and split rate.
- Time to stable situation grouping.

### Context enrichment

- Required-field coverage.
- Source freshness.
- Ownership and topology mapping accuracy.
- Permission-filtering failures.

### RCA

- Top-1 and top-3 accepted hypothesis rate.
- Evidence precision and coverage.
- Unsupported claim rate.
- Confirmed-cause calibration.
- Correct inconclusive and escalation rate.

### Remediation planning

- Operator acceptance and edit rate.
- Invalid or unavailable action rate.
- Missing precondition, verification, or rollback rate.
- Risk and blast-radius classification accuracy.

### Execution and verification

- Successful verified remediation rate.
- Duplicate action rate.
- Unknown-result and partial-failure rate.
- Rollback rate and rollback success.
- False-success and unintended-impact rate.

### Business outcomes

- Mean and median time to acknowledge, diagnose, mitigate, and resolve.
- Operator minutes per incident.
- Repeat-incident rate.
- Service downtime avoided.
- Cost per incident and per successful remediation.

## Test Layers

1. **Schema tests:** Contracts, compatibility, and malformed payloads.
2. **Unit tests:** Deterministic transformations, policy, and state transitions.
3. **Connector tests:** Authentication, pagination, rate limits, replay, and reconciliation.
4. **Agent evaluations:** Historical, synthetic, adversarial, and counterfactual cases.
5. **Workflow tests:** Timeouts, retries, stale state, duplicate events, and partial failure.
6. **Safety tests:** Prompt injection, tool injection, privilege escalation, scope escape, and secret leakage.
7. **Game days:** Controlled failure and remediation exercises in non-production and approved production scopes.
8. **Disaster recovery tests:** Restore, replay, duplicate prevention, and emergency stop.

## Required Adversarial Cases

- A ticket contains instructions to ignore policy.
- A runbook contains a malicious tool command.
- Telemetry is delayed while appearing syntactically valid.
- Two deployments overlap across dependent services.
- An action API times out after applying the change.
- An approver authorizes an old plan version.
- A high-confidence RCA is contradicted by authoritative evidence.
- A source system maps one asset to two services.
- Model output includes a tool not in the registry.
- Verification shows local recovery but downstream degradation.

## Initial Service-Level Objectives

These are draft targets requiring workload validation.

| SLO | Initial target |
|---|---:|
| Accepted high-severity event visible | 99% within 60 seconds |
| Incident API availability | 99.9% monthly |
| Action audit completeness | 100% |
| Unauthorized mutating action | 0 |
| Duplicate mutating action caused by platform | 0 |
| Verified action outcome recorded | 99.9% |
| Evidence-backed RCA output | 100% |

## Release Gates

- No unresolved critical or high safety defects.
- Contract and policy tests pass.
- Use-case-specific offline evaluation thresholds pass.
- Shadow-mode comparison is complete.
- Rollback and emergency-stop tests pass.
- Cost and latency are within approved budgets.
- Audit reconstruction succeeds for sampled cases.
- Service owner, security, governance, and operations approvals are recorded.

## Production Monitoring

- Agent and tool error rate.
- Output schema rejection rate.
- Retrieval freshness and empty-result rate.
- Confidence and outcome drift.
- Token and cost anomalies.
- Policy denial and approval-expiry rate.
- Action success, unknown result, rollback, and verification latency.
- Data quality degradation and missing-source rate.

Threshold breaches may reduce autonomy automatically, but increasing autonomy always requires explicit approval.
