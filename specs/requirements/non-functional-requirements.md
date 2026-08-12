# Non-Functional Requirements

## Availability and Resilience

- `NFR-AVL-001` A failure in an agent, model endpoint, retrieval index, or source connector MUST NOT prevent incident records from being created and viewed.
- `NFR-AVL-002` The event-processing path MUST support replay from a durable checkpoint.
- `NFR-AVL-003` Mutating workflows MUST resume safely after process restart without duplicating completed actions.
- `NFR-AVL-004` Each external dependency MUST define timeout, retry, backoff, circuit-breaker, and dead-letter behavior.
- `NFR-AVL-005` The production emergency-stop path MUST not depend on the model-serving path.

## Performance

- `NFR-PER-001` High-severity events SHOULD become visible in the front end within 60 seconds of successful receipt.
- `NFR-PER-002` Initial enrichment SHOULD complete within 90 seconds for 95 percent of supported incidents, excluding unavailable sources.
- `NFR-PER-003` The user-facing API SHOULD return non-streaming requests within 3 seconds at the 95th percentile, excluding explicitly asynchronous operations.
- `NFR-PER-004` Long-running analysis MUST publish progress and remain cancelable where safe.

## Scalability

- `NFR-SCL-001` Ingestion, normalization, correlation, and agent workloads MUST scale independently.
- `NFR-SCL-002` The design MUST avoid per-incident dedicated infrastructure.
- `NFR-SCL-003` Hot operational state and historical analytical state MAY use different serving patterns while retaining a common source of truth.

## Security and Privacy

- `NFR-SEC-001` All access MUST be authenticated and authorized using enterprise identity and least privilege.
- `NFR-SEC-002` Agent access MUST be evaluated using the agent workload identity, initiating user identity where applicable, and target-resource policy.
- `NFR-SEC-003` Secrets MUST NOT appear in prompts, traces, event payloads, source control, or user-visible error messages.
- `NFR-SEC-004` Sensitive fields MUST support classification, masking, row filtering, and retention controls.
- `NFR-SEC-005` Prompt injection and tool-output injection controls MUST treat retrieved content and external tool output as untrusted data.
- `NFR-SEC-006` Egress to models, tools, and integrations MUST be explicitly allowlisted.

## Auditability

- `NFR-AUD-001` The system MUST reconstruct who or what observed, inferred, approved, changed, and verified each incident state.
- `NFR-AUD-002` Audit records MUST be append-only and protected from the identities whose activity they record.
- `NFR-AUD-003` Model, prompt, agent, skill, policy, schema, and workflow versions MUST be recorded for every decision.
- `NFR-AUD-004` Clock synchronization and timestamp provenance MUST be documented for every event source.

## Explainability and Quality

- `NFR-QLT-001` Material claims MUST reference evidence identifiers.
- `NFR-QLT-002` The user experience MUST label uncertainty and avoid presenting hypotheses as confirmed facts.
- `NFR-QLT-003` Production agent versions MUST meet use-case-specific evaluation thresholds before deployment.
- `NFR-QLT-004` Regression evaluation MUST run when models, prompts, tools, retrieval indexes, schemas, or policies materially change.

## Portability and Maintainability

- `NFR-MNT-001` Source-specific behavior MUST be isolated in connectors and skills.
- `NFR-MNT-002` Canonical schemas MUST not embed one vendor's field names as the primary domain model.
- `NFR-MNT-003` Infrastructure, jobs, application configuration, and permissions SHOULD be declaratively deployed.
- `NFR-MNT-004` Backward compatibility and migration rules MUST exist for persisted schemas and event contracts.

## Cost

- `NFR-CST-001` Token, model endpoint, query, storage, streaming, and tool costs MUST be attributable by use case and incident.
- `NFR-CST-002` The orchestrator MUST enforce budgets for agent iterations, tool calls, retrieved context, and elapsed time.
- `NFR-CST-003` The system SHOULD use deterministic processing and smaller models when they meet quality requirements.

## Recovery Objectives

Initial targets to validate during design:

| Component | RPO | RTO |
|---|---:|---:|
| Raw event and audit data | 0-5 minutes | 4 hours |
| Incident state | 1 minute | 1 hour |
| User experience | Not applicable | 2 hours |
| Agent analysis | Replayable | 4 hours |
| Action gateway | No duplicate action | 1 hour |

