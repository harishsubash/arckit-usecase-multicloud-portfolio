# Order Processing Platform

An architecture governance project in the [ArcKit](https://arckit.org) style, demonstrating a **serverless, event-driven order-processing platform on AWS** for a fictional mid-size online retailer ("Northbridge Goods").

## Background

Northbridge Goods' order-management capability is a monolithic application that struggles under peak-season load and holds card-holder data directly, inflating its PCI-DSS audit scope. This project designs a replacement built on API Gateway, Lambda, SQS, DynamoDB, EventBridge, and Step Functions — scaling automatically from quiet weekdays to Black Friday spikes, with card data never touching our infrastructure.

## Scope

Order acceptance, payment capture, fulfillment orchestration (reserve → pick → pack → dispatch), customer notification, and an order-timeline API for Customer Support. Returns/refunds, international tax, and inventory forecasting are explicitly out of scope for v1.0 — see `ARC-001-REQ-v1.0`.

## Architecture Artifacts

| Artifact | Description |
|---|---|
| [Portfolio Principles](../000-global/ARC-000-PRIN-v1.0.md) | Repository-wide principles applying to every project in this portfolio |
| [Project Principles](ARC-001-PRIN-v1.0.md) | AWS-specific principles for this project |
| [Stakeholder Analysis](ARC-001-STKE-v1.0.md) | Who cares about this platform and how they're engaged |
| [Requirements](ARC-001-REQ-v1.0.md) | Business, functional, and non-functional requirements, traced to business drivers |
| [Risk Register](ARC-001-RISK-v1.0.md) | Ten identified risks, scored and mitigated |
| [Business Case](ARC-001-SOBC-v1.0.md) | Strategic, economic, commercial, financial, and management case for the rebuild |
| [Architecture Strategy](ARC-001-STRAT-v1.0.md) | Target state and the strangler-fig migration approach |
| [Platform Design](ARC-001-PLAT-v1.0.md) | Component architecture, sequence diagrams, and scaling approach |
| [ADR-001: SQS vs Kinesis](decisions/ARC-001-ADR-001-v1.0.md) | Why SQS + Lambda for order-event ingestion |
| [ADR-002: DynamoDB vs RDS](decisions/ARC-001-ADR-002-v1.0.md) | Why single-table DynamoDB for order state |

## Note

This is a demonstration project generated to show what an ArcKit-style architecture governance repository looks like. It is illustrative, not a real client engagement — always review and validate AI-assisted architecture output with human expertise before production use.
