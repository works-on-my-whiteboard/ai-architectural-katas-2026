# ADR-007: Model registry, price sheet and cost policies

## Status

Accepted, 2026-09-16. Supersedes / Superseded by: none.

## Context

The model access layer ([ADR-005](ADR-005-model-access-and-capability-contracts.md)) and the portfolio ([ADR-006](ADR-006-multi-provider-portfolio.md)) need a source of truth: which models exist, what they cost, how well they score and whether they may be used. The brief asks explicitly how a provider changing prices is handled. Without a registry, prices live in invoices discovered a month late, and model choices live in code.

Choosing a single catalogue changes what this has to do, in both directions. It gets easier: most price data now comes from one vendor's published list, cost attribution per feature is available natively through inference profile tags, and there is one bill to reconcile against. It gets harder in exactly one place: **the catalogue alerts on budget, it does not stop inference.** Account-level budget actions exist but are far too blunt to apply to one capability without taking down the others. Anything that actually refuses a call at a hard cap is the estate's own code.

Forces:

- Cost transparency is a ranked characteristic of the base architecture.
- The estate's finance owner must be able to see AI spend by feature and act on it before month end.
- Automated cost responses must not silently degrade quality below what a feature needs.
- Deprecations are announced in advance and should be handled on the estate's schedule.
- Nothing in the platform stops spend unless the estate builds it.

### Alternatives considered

| Option | Summary | Why not (or why partially) |
|---|---|---|
| Model names in code and prices in invoices | No registry | Every change is a deploy; spend is known after the fact |
| Catalogue console and cost tags only | Rely on native per-feature cost reporting | Good for seeing spend, useless for stopping it, and eval scores have nowhere to live |
| Registry without price sheet | Catalogue of models and scores only | Cannot reprice dashboards or enforce budgets |
| Registry with price sheet and automated policies | Chosen | Configuration, not code; immediate repricing; guarded automation |

## Decision

Maintain a versioned model registry. It is a configuration artefact in git, reviewed like code and rendered to the cloud parameter store alongside the capability map, not a service with an API. Each entry records: catalogue and region, model identifier, version pin, capabilities supported, context size, eval score per capability, price sheet (input, output, cached input, batch), rate limits, data-residency flags, status (candidate, active, deprecated, blocked) and deprecation date.

The price sheet is the source of truth for cost. Every model call logs input, output and cached tokens together with the registry price at the time, tagged by capability and feature flag, so a price change applied to the registry reprices every dashboard and forecast immediately. Native cost tags are reconciled against these logs monthly; the logs are the operational view because they are per capability and same-day, the bill is the audit view.

Budget policy per capability, enforced by a metering consumer the estate owns:

- Warn at 120 percent of planned daily or monthly spend.
- At 150 percent, shift traffic to the next eval-passing candidate by updating the capability map.
- At the hard cap, flip the capability's feature flag to Tier 3 for the remainder of the period and page the owner.
- Guest-facing capabilities carry per-user rate limits at the public API edge, because abuse is the realistic budget threat.

The consumer reads the metering stream, not the provider bill, so it acts within minutes rather than within a billing cycle. It is not on the request path and does not need to be highly available: if it is down, spend is unenforced until it returns, which is an hour of risk rather than an uncapped month.

A deprecation calendar raises a ticket 90 days before any pinned model's end of life.

Playbooks:

1. A better model appears: add as candidate; nightly eval; shadow 2 to 5 percent; canary 10 percent; promote to first candidate; all as status changes ([ADR-008](ADR-008-evaluation-gated-promotion.md)). If the model is not carried by the catalogue it cannot be adopted, which is the accepted cost of [ADR-005](ADR-005-model-access-and-capability-contracts.md); the registry records it as `blocked: not-in-catalogue` so the gap is visible rather than forgotten.
2. A provider changes prices: update the price sheet; dashboards reprice; if a budget threshold is breached the policy shifts traffic; apply caching, batch and effort levers ([ADR-014](ADR-014-caching-and-batch-cost-levers.md)); negotiate committed-use pricing on the catalogue.
3. A model is withdrawn: the entry is marked blocked, traffic moves to the next eval-passing candidate in the capability map, and evals are re-run on the remainder.
4. The catalogue itself is lost: circuit breakers open, Tier 2 takes over at eval-verified quality, and the second-catalogue escalation in [ADR-006](ADR-006-multi-provider-portfolio.md) is opened. Nothing is lost because prompts, traces, embeddings and eval sets are the estate's own.

```yaml
# Registry entry, design-level
model: frontier-mid-2026
catalogue: primary-region
status: active
capabilities: {chat.guide: 0.88, summarise.welfare: 0.91}
price_usd_per_million: {input: 1.00, output: 5.00, cached_input: 0.10, batch_input: 0.50}
deprecation_date: null
```

## Consequences

### Positive

- Cost is visible per feature the same day, not at invoice time.
- Price changes, new models and deprecations are handled as configuration.
- Automated responses are bounded by eval scores, so cost policy cannot route to a model that fails the capability.
- One vendor price list is markedly less work to keep accurate than several.

### Negative

- The hard cap is the estate's own code. A bug in it is a bug nobody else will find, and it is the only thing between an abused guest chat and an unbounded bill.
- The registry is another governed artefact with an owner, a review process and an audit trail.
- Automated shifting can surprise feature owners if notifications are weak.
- A better model outside the catalogue is visible in the registry but not adoptable, which turns a technical choice into a procurement conversation.

### Trade-off analysis

| Quality attribute | Effect | Mitigation |
|---|---|---|
| Cost transparency | Strongly improved | Token logs priced at request time; monthly reconciliation against the bill |
| Cost enforcement | Owned rather than inherited | Metering consumer tested in the drill; rate limits at the edge as the first line |
| Quality | Risk of automated downgrade | Only eval-passing candidates are routable; owner notified on every shift |
| Operability | New governance surface, no new service | Registry changes reviewed like code |
| Accuracy | Price sheet drift | Monthly reconciliation against invoices |

## Related

- [ADR-005](ADR-005-model-access-and-capability-contracts.md), [ADR-006](ADR-006-multi-provider-portfolio.md), [ADR-008](ADR-008-evaluation-gated-promotion.md), [ADR-014](ADR-014-caching-and-batch-cost-levers.md)
- [Uncertainty](../uncertainty.md), [Model access config](../implementation/model-access-config.md)
