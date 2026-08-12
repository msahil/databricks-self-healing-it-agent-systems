# Integration Contracts

## Integration Principles

- Keep vendor-specific behavior behind connectors and registered skills.
- Prefer events for state changes and APIs for current-state queries.
- Treat all external payloads as untrusted.
- Retain source identifiers and timestamps.
- Use workload identities, short-lived credentials, and scoped permissions.
- Route every production mutation through the action gateway.

## Event Bridge

The event bridge MUST support:

- Authenticated producers.
- Versioned schemas.
- At-least-once receipt with consumer idempotency.
- Dead-letter handling and replay.
- Event-time and ingestion-time timestamps.
- Source signature or integrity status where available.
- Ordering keys for entities requiring ordered processing.
- Back-pressure and rate-limit handling.

Canonical lifecycle events include:

- `telemetry.received`
- `alert.opened|updated|closed`
- `finding.created|updated`
- `change.started|completed|failed`
- `deployment.started|completed|failed|rolled_back`
- `situation.created|updated`
- `incident.state_changed`
- `approval.granted|rejected|expired`
- `action.started|completed|failed`
- `verification.completed`
- `data_quality.degraded|restored`

## ServiceNow

Initial responsibilities:

- Read and update incidents, changes, problems, CMDB records, and approved knowledge.
- Create or link incident and change records.
- Receive approval and state-change events.
- Reconcile updates using source version or update timestamp.

ServiceNow remains authoritative for records it owns. Conflicts must be surfaced; the Databricks platform does not silently overwrite human updates.

## Atlassian

- Jira issues may provide defect, change, and work-tracking context.
- Confluence may provide approved runbooks and architecture knowledge.
- Retrieval must honor source permissions and document lifecycle state.
- Draft or unreviewed content must be clearly labeled and excluded from autonomous action justification unless policy allows it.

## GitLab

- Ingest pipeline, deployment, merge, commit, environment, and rollback metadata.
- Read-only investigation tokens are separate from deployment or rollback identities.
- Mutating operations require action-gateway approval and protected-environment controls.
- Webhook delivery and API reconciliation are both required for critical events.

## Hyperscalers

- Normalize cloud asset, health, configuration, audit, metric, and deployment signals.
- Use cloud-native workload identity where possible.
- Separate accounts, subscriptions, projects, and regions in policy scope.
- Cloud action skills must declare resource patterns and denied operations.
- Provider A2A interfaces, if used, do not bypass action policy or evidence requirements.

## Databricks and Enterprise Data Hub

- Use system tables and supported APIs for Databricks operational evidence.
- Integrate jobs, pipelines, serving endpoints, SQL warehouses, compute, data quality, cost, and audit signals according to available interfaces.
- Treat Unity Catalog as the governance boundary for managed data and AI assets.
- Remediation examples include retrying an idempotent workload, pausing a failing schedule, rolling back an approved deployment, or applying a pre-approved configuration change.

## Lakewatch

- Consume security findings, entities, evidence references, and investigation state through supported interfaces.
- Preserve Lakewatch identifiers and classification.
- Return ticketing, containment, or resolution state only through supported and approved contracts.
- Keep security response authority separate from general IT remediation authority.
- Validate product availability, regional support, and interface details during implementation planning.

## Lumos

Lumos is treated as an identity, access, and entitlement governance source. This role is assumed from the customer architecture and must be confirmed during implementation planning (Residual Open Decision RD-3 in `specs/reviews/red-team-review-001.md`).

- Consume access-request, grant, revocation, and entitlement-change signals as canonical `identity` and `change` events.
- Use entitlement context to enrich incidents (who has access to an affected asset) and to inform authorization decisions, never to bypass them.
- Read-only by default; any Lumos-mediated access change is a mutating action that MUST pass the action gateway and approval policy.
- Preserve Lumos request and grant identifiers and honor source-of-record precedence with enterprise identity.

## Event Authenticity and Trust

- The event bridge MUST authenticate producers and verify source signatures where available.
- Events used as the sole justification for an autonomous or mutating action MUST carry `integrity.signature_status = verified` (`FR-TEL-007`). Unverified or failed-signature events are advisory: they may inform display and human analysis but MUST NOT be the sole basis for a mutating plan.
- Webhook endpoints MUST enforce authentication, signature verification, and replay protection (nonce or bounded timestamp window). A spoofed or replayed source event MUST NOT be able to drive a remediation recommendation on its own.

## Loop Prevention

- Events causally produced by the platform's own actions (via `ACTION --> EVENT`) MUST be origin-tagged.
- Origin-tagged events MUST NOT be re-ingested as independent source signals for correlation or diagnosis (`FR-ACT-009`).
- The orchestrator MUST enforce a per-target action rate limit and a post-action suppression window so that transient post-remediation telemetry does not trigger further automated action.

## Unknown-Result Reconciliation

- Every mutating skill MUST declare a reconciliation probe that determines whether the action took effect (`FR-ACT-008`).
- On an `unknown_result`, the action gateway MUST reconcile actual target state before any retry. No mutating retry may occur until state is confirmed.
- Targets that support neither idempotency keys nor a reconciliation probe MUST be classified non-autonomous and require manual confirmation before any retry.

## MCP and A2A

- MCP servers are treated as tool adapters, not trust boundaries.
- Every MCP tool requires registry metadata, schema validation, identity mapping, logging, and policy classification.
- A2A communication requires authenticated agent identity, task schema, deadline, authority, and trace propagation.
- Remote agents cannot expand their delegated authority or invoke local production tools directly.

## Error Contract

Integration errors use stable classes:

- `authentication_failed`
- `authorization_denied`
- `rate_limited`
- `timeout`
- `unavailable`
- `invalid_request`
- `conflict`
- `stale_state`
- `partial_failure`
- `unknown_result`

Unknown results are not retried as mutations until target state is reconciled.
