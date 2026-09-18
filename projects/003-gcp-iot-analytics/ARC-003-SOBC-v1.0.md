# Business Case — Fleet IoT Analytics Platform

| Field | Value |
|---|---|
| Document ID | `ARC-003-SOBC-v1.0` |
| Status | Live |
| Version | 1.0 |
| Date | 2026-09-17 |
| Author | ArcKit AI Assistant (reviewed by Harish Subash) |

> This is an illustrative business case for a fictional demonstration company ("Northlane Logistics") and a fictional 1,200-vehicle fleet. Figures are indicative, order-of-magnitude estimates for demonstration purposes, not real financial commitments.

## Strategic Case

Northlane Logistics currently learns about vehicle location, breakdowns, and route deviations up to 24 hours after the fact, via a nightly CSV upload from each depot. This has three consequences: geofence and safety breaches are discovered too late to act on; breakdowns become emergency roadside recoveries instead of scheduled maintenance; and fuel/route inefficiencies are diagnosed retrospectively rather than corrected in-day. Real-time telemetry directly supports the company's stated 2026 strategic priority of "same-day operational visibility across the fleet."

## Economic Case — Options Considered

| Option | Description | Pros | Cons |
|---|---|---|---|
| **A — Status quo** | Continue nightly CSV batch upload from depot Wi-Fi. | No new investment; team already familiar with it. | Does not meet BR-001/BR-002; breakdowns and geofence breaches found too late; fuel inefficiency undiagnosed for weeks. |
| **B — Self-managed streaming (Kafka + Spark/Flink on GKE)** | Run open-source streaming stack on Kubernetes, operated in-house. | Full control; no vendor lock-in on the streaming layer. | Requires 24/7 operational capability the current 4-person data team does not have (Principle P4); higher time-to-value; ongoing patching/scaling burden. |
| **C — Managed GCP streaming stack (recommended)** | Pub/Sub → Dataflow → BigQuery/Bigtable → Looker Studio, as described in `ARC-003-PLAT-v1.0`. | Meets latency and reliability requirements; scales automatically; team can operate it without becoming distributed-systems specialists; fastest time to first depot live. | Ongoing usage-based cloud spend; some GCP-specific skills investment. |

**Preferred option: C.** It is the only option that meets the Must-have requirements (BR-001, BR-002, NFR-001–NFR-004) within a realistic operating model for the existing team size, per Principle P4.

## Commercial Case

No third-party procurement is required beyond standard GCP consumption (existing Google Cloud enterprise agreement) and the existing telemetry hardware already fitted to vehicles, which already produces GPS/fuel/engine data — this project changes *how* that data is transported and processed, not the in-vehicle hardware.

## Financial Case (indicative, order-of-magnitude)

| Item | Estimate (monthly, at full 1,200-vehicle fleet) |
|---|---|
| Pub/Sub ingestion (≈ 1,200 vehicles × 1 msg/10s) | Low hundreds of £/month at this message volume |
| Dataflow streaming jobs (autoscaled, per Principle P5 ceiling) | Low thousands of £/month, primary variable cost |
| BigQuery storage + query (partitioned/clustered per Principle P5) | Low hundreds of £/month storage; query cost controlled via reservations |
| Bigtable (current-state serving) | Low hundreds of £/month, small fixed-size table |
| Looker Studio + Cloud Functions (alerting) | Minimal — largely within free/low tiers at this scale |
| **Indicative total** | **Low-to-mid single-digit thousands of £/month at full fleet scale** |

**Benefit case (indicative):**
- Reduced emergency roadside recoveries via predictive maintenance (FR-007): each emergency recovery avoided is materially more expensive than a scheduled service.
- Fuel efficiency improvement from route-deviation visibility (BR-004): even a low single-digit percentage reduction in fuel spend across a 1,200-vehicle fleet materially outweighs platform cost.
- Faster safety-incident response from real-time geofence/anomaly alerting (BR-002), reducing both risk exposure and manual monitoring effort.

The financial case will be revisited with real depot cost/benefit data after Phase 1 (see `ARC-003-STRAT-v1.0`), rather than relying solely on these pre-pilot estimates.

## Management Case

| Phase | Scope | Gate |
|---|---|---|
| Phase 0 — Foundation | GCP landing zone, IAM, one pilot depot's telemetry pipeline end-to-end | Platform Design and ADRs signed off by IT Platform Engineering |
| Phase 1 — Pilot depot live | Full streaming pipeline + dashboards + geofence alerting for 1 depot (~150 vehicles) | Pilot review against NFR-001/NFR-002; driver communication completed |
| Phase 2 — Regional rollout | Extend to remaining depots in two waves, prioritising connectivity-ready sites | Cost actuals within NFR-005 ceiling at each wave |
| Phase 3 — Predictive maintenance | Enable Vertex AI maintenance alerts in shadow mode, then active | False-positive rate under NFR-007 threshold before going active |

Governance: fortnightly steering review (Head of Fleet Operations, Data & Analytics Team, IT Platform Engineering) per the RACI in `ARC-003-STKE-v1.0`.
