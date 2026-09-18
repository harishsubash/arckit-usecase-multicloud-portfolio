# Architecture Principles — Multi-Cloud Portfolio

| Field | Value |
|---|---|
| Document ID | `ARC-000-PRIN-v1.0` |
| Status | Live |
| Version | 1.0 |
| Date | 2026-09-18 |
| Author | ArcKit AI Assistant (reviewed by Harish Subash) |
| Scope | Repository-wide — applies to every project in this portfolio |

## Purpose

This document sets the portfolio-level principles that apply across every project in this repository, regardless of which cloud it runs on. Each individual project (AWS Order Processing, Azure Support Copilot, GCP Fleet IoT Analytics) additionally follows its own cloud-specific design choices, documented in that project's Platform Design and ADRs — those choices must not contradict the principles below.

## Principles

1. **Best-fit cloud, not a single-vendor mandate.** Each workload is placed on the cloud that best fits its dominant technical driver — event-driven serverless economics (AWS), first-party LLM/RAG tooling (Azure), or large-scale streaming analytics (GCP) — rather than forcing every project onto one provider for the sake of uniformity. Vendor choice is documented and justified per project (see each project's Business Case, economic case section).

2. **Managed services over self-hosted infrastructure.** Across all three clouds, prefer the provider's managed/serverless offering over running and patching the equivalent open-source stack ourselves, unless a specific compliance, cost-at-scale, or portability requirement says otherwise. This trades a small amount of long-run unit-cost efficiency for a large reduction in operational burden.

3. **Every workload has a named accountable owner and a risk register.** No project in this portfolio ships without a Stakeholder Analysis, Requirements set, and Risk Register reviewed by that project's accountable business owner. Architecture is not approved in the abstract — it is approved against named stakeholders' actual concerns.

4. **Event-driven and streaming-first where latency or decoupling matters.** All three projects in this portfolio are built around asynchronous events or streams (order events, retrieval-then-generation requests, vehicle telemetry) rather than synchronous point-to-point coupling, because each domain has independent producers and consumers that must scale and fail independently.

5. **Data residency and classification are decided before the platform is designed, not after.** Each project's data — payment/PII data, customer conversation data, or vehicle/driver telemetry — is classified and its residency/retention requirements are agreed with the relevant compliance stakeholder before the Platform Design document is written, not retrofitted afterward.

6. **Cost is an architectural requirement, not a finance afterthought.** Every project's Business Case includes a cost model, and every Platform Design includes a stated cost-control mechanism (budget alarms, token/context caps, partition/cluster design) — cost overruns are treated as a design failure, not just a billing surprise.

7. **Portability is a conscious trade-off, recorded via ADR, not an accident.** Where a project accepts vendor lock-in in exchange for delivery speed or managed-service benefits (e.g. AWS Step Functions, Azure OpenAI, GCP Dataflow), that trade-off is explicitly recorded as an Architecture Decision Record with alternatives considered — so a future team can revisit the decision with full context, rather than being surprised by hidden coupling.

8. **AI-assisted, human-reviewed governance.** Every artifact in this portfolio — including this one — was drafted with AI assistance and reviewed by a named human architect before being marked Live. AI accelerates the first draft; a human remains accountable for the decision.

## Applies To

| Project | Cloud | Primary Driver |
|---|---|---|
| `001-aws-order-processing` | AWS | Event-driven order processing at peak-season scale |
| `002-azure-support-copilot` | Azure | RAG-grounded generative AI with human-in-the-loop |
| `003-gcp-iot-analytics` | Google Cloud | Real-time streaming telemetry analytics |
