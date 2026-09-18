# Multi-Cloud Architecture Portfolio

A single [ArcKit](https://arckit.org/)-style repository containing **three projects side by side** — one dashboard, one shared set of portfolio-wide principles, three independently-governed cloud architectures. This demonstrates ArcKit's multi-project mono-repo pattern (`projects/000-global` + multiple `projects/00X-*` folders sharing one `docs/` viewer), as opposed to the [three standalone single-project repos](https://harishsubash.github.io/arckit-usecase-hub/) also published under this account.

📖 **[View the documentation site](https://harishsubash.github.io/arckit-usecase-multicloud-portfolio/)**

## Structure

```
projects/
  000-global/                        <- portfolio-wide, applies to every project below
    ARC-000-PRIN-v1.0.md
  001-aws-order-processing/          <- AWS: serverless event-driven order processing
  002-azure-support-copilot/         <- Azure: RAG-based customer support copilot
  003-gcp-iot-analytics/             <- GCP: real-time IoT fleet analytics
```

This is the same repository shape ArcKit itself uses for multi-project workspaces: a single `000-global` folder holds principles that bind every project in the repo, while each numbered project folder is independently scoped, requirement-traced, and risk-assessed — but all browsable from the same dashboard, with a project switcher in the sidebar.

## Projects in This Portfolio

| # | Project | Cloud | Use Case |
|---|---|---|---|
| 001 | Order Processing Platform | AWS | Serverless, event-driven order processing for a mid-size e-commerce retailer |
| 002 | Customer Support Copilot | Azure | Retrieval-augmented AI copilot with human-agent handoff |
| 003 | Fleet IoT Analytics Platform | Google Cloud | Real-time streaming telemetry analytics for a delivery fleet |

Each project has its own Stakeholder Analysis, Requirements, Risk Register, Business Case, Architecture Strategy, Platform Design (with diagrams), project-specific Architecture Principles, and two Architecture Decision Records — nine artifacts each, 28 documents in total across the portfolio.

## Relationship to the Standalone Repos

The three projects here contain the same underlying architecture work as [`arckit-usecase-aws-order-processing`](https://github.com/harishsubash/arckit-usecase-aws-order-processing), [`arckit-usecase-azure-support-copilot`](https://github.com/harishsubash/arckit-usecase-azure-support-copilot), and [`arckit-usecase-gcp-iot-analytics`](https://github.com/harishsubash/arckit-usecase-gcp-iot-analytics) — reorganised here into one repository to demonstrate the mono-repo pattern specifically. Document IDs were renumbered (`ARC-001-*` / `ARC-002-*` / `ARC-003-*`) to avoid collisions across projects sharing one namespace, and a new portfolio-wide `000-global` principles document was written to genuinely apply across all three clouds — the original per-cloud principles are preserved as each project's own project-specific principles document.

## About This Project

This is a **demonstration/portfolio project**, not a real client deliverable. The fictional companies, their numbers, and their stakeholders are illustrative. All artifacts were drafted with AI assistance (Claude) and reviewed by [Harish Subash](https://github.com/harishsubash) — always validate AI-generated architecture output with human expertise before using it for real decisions.

The documentation viewer (`docs/index.html`) is reused from the MIT-licensed [ArcKit](https://github.com/tractorjuice/arc-kit) project — see [NOTICE.md](NOTICE.md) for attribution.

## License

MIT — see [LICENSE](LICENSE).
