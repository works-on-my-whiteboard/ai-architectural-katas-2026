# ADR-009: Retrieval and prompting over fine-tuning closed models

## Status

Accepted, 2026-09-16. Supersedes / Superseded by: none.

## Context

The guest guide, welfare briefs and ops explanations all need park-specific knowledge: animals, rides, opening hours, feeding schedules, safety rules. That knowledge could be baked into a model by fine-tuning, or supplied at request time by retrieval and prompting. The brief's emphasis on vendor uncertainty makes the portability of that knowledge a first-order concern.

Forces:

- Park facts change weekly (a ride closes, a new enclosure opens, prices change).
- A fine-tuned closed model belongs to the provider's platform and cannot move.
- Retrieval keeps the source text under the estate's control and lets answers cite it.
- Some narrow tasks (tone, format) can benefit from tuning.

### Alternatives considered

| Option | Summary | Why not (or why partially) |
|---|---|---|
| Fine-tune a closed frontier model on park data | Provider-hosted tuning | Knowledge frozen at training time, locked to one provider, expensive to repeat, cannot cite sources |
| Fine-tune an open-weight model | Own the tuned weights | Portable, but still freezes facts and needs ML operations; acceptable for Tier 2 style or format tuning only |
| Retrieval with prompting on any model | Chosen | Facts stay current and portable; citations are natural; model swap needs no retraining |

## Decision

Park knowledge is a curated content store owned by the estate: structured records (enclosures, rides, hours, prices) and prose (history, accessibility, safety). It is chunked, embedded and indexed in a vector store the estate controls. Every generative feature retrieves relevant chunks at request time and the prompt requires answers to cite them; uncited claims fail the guardrail.

Prompt templates are written provider-agnostically with small per-model overlays for dialect differences. Embedding source text is stored alongside vectors so a change of embedding model is a batch re-embed, not a data loss.

Fine-tuning is not used on closed models. If a narrow style or format task ever justifies tuning, it is done on an open-weight model, the training set is kept, and the tuned artefact is treated as disposable.

## Consequences

### Positive

- Answers reflect this week's park, not last quarter's training run.
- Citations make outputs checkable by guests, keepers and the eval pipeline.
- Any model in the portfolio can serve any capability without retraining.
- No provider holds a unique asset the estate cannot recreate.

### Negative

- Retrieval quality becomes a system the team must tune and monitor (chunking, ranking, freshness).
- Longer prompts per request, which costs tokens; caching offsets part of this.
- Peak accuracy on very narrow tasks may be slightly below a tuned model.

### Trade-off analysis

| Quality attribute | Effect | Mitigation |
|---|---|---|
| Portability | Strongly improved | Own content store, own vectors, stored source text |
| Freshness | Improved | Content pipeline with review before publish |
| Cost per request | Higher input tokens | Prompt caching of stable chunks ([ADR-014](ADR-014-caching-and-batch-cost-levers.md)) |
| Accuracy | Depends on retrieval | Retrieval metrics in the eval suite |

## Related

- [ADR-004](ADR-004-classic-ml-vs-genai-selection.md), [ADR-005](ADR-005-model-access-and-capability-contracts.md), [ADR-014](ADR-014-caching-and-batch-cost-levers.md)
- [Guest guide](../ai/ai-04-guest-guide.md), [Welfare brief](../ai/ai-02-welfare-anomaly-and-brief.md), [Uncertainty](../uncertainty.md)
