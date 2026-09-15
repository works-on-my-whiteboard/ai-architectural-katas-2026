# Von Digitalis Estates Architecture Kata

A modern architecture for an estate that runs 40 historic rides, a 200-animal exotic collection and a carnivorous plant house, with 5,000 daily visitors growing to 15,000, patchy WiFi, and a budget for MQTT devices. The core is an edge-first, event-driven hybrid: zone gateways keep gates, alarms and keeper tools working offline while the cloud owns ticketing, analytics and AI. Seven AI uses sit on top, each with a fallback, and every generative model is addressed by capability rather than by name, so that model churn, price changes and vendor shutdowns are configuration events, not rewrites.

## Architectural characteristics, ranked

1. **Availability under partition.** Gates, alarms and keeper tools work with the cloud unreachable.
2. **Evolvability.** Services, models and vendors are replaceable behind stable contracts.
3. **Observability.** Every device, service and model call is traced and metered.
4. **Data integrity.** Tickets, payments and animal records are never lost or double-counted.
5. **Elastic scalability.** Cloud scales to 3x; the edge scales by adding zones.
6. **Cost transparency.** Per-feature cost is visible, including AI tokens.
7. **Security and privacy.** Least privilege on devices; consent-driven personal data.

Every AI addition is checked against these in [docs/ai/00-ai-overview.md](docs/ai/00-ai-overview.md).

## Start here

1. [docs/overview.md](docs/overview.md), the short narrative.
2. [docs/architecture/05-characteristics.md](docs/architecture/05-characteristics.md), then [01-context.md](docs/architecture/01-context.md) and [02-containers.md](docs/architecture/02-containers.md).
3. [docs/ai/00-ai-overview.md](docs/ai/00-ai-overview.md), the comprehensive AI view, then the seven targeted use cases.
4. [docs/uncertainty.md](docs/uncertainty.md), models, prices and vendors.
5. [docs/validation.md](docs/validation.md), how we know it works.
6. [docs/adr/README.md](docs/adr/README.md), the decisions.
7. [docs/explainers/](docs/explainers/README.md), concept notes on the ideas the rest of the repository assumes.

## Judges' criteria index

| Criterion | Where it is answered |
|---|---|
| Innovative use of AI | [docs/ai/00-ai-overview.md](docs/ai/00-ai-overview.md); [ai-01-piranha-counting.md](docs/ai/ai-01-piranha-counting.md); [ai-02-welfare-anomaly-and-brief.md](docs/ai/ai-02-welfare-anomaly-and-brief.md); [ai-04-guest-guide.md](docs/ai/ai-04-guest-guide.md) |
| Suitability given the constraints | [docs/architecture/02-containers.md](docs/architecture/02-containers.md); [03-edge-zone.md](docs/architecture/03-edge-zone.md); [04-data-flow.md](docs/architecture/04-data-flow.md); [ADR-001](docs/adr/ADR-001-edge-first-hybrid-architecture.md); [ADR-002](docs/adr/ADR-002-mqtt-topology-and-store-and-forward.md); [ADR-003](docs/adr/ADR-003-offline-verifiable-signed-tickets.md) |
| Appropriate level of detail | C4 views in [docs/architecture/](docs/architecture/); targeted views in [docs/ai/](docs/ai/); shapes in [docs/implementation/](docs/implementation/) |
| Dealing with uncertainty in AI | [docs/uncertainty.md](docs/uncertainty.md); [ADR-005](docs/adr/ADR-005-model-access-and-capability-contracts.md); [ADR-006](docs/adr/ADR-006-multi-provider-portfolio.md); [ADR-007](docs/adr/ADR-007-model-registry-and-cost-policies.md); [ADR-009](docs/adr/ADR-009-rag-over-fine-tuning.md); [ADR-014](docs/adr/ADR-014-caching-and-batch-cost-levers.md) |
| Characteristics match the existing architecture | [docs/architecture/05-characteristics.md](docs/architecture/05-characteristics.md); conformance table in [docs/ai/00-ai-overview.md](docs/ai/00-ai-overview.md); [ADR-004](docs/adr/ADR-004-classic-ml-vs-genai-selection.md) |
| Validation and verification of AI results | [docs/validation.md](docs/validation.md); [docs/implementation/eval-harness.md](docs/implementation/eval-harness.md); [ADR-008](docs/adr/ADR-008-evaluation-gated-promotion.md); [ADR-011](docs/adr/ADR-011-human-in-the-loop.md); [ADR-012](docs/adr/ADR-012-llm-observability-and-kill-switches.md) |

## Repository map

```
README.md
PRD.md
docs/
  overview.md
  architecture/
    01-context.md
    02-containers.md
    03-edge-zone.md
    04-data-flow.md
    05-characteristics.md
  ai/
    00-ai-overview.md
    ai-01-piranha-counting.md
    ai-02-welfare-anomaly-and-brief.md
    ai-03-crowd-flow-and-staffing.md
    ai-04-guest-guide.md
    ai-05-retention-and-revenue.md
    ai-06-ride-condition-monitoring.md
    ai-07-company-copilots.md
  uncertainty.md
  validation.md
  adr/
    README.md
    ADR-001 ... ADR-015
  explainers/
    README.md
    01-zone-gateways.md
    02-estate-to-cloud-path.md
    03-pci-scope.md
    04-device-connectivity.md
    05-site-core-and-spare.md
  implementation/
    mqtt-topics.md
    signed-ticket-format.md
    model-access-config.md
    eval-harness.md
  planning/
    solutioning.md
images/
```

## AI use cases

| ID | Use case | AI type | Runs where | Document |
|---|---|---|---|---|
| AI-1 | Piranha and aquatic population counting | Computer vision, owned model | Edge | [ai-01-piranha-counting.md](docs/ai/ai-01-piranha-counting.md) |
| AI-2 | Welfare anomaly detection and keeper daily brief | Time-series anomaly plus GenAI summary | Edge and cloud | [ai-02-welfare-anomaly-and-brief.md](docs/ai/ai-02-welfare-anomaly-and-brief.md) |
| AI-3 | Crowd flow, demand forecast and staffing | Forecasting and optimisation plus GenAI explanation | Cloud | [ai-03-crowd-flow-and-staffing.md](docs/ai/ai-03-crowd-flow-and-staffing.md) |
| AI-4 | Guest guide and personalised itinerary | GenAI RAG with live occupancy | Cloud, cached in app | [ai-04-guest-guide.md](docs/ai/ai-04-guest-guide.md) |
| AI-5 | Retention and revenue | Propensity scoring plus GenAI content | Cloud, batch | [ai-05-retention-and-revenue.md](docs/ai/ai-05-retention-and-revenue.md) |
| AI-6 | Ride condition monitoring | Vibration anomaly, advisory only | Edge | [ai-06-ride-condition-monitoring.md](docs/ai/ai-06-ride-condition-monitoring.md) |
| AI-7 | Company copilots | GenAI | Cloud | [ai-07-company-copilots.md](docs/ai/ai-07-company-copilots.md) |

## Decisions

All architecture decision records, with alternatives and trade-offs, are indexed in [docs/adr/README.md](docs/adr/README.md).

## Brief

The problem statement and product requirements are in [PRD.md](PRD.md). The original brief screenshots are in [images/](images/). Our planning notes and the analysis of the judges' criteria are in [docs/planning/solutioning.md](docs/planning/solutioning.md).