Final values require business impact analysis and service-tier approval.

## Hardening Requirements (added by Review RT-001)

These requirements were added by the first independent red-team review (`specs/reviews/red-team-review-001.md`) to specify the availability and enforcement of the safety-critical control components. Each cites the finding it resolves.

### Control-Plane Availability

- `NFR-AVL-006` Processing of a single incident MUST be serialized to a single writer. Concurrent orchestrator instances MUST use leader election, leases, or optimistic version checks on `incident.version` such that no state transition is silently overwritten and no incident is processed by two writers simultaneously. (RT-01)
- `NFR-AVL-007` The action gateway MUST fail closed: on gateway degradation or outage, new mutating calls MUST be blocked, pending requests MUST be preserved in an ordered durable queue, and the condition MUST alert. The gateway MUST have its own availability SLO and be monitored independently of the agent and orchestration paths. (RT-01)

### Autonomy Control

- `NFR-AUT-001` Automatic autonomy reduction on threshold breach MUST apply hysteresis and rate limits to prevent flapping, MUST notify affected service owners, and MUST be audited. Autonomy may be reduced automatically but MUST only be increased or restored through explicit approval. (RT-18)
- `NFR-AUT-002` The emergency-stop path MUST block new mutating calls within a bounded activation time (initial target: 5 seconds), MUST be triggerable by defined operator and security roles, and MUST depend on neither the model-serving path nor the analytical orchestration path. Emergency-stop and reconciliation procedures MUST be exercised on a defined schedule. (RT-02)
- `NFR-AUT-003` Approval decisions MUST be presented with a decision-complete summary (targets, blast radius, rollback, expiry). The platform MUST monitor approval latency and throughput to detect rubber-stamping, MUST require a second approver for R2 and above, and MUST record approver dwell time as an audit signal. (RT-11)

### Security and Privacy

- `NFR-SEC-007` Prompt-injection and tool-output-injection defenses MUST be mechanized, not advisory. Trusted instructions MUST be structurally separated from untrusted retrieved content and tool output; tool selection MUST be enforced by the gateway from an orchestrator-supplied allowlist and MUST NOT be inferable from prompt content; agent outputs MUST be schema-constrained so free text can never become a tool invocation. (RT-03)
- `NFR-SEC-008` The tenancy model MUST be explicitly declared and enforced at the Unity Catalog boundary (separate catalogs, row filtering, and workload identities as appropriate). Cross-tenant correlation of situations or incidents is prohibited unless explicitly governed and approved. (RT-12)
- `NFR-SEC-009` Operator labels that feed evaluation MUST be attributable and tamper-evident. Bulk or statistically anomalous labeling activity MUST be detected and reviewed before it affects evaluation datasets or promotion decisions. (RT-19)
- `NFR-SEC-010` Data-subject erasure MUST be reconciled with append-only audit through crypto-erasure or tokenization of personal fields — erasing the key or token while preserving the immutable record. The data-subject-erasure process MUST be defined before production. (RT-14)

### Explainability and Quality

- `NFR-QLT-005` Confidence calibration MUST be measured per agent version (for example, expected calibration error or reliability diagrams). Autonomy and promotion gates MUST rely on outcome-validated metrics, not on raw, uncalibrated model confidence. (RT-08)
- `NFR-QLT-006` Evaluation MUST distinguish operator acceptance from confirmed-outcome correctness (post-incident confirmed cause and verified remediation). Promotion to bounded autonomy (A4) MUST use confirmed outcomes, not acceptance rate alone. (RT-09)
- `NFR-QLT-007` Audit replay MUST reconstruct decisions from recorded agent inputs and outputs rather than by re-inference. Model, version, and generation parameters MUST be pinned and recorded for every decision so a decision is reproducible as a verification of record. (RT-17)

### Cost

- `NFR-CST-004` Exhaustion of any agent budget (iterations, tool calls, retrieved context, or elapsed time) MUST produce a classified `inconclusive` result and escalation. Budget exhaustion MUST NOT yield a silently truncated conclusion presented as complete, nor grant expanded authority. (RT-16)

### Auditability

- `NFR-AUD-005` The maximum tolerated clock skew per event source MUST be documented, and correlation windows MUST widen to account for it. Events whose timestamps fall outside the tolerated skew MUST be flagged and de-weighted in correlation. (RT-15)
