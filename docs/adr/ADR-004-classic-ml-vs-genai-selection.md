# ADR-004: Classic ML versus GenAI selection principle

## Status

Accepted, 2026-09-16. Supersedes / Superseded by: none.

## Context

The brief asks for AI to be applied to counting piranhas, monitoring animal health and feeding, understanding crowd flow, growing repeat visits and improving profitability. The judges reward innovative use of AI but also score suitability, validation, and how the design deals with vendor uncertainty. Generative models are powerful with language and expensive, non-deterministic and vendor-dependent. Classic models are cheap, deterministic enough to test conventionally, and can be owned outright.

Forces:

- Counting fish and detecting a pH excursion do not need language.
- Explaining a roster or answering a guest's question does need language.
- Anything that runs on the edge must run without a cloud API.
- Every external model call carries cost and vendor risk.

### Alternatives considered

| Option | Summary | Why not (or why partially) |
|---|---|---|
| GenAI everywhere | Use a frontier multimodal model for counting, anomaly detection and text | Maximises the "AI" label, but pushes video to the cloud, makes counts non-deterministic, and ties animal safety to a vendor |
| Classic ML only | No generative models | Leaves the guest guide, keeper briefs and ops explanations as templated text; weak on the innovation and customer experience criteria |
| Decide per use case with a stated rule | Chosen | Each use gets the cheapest model class that meets the need, and the rule is auditable |

## Decision

Use classic ML or rules wherever the input is numeric or visual and the output is a number or a class. Use generative AI only where language is the input or the output. Classic models are owned artefacts (ONNX), versioned in the estate's own model registry and deployable to the edge. Generative models are reached only by naming a capability, which configuration resolves to a model in the estate's model catalogue ([ADR-005](ADR-005-model-access-and-capability-contracts.md)), and every generative feature has a non-AI fallback.

Applied to the use cases:

| Use case | Model class | Reason |
|---|---|---|
| AI-1 Piranha counting | Detection and tracking, owned | Visual in, number out; must run on the edge |
| AI-2 Welfare anomalies | Statistical and time-series anomaly, owned; GenAI for the daily brief | Numeric in, flag out; brief is language out |
| AI-3 Crowd flow | Forecasting and optimisation, owned; GenAI for the explanation | Numeric in, roster out; explanation is language |
| AI-4 Guest guide | GenAI with retrieval | Language in and out |
| AI-5 Retention | Propensity scoring, owned; GenAI for offer copy | Numeric in, score out; copy is language |
| AI-6 Ride condition | Vibration anomaly, owned | Numeric in, flag out; advisory only |
| AI-7 Copilots | GenAI | Language in and out |

## Consequences

### Positive

- The safety-relevant models (fish counts, water chemistry, ride vibration) are deterministic enough to test conventionally and are immune to vendor changes.
- Generative spend is confined to features where it adds visible value.
- The rule gives reviewers a one-line test for any proposed new AI feature.

### Negative

- Two model lifecycles must be run: an in-house training and deployment pipeline for owned models, and the capability-and-registry path for generative models.
- Owned models need labelled data and periodic retraining, which is effort the estate must staff.

### Trade-off analysis

| Quality attribute | Effect | Mitigation |
|---|---|---|
| Testability | High for owned models; lower for GenAI | Eval gates for GenAI ([ADR-008](ADR-008-evaluation-gated-promotion.md)) |
| Vendor independence | Strong for owned models | Capability contracts and the portfolio for GenAI ([ADR-006](ADR-006-multi-provider-portfolio.md)) |
| Development effort | Higher per owned model | Start with statistical bands; upgrade to learned models when data exists |
| Innovation | Balanced | GenAI used where it is visible to guests and keepers |

## Related

- [ADR-005](ADR-005-model-access-and-capability-contracts.md), [ADR-009](ADR-009-rag-over-fine-tuning.md), [ADR-010](ADR-010-edge-vision-no-cloud-video.md)
- [AI overview](../ai/00-ai-overview.md)
