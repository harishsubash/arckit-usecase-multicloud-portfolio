# Requirements — Customer Support Copilot

| Field | Value |
|---|---|
| Document ID | ARC-002-REQ-v1.0 |
| Status | Draft |
| Version | 1.0 |
| Date | 2026-09-17 |
| Author | ArcKit AI Assistant (reviewed by Harish Subash) |
| Project | 001-support-copilot |

## Business Requirements

| ID | Requirement | Priority |
|---|---|---|
| BR-001 | Reduce average first-response time on eligible tickets without increasing escalation rate | Must |
| BR-002 | Every AI-generated answer must be traceable to a human-reviewable source in the knowledge base | Must |
| BR-003 | Agents remain the accountable owner of every ticket; the copilot assists, it does not close tickets autonomously in v1 | Must |
| BR-004 | Programme cost (Azure OpenAI + AI Search + supporting infra) stays within the approved monthly budget envelope | Must |
| BR-005 | Solution must be demonstrably compliant with the company's data-protection obligations before general availability | Must |

## Functional Requirements

| ID | Requirement | Priority |
|---|---|---|
| FR-001 | Given an inbound support ticket, the system retrieves the top-k relevant knowledge-base passages and generates a draft answer with inline citations | Must |
| FR-002 | The system computes a confidence score for each draft answer | Must |
| FR-003 | Draft answers below the confidence threshold are routed to a human agent with the draft shown as a suggestion, not sent automatically | Must |
| FR-004 | Draft answers at or above the confidence threshold are still shown to the agent for one-click approval during the pilot phase; direct-to-customer send is a later-phase decision | Must |
| FR-005 | Agents can edit, reject, or approve any draft answer before it reaches the customer | Must |
| FR-006 | The knowledge base ingestion pipeline re-indexes new or updated articles within 15 minutes of publication | Should |
| FR-007 | Every request/response pair is logged with prompt, retrieved sources, model version, and confidence score | Must |
| FR-008 | Agents can flag a bad response with a one-click reason code, feeding the evaluation dataset | Should |
| FR-009 | Customer-facing surfaces disclose that a response was AI-assisted | Must |
| FR-010 | System supports at least English and French knowledge-base content and queries at launch | Could |

## Non-Functional Requirements

| ID | Requirement | Priority |
|---|---|---|
| NFR-001 | P95 end-to-end response generation latency ≤ 4 seconds for the agent-facing draft | Must |
| NFR-002 | System availability ≥ 99.5% measured monthly for the orchestration path; on outage, falls back to human-only queue (see ARC-002-PLAT) | Must |
| NFR-003 | All customer data at rest and in transit is encrypted; PII is redacted from logs and the vector index before storage | Must |
| NFR-004 | Data residency: customer data used for retrieval and generation stays within the customer's contracted Azure region | Must |
| NFR-005 | Groundedness score (offline eval) ≥ 0.9 and hallucination rate ≤ 2% before any prompt/model change is promoted out of shadow mode | Must |
| NFR-006 | System scales to 3x current ticket volume (≈45,000/month) without architecture change | Should |
| NFR-007 | Full audit trail retained for 400 days, matching existing support-ticket retention policy | Must |
| NFR-008 | Authentication and authorisation for internal tooling via Microsoft Entra ID with role-based access | Must |
| NFR-009 | Monthly token spend is visible on a cost dashboard broken down by feature and environment | Should |

## Out of Scope (v1)

- Fully autonomous ticket resolution without agent review.
- Voice/phone channel support.
- Proactive outbound messaging (e.g. AI-initiated check-ins).
- Languages beyond English and French.

## Traceability

Requirements in this document are the basis for the Architecture Decision Records (`decisions/ARC-002-ADR-*`) and the Platform Design (`ARC-002-PLAT-v1.0.md`). FR-001–FR-005 and NFR-005 are the primary drivers for ADR-001 and ADR-002.
