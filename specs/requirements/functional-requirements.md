# Functional Requirements

## Telemetry and Events

- `FR-TEL-001` The system MUST ingest events from supported enterprise systems through batch, streaming, webhook, API, or managed connector patterns.
- `FR-TEL-002` Raw source payloads MUST be stored immutably with source, tenant, region, ingestion time, event time, schema version, and integrity metadata.
- `FR-TEL-003` The system MUST normalize events into canonical asset, service, identity, change, deployment, alert, finding, incident, and action models.
- `FR-TEL-004` The system MUST identify late, duplicate, malformed, missing, and out-of-order events.
- `FR-TEL-005` Producers and consumers MUST support versioned event contracts and defined compatibility rules.
- `FR-TEL-006` Security findings from Lakewatch MUST retain their source identifiers, severity, status, evidence, and permitted response links.

## Correlation

- `FR-COR-001` The system MUST group potentially related signals using time, topology, identity, dependency, change, similarity, and learned historical relationships.
- `FR-COR-002` Correlation output MUST include membership reasons and per-signal confidence.
- `FR-COR-003` Operators MUST be able to merge, split, suppress, and reopen situations.
- `FR-COR-004` Operator corrections MUST be retained as evaluation labels without automatically changing production behavior.
- `FR-COR-005` Correlation MUST preserve the original source alert and finding identifiers.

## Context Enrichment

- `FR-CTX-001` The system MUST enrich situations with service, owner, environment, criticality, dependency, runbook, change, deployment, incident, and maintenance-window context.
- `FR-CTX-002` Every enrichment field MUST include source, retrieval time, and freshness.
- `FR-CTX-003` Conflicting values MUST be retained and surfaced according to source-precedence policy.
- `FR-CTX-004` Sensitive context MUST be filtered according to user and agent identity.

## Root-Cause Analysis

- `FR-RCA-001` The RCA agent MUST produce ranked hypotheses, not an unsupported single conclusion.
- `FR-RCA-002` Each hypothesis MUST include supporting evidence, contradicting evidence, confidence, assumptions, and tests that could confirm or reject it.
- `FR-RCA-003` The system MUST distinguish correlation, probable cause, contributing factors, and confirmed cause.
- `FR-RCA-004` The system MUST support an inconclusive outcome and escalation.
- `FR-RCA-005` Operator acceptance, rejection, and edits MUST be recorded.

## Remediation Planning

- `FR-PLN-001` Plans MUST specify goals, ordered actions, preconditions, expected effects, risks, blast radius, verification, rollback, timeout, and required approval.
- `FR-PLN-002` Plans MUST be generated only from registered skills and actions.
- `FR-PLN-003` Plans MUST identify whether an action is read-only, reversible, conditionally reversible, or irreversible.
- `FR-PLN-004` Plans MUST be validated against current service state immediately before execution.
- `FR-PLN-005` The system MUST detect conflicting concurrent changes and invalidate stale plans.

## Approval and Execution

- `FR-ACT-001` Mutating actions MUST pass a deterministic policy decision.
- `FR-ACT-002` Approval MUST bind approver, incident, plan version, action set, target scope, expiry, and policy version.
- `FR-ACT-003` Execution MUST use short-lived, least-privilege credentials.
- `FR-ACT-004` Every action MUST use an idempotency key where the target supports idempotency; otherwise the gateway MUST implement duplicate protection.
- `FR-ACT-005` The action gateway MUST record request, response, timestamps, target, actor, credential identity, and result.
- `FR-ACT-006` Partial plan failure MUST trigger safe-stop, rollback, or escalation according to the approved plan.
- `FR-ACT-007` Emergency stop MUST prevent new mutating calls and interrupt safely interruptible workflows.

## Verification and Closure

- `FR-VER-001` Verification MUST use health evidence independent of the action API response.
- `FR-VER-002` Verification MUST compare pre-action baseline, expected result, actual result, and unintended effects.
- `FR-VER-003` Automatic closure MUST require the configured stabilization period and no active rollback condition.
- `FR-VER-004` Closed incidents MUST store resolution, cause disposition, actions, outcome, residual risk, and follow-up work.
- `FR-VER-005` Failed or inconclusive verification MUST escalate or roll back according to policy.

## User Experience

- `FR-UX-001` The front end MUST show a chronological incident timeline.
- `FR-UX-002` The front end MUST distinguish observed facts, retrieved context, model inferences, operator input, and executed actions.
- `FR-UX-003` Users MUST be able to inspect evidence behind every hypothesis and recommendation.
- `FR-UX-004` Approval controls MUST show exact targets, permissions, risk, expected impact, rollback, and expiry.
- `FR-UX-005` Live progress SHOULD use server-sent events with resumable event identifiers.
- `FR-UX-006` The experience MUST degrade to polling when live streaming is unavailable.

## Knowledge and Feedback

- `FR-KNW-001` The system MUST retrieve only approved, access-controlled knowledge sources.
- `FR-KNW-002` Retrieved passages MUST retain document, version, section, timestamp, and access-policy metadata.
- `FR-KNW-003` Completed incidents MAY produce draft knowledge artifacts but MUST NOT publish them without the configured review.
- `FR-KNW-004` Feedback MUST be linked to agent, prompt, model, tool, policy, and dataset versions.

## Administration

- `FR-ADM-001` Administrators MUST be able to register, version, enable, disable, and deprecate agents, skills, tools, schemas, and workflows.
- `FR-ADM-002` Promotion to production MUST require evaluation evidence and approval.
- `FR-ADM-003` Administrators MUST be able to set autonomy by service, environment, action type, risk tier, and time window.
- `FR-ADM-004` The system MUST support replay of historical incidents without invoking production actions.
