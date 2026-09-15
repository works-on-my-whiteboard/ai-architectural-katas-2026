# Architectural characteristics and base architecture choice

This document is the yardstick for the whole submission. It records which base architecture was chosen for the Von Digitalis Estates and why, then defines the seven architectural characteristics that every later addition, including every AI feature, must conform to. Each characteristic has a definition, the reason it matters on this estate, a concrete fitness function that can be measured, and the components that carry it. The AI documents in `docs/ai/` cite this file when they show conformance.

## Base architecture options

| Option | Description | Fit to the constraints | Verdict |
|---|---|---|---|
| A. Cloud-centric | Thin devices publish straight to a cloud IoT hub; all logic, validation and alerting run in the cloud | Fails on patchy connectivity. Gates, animal alarms and keeper tools stop the moment the link drops. Cheapest to build, most fragile to run | Rejected |
| B. Edge-heavy | Everything runs on the estate; the cloud is a backup and reporting target only | Robust under partition, but expensive to scale to 15,000 visitors a day, slow to evolve, and it starves analytics and AI of data and elastic compute | Rejected |
| C. Edge-first hybrid, event-driven | Zone gateways run local safety, ticket validation and buffering; the cloud owns ticketing, analytics, AI and long-term data. Everything is an event over MQTT and an event backbone | Meets every constraint. Each side degrades gracefully without the other. Fixed-cost edge, elastic cloud | **Recommended** |

The recommended option is recorded in [ADR-001](../adr/ADR-001-edge-first-hybrid-architecture.md).

## The seven characteristics, ranked

Ranking matters. When two characteristics conflict, the higher one wins. Availability under partition outranks everything because a locked gate or a missed poison-enclosure alarm is a worse day than a late dashboard.

### 1. Availability under partition

| | |
|---|---|
| Definition | Every function that a guest, keeper or ride operator needs in the moment keeps working when the estate cannot reach the cloud, and when one zone cannot reach another |
| Why it matters here | WiFi is patchy and backhaul will drop. Gates must admit ticket holders, enclosure alarms must reach keepers, and rides must be operable regardless |
| Fitness functions | Gate validation succeeds with backhaul down for 8 hours (target 100 percent of valid tickets admitted). Enclosure alarm reaches a keeper device within 30 seconds with the cloud unreachable. Guest app shows map, tickets and cached content with no connectivity. Zone gateway buffers 72 hours of telemetry without loss |
| Carried by | Zone gateway (local MQTT broker, rule engine, ticket cache), signed tickets, offline-first guest and keeper apps, Tier 3 non-AI fallbacks for every GenAI feature |

### 2. Evolvability

| | |
|---|---|
| Definition | Services, models and vendors can be replaced behind stable contracts without rewriting callers |
| Why it matters here | AI models and providers will change faster than the estate's ten-year horizon. The best model today will not be the best next year, and prices and availability will move |
| Fitness functions | A model swap for any capability is a registry change with zero code deployments. A new device type joins the topic scheme without changing consumers. Any domain service can be redeployed independently within one working day |
| Carried by | Capability contracts resolved from configuration, model registry, MQTT topic scheme with a versioned payload envelope, domain services that own their data, event backbone as the only cross-service coupling |

### 3. Observability

| | |
|---|---|
| Definition | Every device, service and model call is traced, metered and attributable |
| Why it matters here | The Countess has no idea which parts of the estate are popular. Ops cannot deploy staff without occupancy data, and nobody can trust an AI feature that cannot be inspected |
| Fitness functions | 100 percent of model calls carry a trace with tokens, cost, latency and capability. Device silence is detected within 5 minutes via last-will messages. Occupancy per attraction is available at 15-minute granularity within 2 minutes of the events arriving |
| Carried by | OpenTelemetry across services, MQTT last-will and retained status topics, token metering emitted by the model access layer on every tier, stream processing for hot-path metrics, an LLM tracing store |

### 4. Data integrity

