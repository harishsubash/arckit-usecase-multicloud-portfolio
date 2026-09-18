# Risk Register — Fleet IoT Analytics Platform

| Field | Value |
|---|---|
| Document ID | `ARC-003-RISK-v1.0` |
| Status | Live |
| Version | 1.0 |
| Date | 2026-09-17 |
| Author | ArcKit AI Assistant (reviewed by Harish Subash) |

Scoring: Likelihood (1–5) × Impact (1–5) = Score. Score ≥15 High, 8–14 Medium, ≤7 Low.

| ID | Risk | Likelihood | Impact | Score | Mitigation | Owner |
|---|---|---|---|---|---|---|
| R-001 | Pub/Sub message loss or duplication during device connectivity flaps leads to gaps or double-counted telemetry. | 3 | 3 | 9 (Medium) | Use Pub/Sub's at-least-once delivery with idempotent message IDs; Dataflow deduplication on device+sequence-number key; monitor subscription backlog and unacked-message age. | Data & Analytics Team |
| R-002 | Dataflow job falls behind during shift-change spikes, causing dashboard staleness and late alerts. | 3 | 4 | 12 (Medium) | Autoscaling with a tested max-worker ceiling; load-test against 3x peak; alert on system lag exceeding 2 minutes; horizontal partitioning by depot region. | Data & Analytics Team |
| R-003 | Unpartitioned or unclustered BigQuery queries from ad-hoc analysis cause a runaway cost spike. | 3 | 4 | 12 (Medium) | Partition all telemetry tables by ingestion date, cluster by vehicle ID; enforce query byte-scan limits via BigQuery reservation/quota; monthly cost review against NFR-005. | IT Platform Engineering |
| R-004 | Driver location data is retained, accessed, or repurposed beyond what was disclosed, creating a privacy/trust or regulatory issue. | 2 | 5 | 10 (Medium) | Enforce automated retention/purge per FR-009; role-based access with audit logging (FR-010); driver communication and, where applicable, worker-representative consultation before each depot goes live. | Data Protection / Legal (advisory), Head of Fleet Operations |
| R-005 | GPS signal loss in tunnels, underground depots, or dense urban canyons produces false "vehicle stationary" or false geofence-breach alerts. | 4 | 2 | 8 (Medium) | Dead-reckoning smoothing using last-known-heading and speed; suppress geofence alerts during confirmed signal-loss windows; tune thresholds per depot's known blackspots during pilot. | Data & Analytics Team |
| R-006 | Predictive-maintenance model produces excessive false positives, causing alert fatigue and staff ignoring genuine warnings. | 3 | 3 | 9 (Medium) | Start with a conservative threshold in shadow mode (alerts logged, not acted on) for the first depot; tune against Maintenance Team feedback before enabling active alerts fleet-wide. | Vehicle Maintenance Team, Data & Analytics Team |
| R-007 | Vendor/API change or deprecation in a managed GCP service used mid-pipeline causes an unplanned rework. | 2 | 3 | 6 (Low) | Prefer GA (generally available) GCP services over preview features for the core path; track GCP release notes for services in use; pin Dataflow template versions. | IT Platform Engineering |
| R-008 | Depot Wi-Fi/cellular backhaul is insufficient for real-time uplink at some rural depots, undermining the latency requirement (NFR-001) for that region. | 3 | 3 | 9 (Medium) | Assess connectivity per depot before inclusion in a rollout wave; buffer-and-forward on the device for short outages (NFR-003); treat consistently poor-connectivity depots as a later rollout wave with local buffering tuned accordingly. | Head of Fleet Operations |
| R-009 | IAM misconfiguration grants broader access to location/telemetry data than intended. | 2 | 4 | 8 (Medium) | Least-privilege IAM roles reviewed at design time; periodic access review; separate roles for raw-trace access vs aggregated-metric access. | IT Platform Engineering |

## Risk Trend

This is the initial risk register (v1.0), raised at the Platform Design stage. It will be reviewed at the end of each rollout phase (see `ARC-003-STRAT-v1.0`) and re-scored based on pilot-depot evidence — several likelihoods above (notably R-005 and R-006) are expected to fall once real depot data is available for tuning.
