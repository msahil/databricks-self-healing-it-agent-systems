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
