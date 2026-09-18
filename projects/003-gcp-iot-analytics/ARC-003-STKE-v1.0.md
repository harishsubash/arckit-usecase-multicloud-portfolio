# Stakeholder Analysis — Fleet IoT Analytics Platform

| Field | Value |
|---|---|
| Document ID | `ARC-003-STKE-v1.0` |
| Status | Live |
| Version | 1.0 |
| Date | 2026-09-17 |
| Author | ArcKit AI Assistant (reviewed by Harish Subash) |

## Context

Northlane Logistics (fictional, illustrative) operates a 1,200-vehicle delivery fleet across six regional depots. Vehicles currently upload GPS/fuel/engine CSV logs once nightly via a depot Wi-Fi sync; fleet operations effectively work with yesterday's data. This project replaces that batch upload with real-time streaming telemetry on Google Cloud.

## Stakeholder Map

| Stakeholder | Role | Interest | Influence | Engagement Approach |
|---|---|---|---|---|
| Head of Fleet Operations | Business sponsor; owns on-time delivery and vehicle utilisation KPIs | High — wants real-time visibility into vehicle location, idle time, and breakdowns | High | Fortnightly steering review; sign-off on rollout phasing and success metrics |
| Data & Analytics Team | Builds and operates the pipeline and dashboards | High — owns delivery of the platform | High | Embedded in delivery; daily standups; owns Platform Design and ADRs |
| Driver Safety & Compliance | Uses telemetry for harsh-braking/speeding coaching and incident investigation | High — needs reliable, auditable event data | Medium | Consulted on retention policy and alert thresholds; reviewer on DPIA-equivalent risk items |
| Finance / Fuel-Cost Owner | Tracks fuel spend and route efficiency savings | Medium — cares about ROI of the programme | Medium | Included in the Business Case economic case; quarterly savings review |
| IT / Platform Engineering | Owns GCP landing zone, IAM, networking, cost governance | High — must approve architecture before production | High | Design review sign-off on Platform Design and cost controls (Principle P5) |
| Depot Managers (6 regions) | Day-to-day consumers of Looker Studio dashboards | Medium — want simple, actionable views, not raw data | Low–Medium | Included in phased regional rollout; UAT feedback loop per depot |
| Drivers (indirect) | Subjects of location and driving-behaviour telemetry | High personal interest, low formal influence | Low (but a veto risk if trust breaks down) | Transparent communication on what is tracked and why; union/works-council consultation before rollout; retention limits per Principle P6 |
| Vehicle Maintenance Team | Consumers of predictive-maintenance alerts | Medium — wants earlier warning of failures than scheduled servicing | Low–Medium | Included in Vertex AI alert design; feedback loop on false-positive rate |
| Data Protection / Legal (advisory) | Ensures location-data handling meets obligations | High for risk sign-off | Medium | Reviewer of retention policy and access controls before go-live |

## Engagement Principles

- **Transparency with drivers first.** Because location tracking directly affects an identifiable workforce, communication to drivers and, where applicable, worker representatives happens *before* the first depot goes live, not after.
- **Depot-by-depot rollout gives every regional manager a low-risk on-ramp** and a chance to raise operational concerns before the platform is fleet-wide (see `ARC-003-STRAT-v1.0`).
- **Finance is engaged on realised savings, not just projected ones** — the economic case in `ARC-003-SOBC-v1.0` is revisited after each rollout phase with actuals.

## RACI Summary (key decisions)

| Decision | Head of Fleet Ops | Data & Analytics | IT Platform Eng | Safety & Compliance |
|---|---|---|---|---|
| Approve target architecture | A | R | C | I |
| Approve driver data retention policy | C | R | I | A |
| Approve rollout phasing | A | R | I | C |
| Approve production cost ceiling | C | R | A | I |

*R = Responsible, A = Accountable, C = Consulted, I = Informed*
