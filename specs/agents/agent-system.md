# Agent System

## Topology

The system uses a supervisor with specialized agents. The supervisor owns routing and workflow progression but does not bypass policy or action controls.

| Agent | Responsibility | Mutating access |
|---|---|---|
| Correlation | Group related signals and explain membership | None |
| Context enrichment | Gather service, change, topology, knowledge, and historical context | None |
| RCA | Rank testable cause hypotheses | None |
| Remediation planning | Build an executable but unapproved plan | None |
| Verification | Assess effect and unintended impact | Read-only |
| Agent observability | Analyze monitored-agent traces and evaluation metrics; detect quality drift, groundedness regression, and cost or latency anomalies (UC2) | None |
| Agent security | Triage injection, jailbreak, PII-leakage, and anomalous-tool-call detector output into security findings (UC2) | None |
| Supervisor | Sequence work and enforce state transitions (deterministic, not a model) | None |

The agent observability and agent security roles are additions for the Agent Observability and Security use case (UC2); the correlation, context-enrichment, RCA, remediation-planning, and verification roles serve both use cases. See `agent-orchestration.md` for the phase-by-phase mapping of agents to the automation for UC1 and UC2.

Production mutations are performed by deterministic action workers through the action gateway, not by the remediation planning agent. The customer-proposed single "remediation agent" is deliberately split into a reasoning Remediation Planning Agent (no mutating access) and deterministic action workers.

## Orchestration Rules

- The orchestrator, not the model, is authoritative for state.
- An agent receives an immutable task snapshot and returns a typed result.
- Results are proposals until validated and committed.
- The orchestrator sets iteration, time, token, and tool budgets.
- Parallel agents may analyze the same incident, but conflicts are retained rather than silently overwritten.
- Agents cannot dynamically grant tools to themselves or other agents.
- A failed or unavailable agent produces a classified failure and deterministic fallback.

## Agent Lifecycle

1. Develop against synthetic and historical cases.
2. Evaluate offline against versioned datasets.
3. Deploy in shadow mode.
4. Enable recommendation-only operation.
5. Enable limited production routing.
6. Expand scope only after approved quality and safety evidence.
7. Deprecate with migration and rollback support.

## Context Assembly

Context is assembled from:

- Current incident snapshot.
- Selected evidence and source quality status.
- Time-valid service topology.
- Recent changes and deployments.
- Approved runbooks and knowledge results.
- Prior similar incidents, with outcome labels.
- Tool descriptions and policy constraints required for the task.

Context assembly must enforce access policy before retrieval and again before prompt construction. Retrieved documents and tool results are untrusted content and cannot redefine system instructions, permissions, or output schemas.

## Memory

- **Working memory:** Incident-scoped, short-lived orchestration state.
- **Operational memory:** Persisted incident facts, evidence, decisions, and outcomes.
- **Knowledge memory:** Approved indexed runbooks, documents, and post-incident reviews.
- **Feedback memory:** Labeled operator responses and evaluation results.

Free-form model memory is not a system of record. Production behavior changes only through a reviewed versioned release.

## Model Strategy

- Select models per task quality, latency, data handling, and cost requirements.
- Use deterministic logic for parsing, validation, policy, arithmetic, and state transitions.
- Use smaller or specialized models when evaluation shows they meet thresholds.
- Support model replacement through stable agent contracts.
- Do not assume confidence scores are calibrated across model versions.

## Failure Modes

Required handling includes hallucinated evidence, stale context, tool timeout, malformed structured output, retrieval poisoning, prompt injection, excessive iteration, model unavailability, conflicting agents, and partial source availability.

The default failure mode is to preserve evidence, mark uncertainty, and escalate—not to invent missing context or continue with expanded authority.
