# Business Case — Customer Support Copilot

| Field | Value |
|---|---|
| Document ID | ARC-002-SOBC-v1.0 |
| Status | Draft |
| Version | 1.0 |
| Date | 2026-09-17 |
| Author | ArcKit AI Assistant (reviewed by Harish Subash) |
| Project | 001-support-copilot |

> This business case is illustrative, written for a fictional mid-size SaaS company ("Meridian Workspace") to demonstrate the ArcKit documentation format. Figures are indicative planning estimates, not audited financials.

## Strategic Case

Meridian Workspace's support volume has grown 40% year-on-year while headcount has grown 12%. First-response time has crept from 2 hours to 6 hours on the standard tier, and CSAT has fallen 4 points over two quarters. The strategic driver is to absorb continued ticket growth without a proportional increase in headcount, while holding or improving CSAT — the copilot is one lever among several (self-serve deflection, better KB search) but is judged to have the largest single impact given ticket-content analysis showing ~55% of tickets are answerable from existing KB content.

## Options Considered

| Option | Description | Verdict |
|---|---|---|
| A — Status quo | Continue scaling the support team headcount linearly with volume | Rejected: cost grows unbounded, doesn't fix response-time trend |
| B — Fixed-flow chatbot | Rule-based/decision-tree bot for a narrow set of FAQs | Rejected: high build/maintenance cost per flow, brittle, ~15% of ticket types covered |
| C — RAG copilot (recommended) | Azure OpenAI + Azure AI Search grounded copilot assisting agents | **Recommended** |
| D — Fully autonomous AI resolution | AI closes tickets without agent review | Rejected for v1: violates P1/P3 (architecture principles), unacceptable risk profile at current trust/evaluation maturity |

## Economic Case

Option C is expected to reduce average handle time on eligible tickets by an estimated 25–35% (draft-and-edit vs. write-from-scratch), based on published case studies of comparable RAG-assisted support deployments, to be validated against Meridian's own pilot data before the business case is finalised for General Availability. Option B's fixed-flow approach caps out at low ticket coverage and would need continuous engineering investment per new flow, making its long-run cost-per-ticket-covered worse than Option C despite a lower headline build cost.

## Commercial Case

Build vs. buy: Meridian will consume Azure OpenAI Service and Azure AI Search as managed services rather than self-host an open-weight model, per architecture principle P4. This avoids GPU procurement, model-serving operational burden, and lets the platform team focus on the orchestration, retrieval quality, and evaluation layers — where the differentiated value actually lives. Re-evaluated annually against alternative providers via the model-abstraction layer established in ADR-001/ADR-002's related platform design.

## Financial Case (Indicative)

| Item | Estimate | Basis |
|---|---|---|
| Azure OpenAI Service (chat + embeddings) | ~$3,800/month at pilot volume (1 team, ~1,500 tickets/mo) | Estimated token usage per ticket × published Azure OpenAI pricing tiers |
| Azure AI Search (vector index, Standard tier) | ~$250/month | Index size for current KB (~1,200 articles) |
| Azure Functions + API Management + Cosmos DB | ~$400/month | Consumption-tier estimate at pilot volume |
| Engineering build (one-time) | ~8 engineer-weeks | Orchestrator, ingestion pipeline, evaluation harness |
| **Total pilot run-rate** | **~$4,450/month** | Scales roughly linearly with ticket volume handled by the copilot |

At full rollout (all 40 agents), run-rate is projected at ~$14,000–18,000/month, well below the fully-loaded cost of the additional 6–8 support agents that linear headcount growth (Option A) would otherwise require over the following 18 months.

## Management Case

**Phased rollout** (see also ARC-002-STRAT-v1.0):
1. **Shadow mode** (4 weeks) — copilot generates drafts, agents never see them; used purely to measure groundedness/accuracy against real traffic.
2. **Pilot** (6 weeks) — one 8-agent team sees drafts and can approve/edit/reject; weekly feedback loop.
3. **General availability** — phased team-by-team rollout once pilot quality gates (NFR-005) are consistently met, with the CFO cost dashboard live throughout.

**Governance:** Fortnightly steering group (Head of Support, Engineering Lead, Data Privacy Officer) owns the go/no-go decision at each phase gate.
