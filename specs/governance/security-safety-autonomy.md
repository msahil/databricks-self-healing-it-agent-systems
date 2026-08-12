# Security, Safety, and Autonomy

## Trust Boundaries

The following are separate trust boundaries:

- End user and browser.
- Front-end backend.
- Orchestrator and agents.
- Retrieval stores and external documents.
- Action gateway and target systems.
- Development, test, staging, and production.
- IT operations and security operations.
- Tenants, business units, regions, and regulated data domains.

## Identity Model

- Users authenticate through enterprise identity.
- Services and agents use dedicated workload identities.
- Read-only investigation and mutating execution use separate identities.
- Approval captures the human identity and the executing workload identity.
- Delegation is explicit, scoped, short-lived, and non-transitive by default.

## Authorization

Authorization decisions consider:

- Initiating user and role.
- Workload identity.
- Service and resource ownership.
- Environment and region.
- Data classification.
- Action class and risk tier.
- Incident severity and declared impact.
- Maintenance and change windows.
- Current autonomy policy version.
- Approval state and expiry.

## Autonomy Levels

| Level | Capability | Example |
|---|---|---|
| A0 Observe | Ingest and display | Build incident timeline |
| A1 Assist | Analyze and recommend | Rank RCA hypotheses |
| A2 Prepare | Build executable plan | Draft rollback plan |
| A3 Approve-to-run | Execute after human approval | Restart a service |
| A4 Bounded autonomous | Execute pre-approved low-risk actions | Retry idempotent failed job |
| A5 Broad autonomous | Reserved; not an initial target | Multi-system remediation |

Autonomy is assigned per service and action class, not globally. A production service may allow A4 retries while requiring A3 for restart and prohibiting configuration mutation.

## Risk Tiers

| Tier | Description | Default authority |
|---|---|---|
| R0 | Read-only | A1 |
| R1 | Low impact, reversible, narrow scope | A3; A4 after qualification |
| R2 | Moderate impact or conditional rollback | A3 with designated approver |
| R3 | High impact, broad scope, security-sensitive | Multiple approvals or manual execution |
| R4 | Irreversible, destructive, legally sensitive | Prohibited from autonomous execution |

## Mandatory Action Controls

1. Registered action and exact version.
2. Valid incident and non-expired plan.
3. Fresh target-state precondition check.
4. Deterministic policy decision.
5. Valid approval token when required.
6. Least-privilege execution identity.
7. Idempotency or duplicate protection.
8. Scope and rate limit.
9. Full request and result audit.
10. Independent verification and rollback decision.

## Prompt and Tool Security

- External content cannot modify system instructions or authorization.
- Tools are selected from an allowlist supplied by the orchestrator.
- Tool descriptions are versioned and reviewed.
- Inputs and outputs are schema validated and size limited.
- Secret patterns and sensitive values are redacted before model calls and traces.
- Retrieved content is labeled by source and trust level.
- Agents must not follow instructions embedded in logs, tickets, documents, or tool output.

## Emergency Controls

- Global and scoped automation kill switches.
- Per-service and per-action disable controls.
- Credential revocation independent of application deployment.
- Queue draining and safe interruption procedures.
- Manual reconciliation of unknown-result actions.
- Break-glass access with enhanced logging and expiry.

## Segregation of Duties

- Agent authors cannot self-approve production promotion.
- Policy authors cannot approve their own high-risk exception.
- The reasoning component cannot approve or execute its plan.
- Action executors cannot alter audit records.
- Evaluation owners must be able to reproduce promotion evidence.

## Data Governance

- Catalog, schema, table, model, function, volume, and search-index ownership is explicit.
- Sensitive data is minimized before prompt construction.
- Row and column controls are enforced before retrieval.
- Trace retention and access are separately governed because traces may contain operational context.
- Data deletion, legal hold, and investigation hold processes are defined before production.

## Autonomy Promotion Gate

An action class may move to A4 only when:

- It is R0 or R1.
- Target scope is bounded and machine-verifiable.
- Rollback or safe-stop behavior has been tested.
- Historical and shadow evaluations meet approved thresholds.
- No unresolved critical safety finding exists.
- Operations and service owners approve the policy.
- Emergency-stop and reconciliation procedures have been exercised.
- The change has a defined review and automatic expiry date.
