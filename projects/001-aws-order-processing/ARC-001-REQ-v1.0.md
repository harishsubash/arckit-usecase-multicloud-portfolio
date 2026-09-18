# Requirements — Order Processing Platform

| Field | Value |
|---|---|
| Document ID | `ARC-001-REQ-v1.0` |
| Status | Live |
| Version | 1.0 |
| Date | 2026-09-17 |
| Author | ArcKit AI Assistant (reviewed by Harish Subash) |

## Business Drivers

- **BD-01:** The current monolithic order module cannot handle Black Friday peak load (last year: 3 outages, 41 minutes of checkout downtime, estimated £180K lost GMV).
- **BD-02:** Order-to-warehouse handoff currently takes up to 25 minutes via a nightly batch job, delaying same-day dispatch cutoffs.
- **BD-03:** PCI-DSS re-certification flagged the monolith's direct card-data handling as a scope-reduction opportunity.
- **BD-04:** Customer Support cannot see a single, ordered timeline of what happened to an order — they query three separate systems.

## Business Requirements

| ID | Requirement | Driver | Priority |
|---|---|---|---|
| BR-001 | The platform shall accept and durably record a customer order within 1 second of checkout submission, independent of downstream processing speed. | BD-01 | Must |
| BR-002 | The platform shall pass order data to the warehouse management system within 2 minutes of payment confirmation. | BD-02 | Must |
| BR-003 | The platform shall never store raw card-holder data; all payment data shall be tokenised by a PCI-DSS Level 1 provider. | BD-03 | Must |
| BR-004 | The platform shall provide a single, chronologically ordered event history per order, queryable by Customer Support. | BD-04 | Must |
| BR-005 | The platform shall support promotional flash-sale traffic bursts of up to 10x baseline without manual intervention. | BD-01 | Should |

## Functional Requirements

| ID | Requirement | MoSCoW | Traces to |
|---|---|---|---|
| FR-001 | The system shall expose a `POST /orders` API accepting a cart, delivery address, and payment token. | Must | BR-001 |
| FR-002 | The system shall validate stock availability synchronously before accepting an order. | Must | BR-001 |
| FR-003 | The system shall publish an `order.placed` domain event to EventBridge immediately after durable persistence. | Must | BR-001, BR-004 |
| FR-004 | The system shall capture payment asynchronously via the payment gateway and publish `order.paid` or `order.payment_failed`. | Must | BR-001, BR-003 |
| FR-005 | The system shall orchestrate the fulfillment sequence (reserve stock → pick → pack → dispatch) via a Step Functions state machine. | Must | BR-002 |
| FR-006 | The system shall notify the customer by email at `order.placed`, `order.paid`, `order.shipped`, and `order.cancelled` transitions. | Should | — |
| FR-007 | The system shall expose a `GET /orders/{orderId}/timeline` endpoint returning every event for an order, for Customer Support tooling. | Must | BR-004 |
| FR-008 | The system shall archive every domain event to S3 in Parquet format for audit and analytics. | Should | BD-04 |
| FR-009 | The system shall support order cancellation up to the point of warehouse pick confirmation. | Should | — |
| FR-010 | The system shall retry failed carrier-API dispatch calls with exponential backoff up to 5 attempts before routing to a manual-intervention queue. | Must | BD-02 |

## Non-Functional Requirements

| ID | Requirement | Category | MoSCoW | Traces to |
|---|---|---|---|---|
| NFR-001 | The order-acceptance API shall respond within 800ms at p99 under normal load. | Performance | Must | BR-001 |
| NFR-002 | The platform shall scale to 10x baseline throughput within 2 minutes without pre-warming, using Lambda concurrency scaling and SQS buffering. | Scalability | Must | BR-005 |
| NFR-003 | The platform shall guarantee no duplicate payment capture, even under at-least-once message delivery. | Reliability | Must | P3 (idempotency principle) |
| NFR-004 | The order-acceptance path shall maintain 99.95% availability, measured monthly. | Availability | Must | BR-001 |
| NFR-005 | No component shall persist raw card-holder data (PAN, CVV) at rest or in transit within our infrastructure. | Security | Must | BR-003 |
| NFR-006 | Customer PII (name, address, email) shall be encrypted at rest using AWS KMS customer-managed keys. | Security | Must | UK GDPR |
| NFR-007 | Every state-changing request shall be traceable end-to-end via a single correlation ID (X-Ray). | Observability | Must | P6 |
| NFR-008 | Infrastructure cost shall scale sub-linearly with order volume above 2x baseline (unit cost per order decreases at scale). | Cost | Should | BD-01 |
| NFR-009 | All infrastructure shall be deployable to a new AWS account from IaC templates in under 30 minutes (disaster-recovery / environment parity). | Operability | Should | P8 |

## Out of Scope (v1.0)

- Returns and refunds processing (planned for a follow-on project, v2).
- International/cross-border tax calculation (current scope is UK-only).
- Real-time inventory forecasting/replenishment (owned by a separate demand-planning system).
