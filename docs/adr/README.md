# Architecture Decision Records

Each record is a short text file, one decision per file, using the kata's format: Title, Status, Context (including alternatives considered), Decision, Consequences (with a trade-off table). Records are numbered in the order they were made; a superseded record keeps its number and points to its successor.

| ADR | Title | Status | Decision in one line |
|---|---|---|---|
| [001](ADR-001-edge-first-hybrid-architecture.md) | Edge-first, event-driven hybrid architecture | Accepted | Zone gateways keep gates, operational welfare alerts and keeper tools working offline; hard-wired panels carry containment and duress; the cloud owns ticketing, analytics and AI |
| [002](ADR-002-mqtt-topology-and-store-and-forward.md) | MQTT topology, QoS and store-and-forward | Accepted | Local broker per zone, QoS 1, persistent sessions, disk-queued bridge to the cloud, idempotent consumers |
| [003](ADR-003-offline-verifiable-signed-tickets.md) | Offline-verifiable signed tickets | Accepted | Ed25519-signed QR tokens verified at the gate with a small synced revocation list |
| [004](ADR-004-classic-ml-vs-genai-selection.md) | Classic ML versus GenAI selection principle | Accepted | Owned classic models for numeric and visual tasks; GenAI only where language is the input or output |
| [005](ADR-005-model-access-and-capability-contracts.md) | Model access through a hyperscaler catalogue | Accepted | One catalogue is the only integration; features name a capability, never a model, and the map is configuration rather than a service |
| [006](ADR-006-multi-provider-portfolio.md) | Model portfolio with a self-hosted tier outside the catalogue | Accepted | Two candidates inside the catalogue, a self-hosted open-weight tier in a separate failure domain, and a non-AI floor |
| [007](ADR-007-model-registry-and-cost-policies.md) | Model registry, price sheet and cost policies | Accepted | Versioned registry with price sheet as source of truth; budget thresholds shift traffic automatically within eval limits |
| [008](ADR-008-evaluation-gated-promotion.md) | Evaluation-gated model promotion | Accepted | Offline eval, shadow, canary and automatic rollback; promotion is a registry status change |
| [009](ADR-009-rag-over-fine-tuning.md) | Retrieval and prompting over fine-tuning | Accepted | Park knowledge is retrieved and cited at request time; no fine-tuning of closed models |
| [010](ADR-010-edge-vision-no-cloud-video.md) | Edge computer vision with no cloud video | Accepted | Vision models run on the zone gateway; only counts, confidence and blurred samples leave the estate |
| [011](ADR-011-human-in-the-loop.md) | Human-in-the-loop for welfare, ride and pricing decisions | Accepted | AI recommends; a named human decides; deterministic alarms remain the safety path |
| [012](ADR-012-llm-observability-and-kill-switches.md) | LLM observability, guardrails and kill switches | Accepted | OpenTelemetry traces, online quality signals with thresholds, and a per-feature switch to the non-AI fallback |
| [013](ADR-013-privacy-preserving-footfall-and-consent.md) | Privacy-preserving footfall and consent | Accepted | Anonymous counters and gate scans for popularity; personalisation only with granular consent |
| [014](ADR-014-caching-and-batch-cost-levers.md) | Caching and batch as the first cost levers | Accepted | Prompt caching, semantic cache, batch and effort settings are applied before any model downgrade |
| [015](ADR-015-occupancy-grain-and-operating-hour-normalisation.md) | Fifteen-minute occupancy buckets per operating hour | Accepted | One 15-minute grain for all popularity views, computed at the edge, with closed hours excluded from the denominator |
| [016](ADR-016-local-incident-response.md) | Local, role-based incident response for venomous containment and keeper safety | Accepted | Hard-wired detection, duress stations and a drilled radio-led human response; no cloud, no WiFi and no AI on the life-safety path |
| [017](ADR-017-simulation-gym.md) | Simulation gym for estate and delivery decisions | Accepted | Two owned, read-only calibrated models; hard safety and welfare constraints gate a scenario before revenue is compared |
