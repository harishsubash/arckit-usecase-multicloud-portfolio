# Architecture Strategy — Order Processing Platform

| Field | Value |
|---|---|
| Document ID | `ARC-001-STRAT-v1.0` |
| Status | Live |
| Version | 1.0 |
| Date | 2026-09-17 |
| Author | ArcKit AI Assistant (reviewed by Harish Subash) |

## Target State Narrative

The target state replaces a single monolithic order-management application with a set of small, independently deployable services connected by domain events on Amazon EventBridge. Order acceptance, payment capture, fulfillment orchestration, and customer notification each become a bounded context with its own Lambda functions and, where it owns state, its own DynamoDB table. No service reaches into another's data store; all cross-context communication is event-based, per Principle P2.

The fulfillment process — historically a nightly batch job — becomes a Step Functions state machine that starts the moment payment is confirmed, with explicit states for stock reservation, warehouse pick request, pack confirmation, and carrier dispatch, each with its own timeout and failure path.

## Migration Approach: Strangler Fig

Rather than a big-bang cutover, the monolith is strangled incrementally:

1. **Shadow mode.** The new order-acceptance API runs in parallel with the monolith. Every checkout request is sent to both; only the monolith's response is returned to the customer. Results are compared offline to catch discrepancies with zero customer risk.
2. **Segment cutover.** Once shadow-mode parity is proven, a small, low-risk customer segment (e.g. newsletter sign-ups checking out via the mobile app) is routed live to the new platform via a feature flag.
3. **Progressive rollout.** Traffic is increased in stages (10% → 25% → 50% → 100%) with a rollback switch at every stage, monitored against the NFRs in `ARC-001-REQ-v1.0`.
4. **Full peak-season proof.** The new platform must handle one complete high-traffic event (a bank-holiday sale) end-to-end before the monolith's order module is scheduled for decommission.
5. **Decommission.** The monolith's order code is retired; its database becomes read-only for historical reporting until archived.

This mirrors the phased timeline in `ARC-001-SOBC-v1.0` and keeps the "do nothing" fallback (Option A) available at every stage until the final cutover.

## Guardrails

- **No shared database.** The new platform never reads from or writes to the monolith's SQL Server database directly; all data exchange during the transition period happens via a one-way event bridge (monolith emits `legacy.order.completed` for orders it still owns).
- **Feature-flag everything customer-facing.** Every rollout stage must be reversible within minutes via a flag flip, not a deployment.
- **No new synchronous coupling.** Any proposal to add a synchronous call between bounded contexts requires an ADR explaining why an event cannot satisfy the requirement (Principle P2).
- **Cost visibility from day one.** AWS Cost and Usage Reports are tagged per bounded context so the financial case in `ARC-001-SOBC-v1.0` can be validated against actuals during rollout, not just at year-end.
- **Security review before segment cutover.** No customer traffic is routed to the new platform until the Security & Compliance Officer has signed off on the PCI-scope reduction described in `ARC-001-PLAT-v1.0`.

## Technology Direction

| Layer | Decision | Reference |
|---|---|---|
| API | Amazon API Gateway (REST) fronting Lambda | `ARC-001-PLAT-v1.0` |
| Compute | AWS Lambda, Node.js runtime, one function per use case | Principle P1 |
| Async ingestion | Amazon SQS (standard queues) with per-queue DLQs | `ARC-001-ADR-001-v1.0` |
| Domain events | Amazon EventBridge, schema-registered | Principle P2 |
| State | Amazon DynamoDB, on-demand capacity, single-table design per bounded context | `ARC-001-ADR-002-v1.0` |
| Orchestration | AWS Step Functions (Standard workflows) for the fulfillment saga | `ARC-001-PLAT-v1.0` |
| Notifications | Amazon SNS + SES for customer email | `ARC-001-PLAT-v1.0` |
| Archive/audit | Amazon S3 (Parquet), lifecycle to Glacier after 90 days | Principle P6 |
| IaC | AWS SAM | Principle P8 |

## Success Measures

The strategy is judged successful when, over one full peak-trading period: checkout availability meets NFR-004 (99.95%), no duplicate-charge incidents occur (NFR-003), warehouse handoff time drops from ~25 minutes to under 2 minutes (BR-002), and the PCI-DSS audit scope is reduced to SAQ-A equivalent.
