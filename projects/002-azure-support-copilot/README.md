# Customer Support Copilot (Azure)

An architecture governance project using the [ArcKit](https://arckit.org/) documentation format to design an **AI-powered customer-support copilot** on Microsoft Azure, for a fictional mid-size SaaS company ("Meridian Workspace") used purely to make the artifacts concrete and illustrative.

## Background

Support ticket volume has grown faster than headcount, and first-response time has crept upward. This project designs a retrieval-augmented generation (RAG) copilot — built on Azure OpenAI Service and Azure AI Search — that drafts grounded, citation-backed answers for support agents to review, edit, and send, rather than replacing agents outright.

## Scope

- AI-assisted drafting of support responses, grounded in the existing knowledge base.
- Confidence-based routing between "agent reviews a draft" and "ticket goes straight to the human queue".
- Full audit trail of every AI-generated response for compliance and quality review.
- Explicitly **out of scope** for v1: fully autonomous ticket resolution, voice support, proactive outbound messaging.

## Artifacts

| Artifact | File | Description |
|---|---|---|
| Portfolio Principles | [`../000-global/ARC-000-PRIN-v1.0.md`](../000-global/ARC-000-PRIN-v1.0.md) | Repository-wide principles applying to every project in this portfolio |
| Project Principles | [`ARC-002-PRIN-v1.0.md`](ARC-002-PRIN-v1.0.md) | Azure-specific principles for this project |
| Stakeholder Analysis | [`ARC-002-STKE-v1.0.md`](ARC-002-STKE-v1.0.md) | Who is affected by, or can influence, the copilot programme |
| Requirements | [`ARC-002-REQ-v1.0.md`](ARC-002-REQ-v1.0.md) | Business, functional, and non-functional requirements |
| Risk Register | [`ARC-002-RISK-v1.0.md`](ARC-002-RISK-v1.0.md) | 10 identified risks with likelihood/impact scoring and mitigations |
| Business Case | [`ARC-002-SOBC-v1.0.md`](ARC-002-SOBC-v1.0.md) | Strategic, economic, commercial, financial, and management cases |
| Architecture Strategy | [`ARC-002-STRAT-v1.0.md`](ARC-002-STRAT-v1.0.md) | Target state, phased rollout roadmap, and guardrails |
| Platform Design | [`ARC-002-PLAT-v1.0.md`](ARC-002-PLAT-v1.0.md) | Full Azure architecture and request-sequence diagrams |
| ADR-001 | [`decisions/ARC-002-ADR-001-v1.0.md`](decisions/ARC-002-ADR-001-v1.0.md) | Why Azure AI Search over a self-hosted vector database |
| ADR-002 | [`decisions/ARC-002-ADR-002-v1.0.md`](decisions/ARC-002-ADR-002-v1.0.md) | Why confidence-threshold agent handoff over full AI autonomy |

## Key Azure Services

Azure OpenAI Service · Azure AI Search · Azure Functions · Azure Blob Storage · Azure Cosmos DB · Azure API Management · Microsoft Entra ID · Azure Application Insights

---

Browse the published documentation dashboard for the full, interactive version of these artifacts.
