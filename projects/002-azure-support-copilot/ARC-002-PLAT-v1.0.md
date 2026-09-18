# Platform Design — Customer Support Copilot

| Field | Value |
|---|---|
| Document ID | ARC-002-PLAT-v1.0 |
| Status | Draft |
| Version | 1.0 |
| Date | 2026-09-17 |
| Author | ArcKit AI Assistant (reviewed by Harish Subash) |
| Project | 001-support-copilot |

## Overview

The platform is a retrieval-augmented generation (RAG) system built entirely on managed Azure services. It has two pipelines: an **ingestion pipeline** that keeps the knowledge base searchable, and a **query pipeline** that answers a support ticket at request time.

## Architecture Diagram

```mermaid
flowchart TB
    subgraph Client
        A[Agent Desk UI]
        C[Customer Channel<br/>email / in-app chat]
    end

    subgraph Edge
        APIM[Azure API Management<br/>front door, auth, rate limiting]
    end

    subgraph Orchestration
        FN[Azure Functions<br/>Orchestrator]
        EVAL[Evaluation Harness<br/>offline groundedness checks]
    end

    subgraph "Retrieval & Generation"
        SEARCH[Azure AI Search<br/>vector + hybrid index]
        AOAI[Azure OpenAI Service<br/>chat + embeddings models]
    end

    subgraph Data
        BLOB[Azure Blob Storage<br/>source KB documents]
        COSMOS[Azure Cosmos DB<br/>conversation & draft history]
    end

    subgraph Platform
        ENTRA[Microsoft Entra ID<br/>auth for internal tooling]
        AI_INSIGHTS[Application Insights<br/>traces, metrics, cost telemetry]
    end

    C --> APIM
    A --> APIM
    APIM --> FN
    FN -->|1. embed query| AOAI
    FN -->|2. retrieve top-k passages| SEARCH
    FN -->|3. generate grounded answer| AOAI
    FN -->|4. write draft + confidence| COSMOS
    FN -->|below threshold| A
    FN -->|outage / low confidence| HUMANQ[Human Agent Queue<br/>fallback path]
    BLOB -->|chunk + embed on publish| SEARCH
    ENTRA -.auth.-> APIM
    ENTRA -.auth.-> A
    FN -.telemetry.-> AI_INSIGHTS
    EVAL -.nightly eval run.-> AOAI
    EVAL -.nightly eval run.-> SEARCH
```

## Request Sequence

```mermaid
sequenceDiagram
    participant Cust as Customer
    participant APIM as API Management
    participant FN as Orchestrator (Azure Function)
    participant Search as Azure AI Search
    participant AOAI as Azure OpenAI Service
    participant Cosmos as Cosmos DB
    participant Agent as Support Agent

    Cust->>APIM: New support ticket
    APIM->>FN: Forward ticket (authenticated)
    FN->>AOAI: Embed ticket text
    AOAI-->>FN: Query embedding
    FN->>Search: Vector + keyword hybrid search (top-k)
    Search-->>FN: Ranked KB passages + citations
    FN->>AOAI: Generate answer (grounded on retrieved passages only)
    AOAI-->>FN: Draft answer + rationale
    FN->>FN: Compute confidence score
    FN->>Cosmos: Persist draft, sources, confidence, model version
    alt confidence >= threshold
        FN->>Agent: Show draft with citations for one-click approval
    else confidence < threshold
        FN->>Agent: Route to human queue, draft shown only as a hint
    end
    Agent->>Cust: Send reviewed/edited response
    Agent->>Cosmos: Log approve/edit/reject decision (feeds evaluation dataset)
```

## Ingestion Pipeline

1. Knowledge-base articles are authored/updated in the existing CMS and land in **Azure Blob Storage** on publish.
2. An event-triggered **Azure Function** chunks each document (semantic chunking, ~400-token windows with overlap), generates embeddings via **Azure OpenAI's embeddings model**, and writes both vectors and metadata (source URL, last-reviewed date, product area) into **Azure AI Search**.
3. Re-indexing completes within the 15-minute SLA defined in FR-006. Articles not reviewed in 90 days are flagged stale (RISK-04) and surfaced to the KB owner, not silently served.

## Scaling & Observability

- **Scaling:** Azure Functions (consumption plan) and Azure AI Search (Standard tier, replica/partition scaling) scale independently of each other; NFR-006 (3x volume) is met by adding Search replicas and raising the Functions plan tier — no architectural change required.
- **Observability:** every orchestrator invocation emits a trace to **Application Insights** including latency per stage (embed / retrieve / generate), token counts, and cost. A Grafana-style cost dashboard (fed from Application Insights) satisfies NFR-009.
- **Resilience:** the orchestrator wraps calls to Azure OpenAI and Azure AI Search in a circuit breaker; on sustained failure it routes directly to the human agent queue (P6), never returning an error to the customer-facing surface.

## Security

- **Entra ID** provides role-based access for the agent desk and admin tooling; the customer-facing channel authenticates via existing session mechanisms, not Entra.
- PII redaction runs on both the ingestion path (before indexing) and the query path (before the prompt is constructed) — see architecture principle P3 and RISK-03.
- Prompt-injection mitigations (input sanitisation, system-prompt isolation, tenant-scoped retrieval) are detailed in ADR-001.
