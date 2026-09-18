# Architecture Strategy — Fleet IoT Analytics Platform

| Field | Value |
|---|---|
| Document ID | `ARC-003-STRAT-v1.0` |
| Status | Live |
| Version | 1.0 |
| Date | 2026-09-17 |
| Author | ArcKit AI Assistant (reviewed by Harish Subash) |

## Target State

The target state is a single, managed, streaming-first telemetry platform on Google Cloud, replacing per-depot nightly CSV batch uploads with continuous ingestion. All 1,200 vehicles publish telemetry to a shared Pub/Sub topic; a single Dataflow pipeline processes all depots' data, partitioning downstream by depot/region for reporting; BigQuery is the analytical system of record; Bigtable serves live "current vehicle state" queries; Looker Studio is the standard dashboarding layer for depot managers and fleet operations.

## Migration Approach

The migration is **strangler-style, run in parallel with the existing batch process**, not a big-bang cutover:

1. **Dual-run period.** The new streaming pipeline is stood up and validated against the pilot depot while the existing nightly CSV batch continues unchanged. Streaming output is cross-checked against the batch feed for the same period to build confidence before batch is retired for that depot.
2. **Retire batch depot-by-depot.** Once a depot's streaming pipeline has run reliably for an agreed validation window (meeting NFR-001/NFR-002) and depot managers have adopted the new dashboards, that depot's nightly batch upload is switched off.
3. **No historical backfill required.** Existing historical CSV data remains queryable in its current location for trend continuity; the new BigQuery tables become the source of truth going forward from each depot's cutover date.

## Rollout Phasing by Depot/Region

| Wave | Depots | Rationale |
|---|---|---|
| Pilot | 1 depot with strong cellular/Wi-Fi backhaul (~150 vehicles) | Lowest connectivity risk (R-008); fastest path to validate latency and reliability requirements |
| Wave 1 | 2 depots with good connectivity | Builds operational confidence and tunes alert thresholds (R-005, R-006) before wider rollout |
| Wave 2 | Remaining 3 depots, including rural/lower-connectivity sites | Benefits from buffer-and-forward tuning (NFR-003) proven in earlier waves |

## Guardrails

- **No wave proceeds until the previous wave's risk register review (`ARC-003-RISK-v1.0`) shows no open High-severity items.**
- **Cost ceiling (NFR-005) is checked after every wave**, not just at the end — Dataflow autoscaling limits and BigQuery reservations are re-tuned per Principle P5 before scaling to the next wave.
- **Driver communication and, where applicable, worker-representative consultation completes before a depot's vehicles start streaming location data** — this is a hard gate, not a parallel-track activity (Principle P6, `ARC-003-STKE-v1.0`).
- **The old batch pipeline is not decommissioned platform-wide until every depot has cut over** — it remains the fallback for any depot not yet migrated.

## Alignment to Principles

This strategy directly implements Principle P1 (streaming-first), P2 (single source of truth with purpose-built serving), P4 (managed services, enabling a phased rollout without needing to scale an ops team), and P6 (privacy gate before rollout). Deviations, if any arise during a wave, will be documented as ADRs against the relevant principle.
