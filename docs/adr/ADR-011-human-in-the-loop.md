# ADR-011: Human-in-the-loop for welfare, ride and pricing decisions

## Status

Accepted, 2026-09-16. Supersedes / Superseded by: none.

## Context

AI will flag sick or under-fed animals, unusual ride vibration and pricing opportunities. Each of these could in principle trigger an action automatically: change a feed schedule, close a ride, alter a price. Animals can be poisonous and expensive to treat, the rides are 18th century and freshly re-inspected, and pricing decisions carry legal and reputational weight. The judges ask how AI results are validated; the estate's owner asks who is accountable.

Forces:

- Animal welfare and ride safety are regulated and inspected; accountability must sit with a named person.
- Anomaly models produce false positives, especially early on with little training data.
- Keepers and inspectors will distrust and ignore a system that acts on its own.
- Every AI feature must degrade to a safe state without the model.

### Alternatives considered

| Option | Summary | Why not (or why partially) |
|---|---|---|
| Fully automated actions | Model outputs drive feeders, ride interlocks and price tables | Unacceptable accountability and safety risk; an anomaly model cannot be certified as a safety function |
| Automated with human veto | Actions execute unless a human objects within a window | Silent execution when nobody is watching, which is exactly when errors are costly |
| AI recommends, human decides | Chosen | Slower for the rare true emergency, but accountable, auditable and trust-building |

## Decision

Every AI output that concerns an animal, a ride or a price is a recommendation. A named human role owns the decision: keepers for feeding and welfare, the ride inspection lead for ride condition, the commercial manager for pricing. The system records the recommendation, the decision and the reason.

For welfare and rides, deterministic safety functions remain as they are: threshold alarms on water chemistry run on the gateway rule engine, and ride safety remains the inspection regime. AI only prioritises human attention.

Each accept or reject becomes a labelled example, so the models improve from the humans they serve. Acceptance rate is an online quality signal ([ADR-012](ADR-012-llm-observability-and-kill-switches.md)); a falling rate indicates a model that is losing trust.

Response-time cost is accepted: a keeper reads the morning brief rather than a feeder adjusting overnight. Where speed matters, the deterministic alarm path, not the AI path, provides it.

## Consequences

### Positive

- Accountability is clear to regulators, inspectors and the owner.
- False positives cost a few minutes of a keeper's attention, not a starved fish or a closed ride.
- Human decisions create the training data the models need.
- Trust builds because staff stay in control.

### Negative

- Recommendations wait for a human, so response time is bounded by staff availability.
- Keepers must spend time reviewing briefs; a noisy model becomes a burden rather than a help.
- Automation benefits (overnight adjustment, instant pricing) are foregone.

### Trade-off analysis

| Quality attribute | Effect | Mitigation |
|---|---|---|
| Safety | Strongly improved | Deterministic alarms unchanged; AI advisory only |
| Response time | Slower for AI-flagged events | Alarm path for urgent thresholds; brief delivered at shift start |
| Staff load | Increased review time | Precision threshold on recommendations; suppression of repeats |
| Auditability | Improved | Recommendation, decision and reason logged |

## Related

- [ADR-004](ADR-004-classic-ml-vs-genai-selection.md), [ADR-008](ADR-008-evaluation-gated-promotion.md), [ADR-012](ADR-012-llm-observability-and-kill-switches.md)
- [Welfare brief](../ai/ai-02-welfare-anomaly-and-brief.md), [Ride condition](../ai/ai-06-ride-condition-monitoring.md), [Retention and revenue](../ai/ai-05-retention-and-revenue.md)