| | |
|---|---|
| Definition | Tickets, payments and animal records are never lost, duplicated or silently altered |
| Why it matters here | A double-charged family or a lost vet record is a refund, a complaint or a dead animal. Store-and-forward replays events, so consumers must be idempotent |
| Fitness functions | Zero duplicate admissions after a link outage and replay (each ticket and scan carries a unique identifier; consumers deduplicate). Payment reconciliation with the provider balances daily. Animal records are append-only with a full audit trail. AI never writes directly to these stores |
| Carried by | Idempotent consumers keyed on event sequence and device identifier, QoS 1 with deduplication, payment provider as the system of record for money, append-only welfare records, human-in-the-loop for every AI recommendation |

### 5. Elastic scalability

| | |
|---|---|
| Definition | The cloud tier scales with demand; the edge tier scales by adding zones and devices |
| Why it matters here | Visitors must triple in three years. Summer weekends will be several times a winter weekday. The estate cannot afford idle hardware sized for the peak |
| Fitness functions | Cloud services sustain 3x current event volume and 3x concurrent guest app sessions with no design change. A new zone is added by deploying one gateway and a topic prefix. Batch AI workloads run off-peak without affecting interactive latency |
| Carried by | Managed event backbone, stateless domain services on Kubernetes or serverless, model calls made in-process so there is no shared component to scale, per-zone gateways with a uniform image |

### 6. Cost transparency

| | |
|---|---|
| Definition | The cost of every feature is visible, including AI tokens, and budgets are enforced |
| Why it matters here | The alternative is the garden gnome business. AI spend is small against gate revenue but can run away through abuse or a price change, and management needs to see what each feature costs |
| Fitness functions | Cost per capability is visible daily and reprices within one hour of a price-sheet change. Budget alerts fire at 120 percent of plan and automatic routing shifts at 150 percent. Per-user rate limits cap guest chat spend |
| Carried by | Model registry price sheet, token metering from the model access layer, a budget consumer that owns the hard cap, per-feature inference-profile cost tags |

### 7. Security and privacy

| | |
|---|---|
| Definition | Devices and services hold least privilege; personal data is collected with consent and minimised; video never leaves the estate |
| Why it matters here | Guests include families and children. Footfall must be measured without tracking individuals. Enclosure cameras and keeper voice notes are sensitive |
| Fitness functions | People counters store no identifiers. No video frame crosses the cloud bridge except sampled, reviewed audit frames. PII is redacted before any external model call. Every device has a unique certificate and a topic ACL. Payment card data never touches estate systems |
| Carried by | Per-device mTLS certificates and broker ACLs, edge-only vision, PII redaction by the catalogue's managed guardrail policy, consent service, PCI scope delegated to the payment provider |

## Conformance rule

Any new component, including every AI addition, must state in its own document how it upholds each of the seven characteristics, or justify a deviation and get it recorded in an ADR. The AI overview at [00-ai-overview.md](../ai/00-ai-overview.md) carries the conformance table for the AI additions.

## Related

- [ADR-001 Edge-first hybrid architecture](../adr/ADR-001-edge-first-hybrid-architecture.md)
- [ADR-002 MQTT topology and store-and-forward](../adr/ADR-002-mqtt-topology-and-store-and-forward.md)
- [ADR-003 Offline-verifiable signed tickets](../adr/ADR-003-offline-verifiable-signed-tickets.md)
- [ADR-005 Model access through a hyperscaler catalogue](../adr/ADR-005-model-access-and-capability-contracts.md)
- [ADR-007 Model registry and cost policies](../adr/ADR-007-model-registry-and-cost-policies.md)
- [ADR-012 LLM observability and kill switches](../adr/ADR-012-llm-observability-and-kill-switches.md)
- [ADR-013 Privacy-preserving footfall and consent](../adr/ADR-013-privacy-preserving-footfall-and-consent.md)
- [AI overview and conformance table](../ai/00-ai-overview.md)
