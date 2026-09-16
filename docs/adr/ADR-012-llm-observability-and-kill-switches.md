# ADR-012: LLM observability, guardrails and kill switches

## Status

Accepted, 2026-09-16. Supersedes / Superseded by: none.

## Context

Generative output drifts with model updates, prompt edits, content changes and adversarial guests. Observability is a ranked characteristic of the base architecture, and every device and service is already traced; model calls must be traced to the same standard.

Forces:

- A silent quality regression in the guest guide is discovered by guests first unless signals exist.
- Traces of guest conversations contain personal data.
- A misbehaving feature must be turned off in seconds without a deploy.
- Trace storage for every call costs money.

### Alternatives considered

| Option | Summary | Why not (or why partially) |
|---|---|---|
| Application logs only | Log request and response text | No structure, no metrics, no linkage to retrieval or cost, PII everywhere |
| Catalogue-native observability | Rely on the model catalogue's own logging and metrics | Real and useful, but it sees only the calls that reach the catalogue: nothing about retrieval, nothing about Tier 2 or Tier 3, and nothing that survives a move to another catalogue |
| Full tracing with online signals and kill switches | Chosen | One view across providers and features, owned by the estate |

## Decision

Every model call emits an OpenTelemetry trace, produced by the model access layer ([ADR-005](ADR-005-model-access-and-capability-contracts.md)) rather than by a gateway, that includes: capability, model and version, provider, prompt template version, retrieved chunk identifiers, token counts and priced cost, latency, guardrail outcomes, and the structured output or its schema failure. Traces flow to an LLM observability store (Langfuse, Arize Phoenix or equivalent) that the estate operates.

Online signals per capability: schema failure rate, citation coverage, refusal rate, guest thumbs up and down, keeper accept and reject rate, escalation-to-human rate, forecast error against actuals, count disagreement against manual counts, latency and cost. Each has a threshold and a time window; a breach alerts the owning team and, for canaries, triggers automatic rollback ([ADR-008](ADR-008-evaluation-gated-promotion.md)).

Guardrails run in two places, and the split follows one rule: **anything that must hold on every tier is implemented by the model access layer; only what may degrade with the catalogue is left to the catalogue.**

**PII redaction is ours, and it runs before routing.** The model access layer applies deterministic redaction and tokenisation to the prompt *before* it selects a candidate, so the payload that leaves the estate is already stripped whether it goes to Tier 1a, Tier 1b or the self-hosted Tier 2. This is not a stylistic choice: Tier 2 sits outside the catalogue by design ([ADR-006](ADR-006-multi-provider-portfolio.md)), so a redaction control that lived only in the catalogue would be bypassed by the exact failover the portfolio exists to provide — and the security-and-privacy invariant would hold in normal operation and fail during an incident. Redaction is deterministic rather than model-based for the same reason a probabilistic component cannot be a safety function: an invariant enforced by a model is not an invariant.

Content safety and denied topics remain a managed guardrail policy in the catalogue, named per capability and applied on the way in and on the way out. These may legitimately degrade at Tier 2, and the capability's own refusal rules — asserted by the library — are the floor that survives.

Schema validation, citation coverage and the capability's refusal rules are asserted by the model access layer after the response returns, because they are specific to the feature and have to hold for Tier 2 and Tier 3 answers too. Outcomes from both places are traced identically, so a guardrail regression looks the same whichever side produced it.

Every AI feature sits behind a feature flag that acts as a kill switch. The model access layer reads the flag from the same short-TTL parameter cache as the capability map, so flipping it routes the capability to its Tier 3 non-AI fallback estate-wide within a minute, with no deploy. The budget consumer in [ADR-007](ADR-007-model-registry-and-cost-policies.md) flips the same flag, so the manual and automatic paths to Tier 3 are one mechanism rather than two.

PII handling in traces: inputs are redacted before storage, raw text is retained for a short window in a restricted store for incident review only, and eval samples are pseudonymised before labelling.

## Consequences

### Positive

- Drift is detected from signals, not from complaints.
- One view across every provider survives a provider change.
- A bad model or prompt can be switched off in seconds.
- Traces feed the eval pipeline with real examples.

### Negative

- Trace storage and observability tooling are ongoing costs.
- Redaction is imperfect; some personal data may reach traces and must be governed.
- Threshold tuning takes time; early alerts will be noisy.
- Per-call assertion and metering code now runs in-process, so fixing a bug in it means deploying services rather than changing one configuration value. This is the observability cost of dropping the gateway and it is paid on every such fix.

### Trade-off analysis

| Quality attribute | Effect | Mitigation |
|---|---|---|
| Observability | Strongly improved | Standard OpenTelemetry attributes per call, emitted by the library so Tier 2 and Tier 3 calls are traced the same way |
| Privacy | Traces hold sensitive text | Redaction, short raw retention, restricted access |
| Cost | Storage and tooling | Sampling of full traces after the first months; metrics always kept |
| Operability | Alert noise early | Thresholds reviewed monthly; canary comparisons rather than absolutes |

## Related

- [ADR-005](ADR-005-model-access-and-capability-contracts.md), [ADR-008](ADR-008-evaluation-gated-promotion.md), [ADR-011](ADR-011-human-in-the-loop.md), [ADR-013](ADR-013-privacy-preserving-footfall-and-consent.md)
- [Validation](../validation.md), [Model access config](../implementation/model-access-config.md)
