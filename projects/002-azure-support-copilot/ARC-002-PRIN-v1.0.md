# Architecture Principles (Project-Specific)

| Field | Value |
|---|---|
| Document ID | ARC-002-PRIN-v1.0 |
| Status | Live |
| Version | 1.0 |
| Date | 2026-09-17 |
| Author | ArcKit AI Assistant (reviewed by Harish Subash) |
| Scope | Global — applies to all projects in this repository |

## Purpose

These principles govern every AI-assisted architecture decision made for **Meridian Workspace**'s customer-support copilot programme. They are the standard against which designs, ADRs, and platform choices are reviewed.

## Principles

### P1 — AI augments agents, it does not replace judgement
The copilot exists to make support agents faster and more consistent, not to remove them from the loop. Any design that routes a customer to a fully autonomous AI-only resolution path for anything beyond low-risk, high-confidence queries is out of scope for this phase.

**Rationale:** Support quality and customer trust depend on a human able to override the system. Full autonomy is a possible future phase, not a v1 goal.

### P2 — Every answer must be grounded in retrieved source content
The copilot must never answer from the model's parametric memory alone. Each response is generated only from passages retrieved from the approved knowledge base, and every claim in a response must be traceable to a cited source document.

**Rationale:** Ungrounded answers are the primary source of hallucination risk in support contexts, where an incorrect answer can cause direct customer harm (billing, data loss, security guidance).

### P3 — Minimise personal data in prompts, logs, and indexes
Customer PII is redacted or tokenised before it reaches the language model or is written to logs, telemetry, or the vector index, unless strictly required to answer the query and covered by an explicit data-handling agreement.

**Rationale:** Reduces blast radius of a prompt-injection, logging, or model-provider incident, and keeps the system compatible with data-protection obligations without per-feature legal review.

### P4 — Prefer managed Azure PaaS over self-hosted infrastructure
Default to Azure OpenAI Service, Azure AI Search, Azure Functions, and other managed offerings over self-hosted models or self-managed vector databases, unless a documented requirement (cost at scale, data residency, or model choice) cannot be met by the managed service.

**Rationale:** A small platform team cannot safely operate GPU infrastructure, vector database clusters, and model-serving stacks at the reliability bar support relies on. Managed services trade some flexibility for materially lower operational risk.

### P5 — Ship behind measurable quality gates, not vibes
No model, prompt, or retrieval change reaches production without passing an offline evaluation suite (groundedness, relevance, harmful-content rate) and a shadow-mode comparison against the previous version.

**Rationale:** LLM behaviour is non-deterministic and regressions are often silent. A numeric gate is the only reliable way to catch a bad change before customers do.

### P6 — Design for graceful degradation
If Azure OpenAI Service, Azure AI Search, or any dependency is degraded or unavailable, the system fails over to a plain human-agent queue rather than surfacing an error or a low-quality answer to the customer.

**Rationale:** Support is a trust-critical channel; a visible AI failure is worse for the brand than briefly reverting to the pre-copilot experience.

### P7 — Every AI response is auditable after the fact
Each generated response is logged with the prompt, retrieved sources, model version, and confidence score, retained for the same period as the underlying support ticket.

**Rationale:** Enables root-cause analysis of complaints, supports the DPIA, and provides the evidence base for the quarterly quality review referenced in the business case.

### P8 — Build for incremental rollout, not a big-bang launch
The architecture must support running the copilot in shadow mode, then for a single pilot team, then generally available, without a redesign between stages.

**Rationale:** Keeps blast radius small while the team calibrates the confidence threshold and builds trust with support leadership.
