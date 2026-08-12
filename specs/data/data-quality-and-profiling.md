# Data Quality Monitoring and Data Profiling

## Purpose

Operational decisions are only as reliable as their telemetry. Unity Catalog data quality monitoring and data profiling are used to measure freshness, completeness, validity, distribution changes, drift, and model-inference dataset quality. Data profiling does not replace event-processing health checks or security monitoring.

## Profile Scope

### Time-series profiles

Use for telemetry and operational tables with event timestamps. Required metrics include volume, delay, missingness, uniqueness, severity distribution, source distribution, and key service dimensions.

### Inference profiles

Use for supported agent input and output datasets where prediction, label, or model metadata is represented in a compatible inference-log pattern. Measure confidence distribution, accepted or rejected results, latency, token use, failure class, and eventual operator or incident outcome.

### Snapshot profiles

Use for slowly changing reference datasets such as service ownership, skill registry, autonomy policy, source precedence, and criticality mappings.

## Required Monitoring Domains

| Domain | Minimum checks |
|---|---|
| Source ingestion | Freshness, volume, schema, duplicates, parse failures |
| Service topology | Orphan assets, cycles, stale ownership, ambiguous mappings |
| Incident data | State validity, missing evidence, invalid transitions |
| Agent outputs | Schema validity, evidence coverage, confidence distribution |
| Actions | Missing approvals, duplicate keys, incomplete outcomes |
| Knowledge | Stale documents, missing owners, access metadata |
| Service health (UC1) | Golden-signal freshness, synthetic success, KPI availability, baseline coverage |
| Monitored agents (UC2) | Trace completeness, evaluation coverage, groundedness and quality drift, cost anomalies |
| Agent security (UC2) | Injection and jailbreak attempt rate, PII-redaction coverage, tool-denial rate, cross-account attempts |

## Custom Metrics

- Percentage of events mapped to a known service.
- Percentage of incidents with at least one immutable evidence item.
- Percentage of hypotheses containing contradicting-evidence analysis.
- Profile of confidence by accepted, rejected, and inconclusive outcome.
- Percentage of action records with completed verification.
- Data freshness relative to the incident-analysis time.
- Source availability by connector and region.
- UC1: synthetic success rate by journey, golden-signal recovery time after an action, and payment-success KPI availability.
- UC2: injection and jailbreak catch rate, unauthorized-action attempts blocked, PII-leakage incidents, groundedness by agent version, and cost per conversation.

## Drift Handling

1. A profile or explicit validation detects drift.
2. The system emits a `data_quality.degraded` event.
3. The orchestrator marks affected evidence sources as degraded.
4. Agents receive the quality status and reduce or withhold conclusions according to policy.
5. Autonomous actions relying on the degraded source are blocked unless an approved fallback exists.
6. Resolution requires data-quality verification, not only connector recovery.

## Baselines

Baselines must reflect comparable service, environment, region, and time windows. Production must not be compared indiscriminately with development, and peak periods must not be compared with ordinary periods without seasonal controls.

## Access and Governance

- Profile and drift metric tables are governed Unity Catalog assets.
- Monitoring outputs inherit or strengthen source classifications.
- Raw sensitive fields should not be copied into custom metric definitions.
- Owners, schedules, retention, and consumers must be recorded.
- Deleting a profile requires an impact check for alerts, dashboards, and agent dependencies.

## Failure Behavior

Monitoring or profiling failure must be visible as an operational incident. Missing metrics must not be interpreted as healthy data. Agents receive an explicit `unknown`, `degraded`, or `unavailable` status.
