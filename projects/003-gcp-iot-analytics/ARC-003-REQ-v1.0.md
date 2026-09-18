# Requirements — Fleet IoT Analytics Platform

| Field | Value |
|---|---|
| Document ID | `ARC-003-REQ-v1.0` |
| Status | Live |
| Version | 1.0 |
| Date | 2026-09-17 |
| Author | ArcKit AI Assistant (reviewed by Harish Subash) |

## Business Requirements

| ID | Requirement | Priority |
|---|---|---|
| BR-001 | Fleet operators can see vehicle location, status, and key telemetry updated within seconds, not next-day. | Must |
| BR-002 | Operations staff are alerted automatically when a vehicle breaches a defined geofence (e.g. leaves an approved route corridor). | Must |
| BR-003 | Maintenance staff receive predictive-maintenance alerts ahead of a likely component failure, reducing unplanned breakdowns. | Should |
| BR-004 | Finance can report on fuel efficiency and route deviation trends per depot and per vehicle class. | Should |
| BR-005 | Driver-identifiable location data is retained no longer than operationally necessary and is access-controlled. | Must |
| BR-006 | The platform scales to the full 1,200-vehicle fleet without redesign. | Must |

## Functional Requirements

| ID | Requirement | Linked BR | Priority |
|---|---|---|---|
| FR-001 | Vehicle telemetry devices publish GPS, fuel level, and engine diagnostic events to a durable ingestion endpoint at up to 1 message per 10 seconds per vehicle. | BR-001 | Must |
| FR-002 | The platform computes rolling per-vehicle aggregates (avg speed, idle time, fuel consumption rate) over 1-minute and 15-minute tumbling windows. | BR-001, BR-004 | Must |
| FR-003 | The platform evaluates each location update against depot-defined geofence polygons and raises an alert event within 30 seconds of a breach. | BR-002 | Must |
| FR-004 | Geofence and anomaly alerts are delivered to on-call operations staff via a notification channel (e.g. email/SMS/chat webhook). | BR-002 | Must |
| FR-005 | A "current state" view for any vehicle (latest location, status, last-seen timestamp) is queryable with p99 latency under 100ms. | BR-001 | Must |
| FR-006 | Historical telemetry is queryable in the analytical warehouse for trend reporting going back at least 13 months. | BR-004 | Should |
| FR-007 | A predictive-maintenance model scores engine-diagnostic patterns and raises a maintenance-review alert when risk exceeds a configurable threshold. | BR-003 | Should |
| FR-008 | Dashboards are available per-depot, filterable by vehicle, date range, and alert type. | BR-004 | Should |
| FR-009 | Raw precise-location telemetry is automatically purged or anonymised after a configurable retention period (default 90 days); aggregated metrics may be retained longer. | BR-005 | Must |
| FR-010 | Access to raw location traces is restricted by role and all access is logged. | BR-005 | Must |

## Non-Functional Requirements

| ID | Requirement | Category | Priority |
|---|---|---|---|
| NFR-001 | End-to-end latency from telemetry publish to dashboard-visible update: p95 < 60 seconds. | Performance | Must |
| NFR-002 | Ingestion durability: no more than 0.01% of published telemetry messages lost under normal operation (target 99.99% durability, aligned to Pub/Sub's design). | Reliability | Must |
| NFR-003 | The platform tolerates loss of connectivity for an individual vehicle for up to 30 minutes (e.g. tunnel, rural dead zone) without data loss, buffering and forwarding on reconnect. | Reliability | Must |
| NFR-004 | Streaming compute scales automatically to handle at least 3x normal peak load (e.g. depot shift-change spikes) without manual intervention. | Scalability | Must |
| NFR-005 | Monthly platform cost (ingestion + processing + storage + serving) stays within an agreed budget ceiling, reviewed against actuals monthly. | Cost | Must |
| NFR-006 | All data in transit and at rest is encrypted using GCP default or stronger encryption; access is via IAM roles following least privilege. | Security | Must |
| NFR-007 | Geofence and anomaly alert false-positive rate is kept under 5% after the first full month of tuning per depot. | Quality | Should |
| NFR-008 | Pipeline health (subscription backlog, processing lag, job failures) is observable via dashboards and alertable within 5 minutes of an incident. | Observability | Must |
| NFR-009 | The platform supports staged rollout by depot without requiring a full-fleet cutover. | Operability | Must |

## MoSCoW Summary

- **Must (12):** BR-001, BR-002, BR-005, BR-006, FR-001–FR-005, FR-009, FR-010, NFR-001–NFR-006, NFR-008, NFR-009
- **Should (6):** BR-003, BR-004, FR-006–FR-008, NFR-007
- **Could / Won't (this phase):** Third-party insurance telematics integration; driver mobile app — explicitly out of scope for v1, noted for a future phase.
