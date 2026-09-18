# Fleet IoT Analytics Platform

An architecture governance project, generated in the style of [ArcKit](https://arckit.org), for a fictional, illustrative use case: modernising a 1,200-vehicle delivery fleet from nightly batch CSV telemetry uploads to real-time streaming analytics on Google Cloud.

## Background

Northlane Logistics (fictional, illustrative company) operates six regional depots. Today, fleet operations discover vehicle location, breakdowns, and route deviations up to 24 hours late, via a nightly CSV upload. This project designs a streaming platform — vehicle telemetry published to **Pub/Sub**, processed by **Dataflow**, landed in **BigQuery** (analytics) and **Bigtable** (low-latency serving), visualised in **Looker Studio**, with real-time geofence alerting via **Cloud Functions** and predictive maintenance via **Vertex AI**.

## Scope

In scope: real-time GPS/fuel/engine telemetry ingestion, geofence breach alerting, current-vehicle-state serving, historical analytics dashboards, predictive maintenance scoring, and driver-location privacy/retention controls. Out of scope for this phase: third-party insurance telematics integration and a driver-facing mobile app.

## Artifacts

| Artifact | Description |
|---|---|
| [`ARC-003-PRIN-v1.0.md`](./ARC-003-PRIN-v1.0.md) | Project-specific architecture principles for this GCP project |
| [`ARC-003-STKE-v1.0.md`](./ARC-003-STKE-v1.0.md) | Stakeholder analysis — who is affected and how they're engaged, including driver privacy considerations |
| [`ARC-003-REQ-v1.0.md`](./ARC-003-REQ-v1.0.md) | Functional and non-functional requirements, MoSCoW-prioritised |
| [`ARC-003-RISK-v1.0.md`](./ARC-003-RISK-v1.0.md) | Risk register covering data loss, cost, privacy, and connectivity risks |
| [`ARC-003-SOBC-v1.0.md`](./ARC-003-SOBC-v1.0.md) | Business case — strategic, economic, commercial, financial, and management cases |
| [`ARC-003-STRAT-v1.0.md`](./ARC-003-STRAT-v1.0.md) | Architecture strategy — target state and phased, depot-by-depot migration approach |
| [`ARC-003-PLAT-v1.0.md`](./ARC-003-PLAT-v1.0.md) | Platform design — full architecture with diagrams, component detail, scaling and cost controls |
| [`decisions/ARC-003-ADR-001-v1.0.md`](./decisions/ARC-003-ADR-001-v1.0.md) | ADR: Pub/Sub + Dataflow over self-managed Kafka + Flink |
| [`decisions/ARC-003-ADR-002-v1.0.md`](./decisions/ARC-003-ADR-002-v1.0.md) | ADR: Bigtable for current-vehicle-state serving over querying BigQuery directly |

Portfolio-wide architecture principles applying to every project in this repository are in [`../000-global/ARC-000-PRIN-v1.0.md`](../000-global/ARC-000-PRIN-v1.0.md).

## Disclaimer

This is a demonstration project. The company, fleet, and all figures are fictional and illustrative. Content was AI-generated to demonstrate an ArcKit-style architecture governance documentation set and should not be treated as a real deliverable.
