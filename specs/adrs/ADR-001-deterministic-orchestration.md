# ADR-001: Deterministic Orchestration Around Specialized Agents

Status: Proposed
Date: 2026-08-12

## Context

The customer architecture proposes correlation, context-enrichment, RCA, and remediation agents behind a shared integration and orchestration layer. Agent reasoning is useful for ambiguous operational analysis, but incident state, approvals, side effects, retries, and recovery require predictable behavior.

## Decision

Use deterministic orchestration as the authority for workflow state, budgets, routing, approval, policy, persistence, and action execution. Specialized agents receive typed tasks and return typed proposals. Agents cannot directly mutate production systems.

Production actions are executed by deterministic workers through a policy-controlled action gateway. Verification is a separate stage and should use evidence independent of the executor response.

## Consequences

### Positive

- State transitions are reproducible and auditable.
- Agent or model replacement does not redefine workflow semantics.
- Safety and approval controls remain outside probabilistic reasoning.
- Retries and recovery can prevent duplicate actions.
- Evaluation can isolate reasoning quality from orchestration correctness.

### Negative

- More schemas and workflow code are required.
- Agents have less freedom to invent workflows dynamically.
- Long-running workflow technology must be selected and operated.
- New action types require registry, policy, and test updates.

## Alternatives Rejected

### Fully autonomous supervisor agent

Rejected because a model-controlled supervisor could alter sequencing, tools, and side effects in ways that are difficult to reproduce and govern.

### Single general-purpose agent

Rejected because tool scope, evaluation, ownership, and failure isolation would be too broad for production remediation.

### Deterministic rules only

Rejected because correlation, unstructured knowledge retrieval, hypothesis generation, and context synthesis benefit from model reasoning when bounded by evidence and evaluation.
