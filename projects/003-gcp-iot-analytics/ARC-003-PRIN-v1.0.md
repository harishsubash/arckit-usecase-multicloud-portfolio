# Architecture Principles (Project-Specific)

| Field | Value |
|---|---|
| Document ID | `ARC-003-PRIN-v1.0` |
| Status | Live |
| Version | 1.0 |
| Date | 2026-09-17 |
| Author | ArcKit AI Assistant (reviewed by Harish Subash) |

## Purpose

These principles govern all architecture decisions made across projects in this workspace. They apply globally and are inherited by project-level artifacts unless explicitly overridden with a documented rationale.

## Principles

### P1 — Streaming-first where latency matters
Where a business decision (alerting, dispatch, safety) depends on data being less than a few minutes old, design for continuous streaming ingestion and processing rather than periodic batch jobs. Batch remains acceptable for historical reporting and reconciliation.

**Rationale:** Nightly batch pipelines hide operational problems (a stranded vehicle, a geofence breach, a fuel anomaly) for up to 24 hours. Streaming closes that gap to seconds.

**Implications:** Ingestion paths must be built on a durable pub/sub layer, not file drops; processing jobs must support unbounded, windowed computation.

### P2 — One source of truth, purpose-built serving stores
BigQuery is the single analytical source of truth for all fleet telemetry and derived facts. Low-latency serving paths (e.g. "where is vehicle X right now") use a purpose-built store fed from the same pipeline, never a second independent write path.

**Rationale:** Divergent write paths cause data to drift apart and make debugging discrepancies extremely expensive.

**Implications:** Serving stores (e.g. Bigtable) are always populated as a fan-out from the canonical streaming pipeline, never populated independently.

### P3 — Design for disorder: out-of-order and late-arriving data
Telemetry will arrive late or out of order (connectivity loss, device buffering, retries). Pipelines must use event-time processing with explicit watermarks and a defined lateness policy, not arrival-time assumptions.

**Rationale:** A moving vehicle fleet has intermittent connectivity by nature (tunnels, rural routes, depot basements). Assuming in-order arrival produces silently wrong aggregates.

**Implications:** All windowed aggregations must specify allowed lateness and a strategy (drop, side-output, or late-update) for data that arrives after the watermark.

### P4 — Prefer managed services over self-managed infrastructure
Default to fully managed GCP services (Pub/Sub, Dataflow, BigQuery, Bigtable) over self-operated equivalents (Kafka, Flink/Spark clusters) unless a specific, documented requirement cannot be met by the managed option.

**Rationale:** A small platform team cannot sustainably operate stateful distributed systems 24/7 for a fleet that runs around the clock. Operational risk and toil outweigh the marginal cost or flexibility gains of self-managed infrastructure at this scale.

**Implications:** Any proposal to self-manage a data-plane component requires an ADR justifying why the managed alternative is insufficient.

### P5 — Cost is a first-class design constraint
Every pipeline and query must be designed with its steady-state and peak cost understood and bounded. BigQuery tables are partitioned and clustered by default; streaming jobs have autoscaling ceilings; ad-hoc full-table scans are treated as defects.

**Rationale:** Streaming analytics platforms can silently become far more expensive than the batch systems they replace if written carelessly — an unpartitioned scan or an unbounded autoscaler is a budget incident waiting to happen.

**Implications:** Table design (partitioning/clustering keys) and Dataflow max-worker limits are reviewed at design time, not discovered from a bill.

### P6 — Privacy and minimisation for driver and location data
Vehicle telemetry that can identify or track an individual driver (precise GPS trace, driver ID association) is minimised, access-controlled, and retained only as long as operationally or legally required.

**Rationale:** Continuous location tracking of employees carries real privacy risk and, in many jurisdictions, specific legal obligations. Treating it as "just another metric" is a governance failure.

**Implications:** Raw precise-location data has a defined retention window; aggregated/derived data (e.g. route efficiency) may be retained longer; access to raw traces is role-restricted and logged.

### P7 — Observability is built in, not bolted on
Every pipeline stage exposes health, lag, and error-rate metrics before it is considered production-ready. Silent data loss is treated as a severity-1 defect class.

**Rationale:** A streaming pipeline that "looks fine" from the dashboard but is silently dropping messages upstream is worse than one that visibly fails, because it erodes trust in the data.

**Implications:** Pub/Sub subscription backlog, Dataflow system lag, and BigQuery load-job failures are alertable metrics from day one of production rollout.

## Application

Every project-level Platform Design (`ARC-XXX-PLAT`) and Architecture Decision Record (`ARC-XXX-ADR`) must reference which of these principles it applies, and document any deviation explicitly with a rationale.
