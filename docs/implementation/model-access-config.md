# Model Access Configuration Shape

Illustrative and vendor-neutral. This shows the *shape* of the configuration the model access layer consumes; it is not a runnable file for any particular product. The point is that everything below is data, so the scenarios in [uncertainty.md](../uncertainty.md) are handled by editing it. There is no gateway service here: these files are rendered from git to the cloud parameter store and read by a shared client library ([ADR-005](../adr/ADR-005-model-access-and-capability-contracts.md)).

## Capability contract

What a feature declares. Feature code never names a model.

```yaml
capability: chat.guide
owner: guest-experience
requires:
  structured_output: true
  tool_use: true
  vision: false
  min_context_tokens: 32000
slo:
  p95_latency_ms: 2500
  max_cost_per_1k_requests_usd: 3.00
quality:
  golden_set: golden/chat-guide-v7.jsonl
  min_eval_score: 0.85
  min_citation_coverage: 0.95
data_class: may-leave-estate-pii-redacted
guardrail_policy: guest-facing-v4     # managed policy in the catalogue
fallback_tier3: faq-keyword-search
```

`data_class` decides which catalogue regions and which models are eligible. `guardrail_policy` names a policy configured in the catalogue rather than code in the estate. `fallback_tier3` names the behaviour the feature shows when the kill switch is on or every candidate is unavailable.

## Registry entry

One per model per catalogue region, plus one per self-hosted model. Prices are versioned with an effective date so historical cost is repriced correctly.

```yaml
model_id: frontier-mid-2026
catalogue: primary-region         # or: self_hosted
tier: 1a                          # 1a | 1b | 2
access: iam-role/model-invoke     # no API key exists to leak or rotate
status: active                    # candidate | active | deprecated | blocked
residency: eu
retention_days: 30
price_sheet:
  version: 2026-06-24
  effective_from: 2026-06-24
  input_per_mtok_usd: 2.00
  output_per_mtok_usd: 10.00
  cached_input_per_mtok_usd: 0.20
  batch_discount: 0.50
limits:
  requests_per_minute: 2000
  tokens_per_minute: 4000000
deprecation:
  end_of_life: null
  successor: null
eval_scores:
  chat.guide: { score: 0.91, citation_coverage: 0.97, run: 2026-09-08 }
  summarise.welfare: { score: 0.88, run: 2026-09-08 }
```

A price change is a new `price_sheet` version. A provider notice sets `deprecation.end_of_life` and starts the 90-day calendar. A model the estate would like but the catalogue does not carry is recorded with `status: blocked` and `reason: not-in-catalogue`, so the cost of the single-catalogue decision stays visible.

## Capability map

Ordered candidates per capability, with the circuit breaker and budget rules. This is the file a model swap edits.

```yaml
capability: chat.guide
candidates:
  - model: frontier-mid-2026        # tier 1a, catalogue primary region
  - model: frontier-alt-2026        # tier 1b, catalogue second family or region
  - model: open-weight-large-2026   # tier 2, self-hosted, separate account
circuit_breaker:
  error_rate_threshold: 0.05
  latency_p95_ms: 4000
  window_seconds: 60
  open_seconds: 120
budget:
  plan_usd_per_day: 90
  warn_at: 1.2
  auto_shift_at: 1.5
  hard_cap_usd_per_day: 300
  hard_cap_action: tier3            # enforced by the estate's metering consumer
caching:
  prompt_prefix_cache: true         # catalogue-native
  semantic_cache_ttl_seconds: 900   # built in the Guest Engagement service
effort: medium
refresh_ttl_seconds: 30             # how fast an edit reaches every service
```

Only candidates whose registry `eval_scores` meet the contract are eligible; the library refuses to call a candidate that has not passed. `refresh_ttl_seconds` is the whole rollout mechanism: an edit here reaches every service within half a minute and is rolled back the same way.

## Guardrail policy

Two halves, and which half a rule belongs in is decided by whether it must also hold for a Tier 2 or Tier 3 answer.

```yaml
capability: chat.guide

# Half one: configured in the catalogue, applied by the catalogue
managed_policy: guest-facing-v4
  pii_redaction: [email, phone, card_number, exact_location]
  denied_topics: [medical_advice, animal_feeding_by_guests, ride_safety_override]
  blocked_content: [profanity, self_harm]
  grounding_check: true

# Half two: asserted by the model access layer, on every tier
assertions:
  max_input_tokens: 4000
  schema: schemas/guide-answer-v3.json
  citations_required: true
  on_violation: retry_once_then_tier3

# Half three: not a model concern at all
rate_limit:                          # public API edge, before a call is made
  per_user_per_hour: 60
  per_device_per_hour: 120

tracing:
  store_retrieved_chunks: true
  store_output: true
  sample_for_human_review: 0.02
```

`on_violation` keeps the guest experience intact: one retry with the same model, then the Tier 3 answer. The rate limit deliberately sits at the edge rather than beside the model call, because the cheapest abusive request to handle is the one that never reaches a model.

## How the pieces connect

- The **contract** says what the feature needs.
- The **registry** says what exists, what it costs and how good it is.
- The **capability map** says which of those to use, in what order, and when to stop spending.
- The **guardrail policy** says what may go in and out, and who enforces each rule.

All four are configuration artefacts in git, reviewed like code and deployed without touching feature services. Nothing in this document describes a component that has to be kept running.
