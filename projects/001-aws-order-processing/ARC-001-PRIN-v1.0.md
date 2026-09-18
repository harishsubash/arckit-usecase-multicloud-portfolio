# Architecture Principles (Project-Specific) — Order Processing Platform

| Field | Value |
|---|---|
| Document ID | `ARC-001-PRIN-v1.0` |
| Status | Live |
| Version | 1.0 |
| Date | 2026-09-17 |
| Author | ArcKit AI Assistant (reviewed by Harish Subash) |

## Purpose

These principles govern all architecture and design decisions for the Order Processing Platform programme. They apply globally across every project in this repository and take precedence over local convenience unless formally superseded via an Architecture Decision Record (ADR).

## Principles

### P1 — Prefer managed and serverless services over self-managed compute
**Statement:** Default to AWS managed and serverless services (Lambda, API Gateway, SQS, DynamoDB, Step Functions, EventBridge) instead of self-managed EC2/containers, unless a documented requirement cannot be met any other way.
**Rationale:** The team is small relative to peak-season traffic swings; operating patched fleets and capacity planning for Black Friday-scale bursts is not a differentiating activity.
**Implications:** No new EC2 Auto Scaling Groups without an ADR justifying the exception. Operational runbooks focus on service configuration, not host management.

### P2 — Events are the integration contract between bounded contexts
**Statement:** Cross-domain communication (Ordering, Payments, Fulfillment, Notifications) happens via well-defined domain events on EventBridge, never via direct database access or synchronous point-to-point calls between domains.
**Rationale:** Decouples release cycles, allows independent scaling, and gives every consumer a replayable audit trail of what happened.
**Implications:** Every domain event has a versioned schema in the EventBridge schema registry. New consumers subscribe to existing events rather than requesting new synchronous endpoints.

### P3 — Idempotency by default
**Statement:** Every state-changing operation (API request, queue message, Step Functions task) must be safe to retry and safe to receive more than once.
**Rationale:** At-least-once delivery is the norm across SQS, EventBridge, and Lambda retries; designing for exactly-once is unrealistic and idempotency is the practical alternative.
**Implications:** All write paths use a deterministic idempotency key (e.g. `orderId` + `eventType`) and conditional writes in DynamoDB (`ConditionExpression`) to reject duplicates.

### P4 — Pay for what you use, not what you provision
**Statement:** Compute and throughput costs should scale with actual order volume. Avoid reserved capacity and fixed-size clusters unless a cost model proves it cheaper at sustained, predictable load.
**Rationale:** Order volume is seasonal (5-8x spike around Black Friday / Cyber Monday) and unpredictable during flash sales; fixed capacity is either wasted or insufficient.
**Implications:** DynamoDB on-demand capacity mode by default. Lambda concurrency is metered, not pre-warmed, except for a small provisioned-concurrency buffer on the checkout-critical path (see `ARC-001-PLAT-v1.0`).

### P5 — Security and compliance scope is minimised by design
**Statement:** Reduce the surface area that touches card-holder data and personal data to the smallest possible set of components; prefer tokenisation and delegation to certified third parties over handling regulated data directly.
**Rationale:** PCI-DSS and UK GDPR compliance cost is proportional to the number of systems in scope. A hosted payment page and tokenised references keep most of the platform out of PCI scope entirely.
**Implications:** Card data never enters our Lambda functions or DynamoDB tables — only payment-provider tokens and transaction references are stored.

### P6 — Observability is built in, not bolted on
**Statement:** Every Lambda function, state machine, and queue emits structured logs, metrics, and trace IDs from day one; no artifact ships without a corresponding CloudWatch dashboard and alarm set.
**Rationale:** Distributed, event-driven systems are hard to debug after the fact; tracing an order across eight services requires correlation IDs designed in from the start.
**Implications:** All events carry a `correlationId` (the `orderId`). AWS X-Ray tracing is enabled end-to-end. Every Lambda has a paired CloudWatch alarm on error rate and iterator/queue age.

### P7 — Failure is expected and designed for
**Statement:** Every integration point must define its failure and retry behaviour explicitly: dead-letter queues, exponential backoff, circuit breakers, and a documented "what happens if this fails" answer.
**Rationale:** Downstream dependencies (payment gateway, carrier APIs, warehouse systems) will fail or slow down; the platform's resilience is what customers actually experience during those failures.
**Implications:** Every SQS queue has a DLQ with alerting. Step Functions catch blocks are mandatory, not optional, on every task state.

### P8 — Infrastructure is defined as code and reviewed like application code
**Statement:** All AWS resources are provisioned through infrastructure-as-code (AWS SAM / CDK), version-controlled, and changed only through pull requests with peer review.
**Rationale:** Click-ops changes are invisible to the rest of the team and cannot be reliably reproduced across environments (dev/staging/prod).
**Implications:** No manual console changes to production resources. Every environment is a deployment of the same IaC templates with environment-specific parameters.
