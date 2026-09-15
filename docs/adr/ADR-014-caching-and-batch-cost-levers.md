# ADR-014: Caching and batch as the first cost levers

## Status

Accepted, 2026-09-16. Supersedes / Superseded by: none.

## Context

The guest guide will dominate generative spend once visitor numbers triple. The obvious response to a price rise is to move to a cheaper model, which also lowers quality. Several levers reduce spend without touching quality and should be exhausted first. The budget policies in [ADR-007](ADR-007-model-registry-and-cost-policies.md) need a defined order of response.

Forces:

- Most of each guest-guide prompt is the same stable prefix: system instructions and park knowledge.
- Many guest questions repeat ("where are the toilets", "when is the piranha feeding").
- Briefs, ops summaries and offer copy are produced overnight and are not latency-sensitive.
- Cached answers can go stale when the park changes.

### Alternatives considered

| Option | Summary | Why not (or why partially) |
|---|---|---|
| Model downgrade first | Route to a cheaper model as soon as cost rises | Trades quality for money before free options are used |
| No caching, real-time everything | Every call fresh at interactive pricing | Simplest, and the most expensive; wastes the stable prefix on every turn |
| Cache and batch first, model change last | Chosen | Quality preserved while the cheapest levers are applied; three of the five levers are catalogue features rather than estate code |

## Decision

Cost levers are applied in this order, and a model downgrade is only considered after the first four are in place:

1. Prompt caching of the stable prefix. System instructions, tool definitions and the retrieved park knowledge that does not vary per request are placed first and marked cacheable, so repeated turns pay the reduced cached-input rate. This is native to the catalogue ([ADR-005](ADR-005-model-access-and-capability-contracts.md)); the estate's work is prompt discipline, not machinery.
2. Semantic cache of repeated questions. Guest questions are matched against recent answers by embedding similarity; a hit returns the stored answer without a model call. The catalogue does not provide this, so it is built once in the Guest Engagement service, which is the only place a repeated question arrives; the other capabilities do not need it. Staleness rules: entries expire at the end of the operating day, are invalidated when the content store changes for any cited chunk, and are never used for questions classified as personal or time-sensitive.
3. Batch processing for non-interactive work. Nightly welfare briefs, ops summaries, offer copy and eval runs use the catalogue's batch inference pricing.
4. Effort settings. Routine calls run at lower effort; only capabilities whose eval score depends on it run at higher effort.
5. Model change. Only if the capability still breaches budget, and only to an eval-passing candidate.

Cache hit rate, batch share and effort distribution are tracked per capability as cost signals alongside spend.

## Consequences

### Positive

- Substantial cost reduction with no quality change, before any model decision.
- Latency improves for cached answers.
- Price changes are absorbed first by the levers, which buys time to negotiate or re-evaluate.

### Negative

- A stale cached answer can be wrong after a ride closure or price change; invalidation must be reliable.
- Prompt structure must be disciplined to keep the prefix stable; a stray timestamp breaks the cache silently.
- Batch output arrives hours later, so it cannot serve anything interactive.

### Trade-off analysis

| Quality attribute | Effect | Mitigation |
|---|---|---|
| Cost | Reduced substantially | Levers tracked as signals |
| Freshness | Semantic cache risk | Daily expiry, content-change invalidation, exclusions |
| Latency | Improved on hits; batch is delayed | Batch only for overnight work |
| Maintainability | Prompt discipline required | Cache hit rate alert when it drops |

## Related

- [ADR-005](ADR-005-model-access-and-capability-contracts.md), [ADR-007](ADR-007-model-registry-and-cost-policies.md), [ADR-009](ADR-009-rag-over-fine-tuning.md)
- [Uncertainty](../uncertainty.md), [Guest guide](../ai/ai-04-guest-guide.md), [Model access config](../implementation/model-access-config.md)
