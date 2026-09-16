# Architectural characteristics and base architecture choice

This document is the yardstick for the whole submission. It records which base architecture was chosen for the Von Digitalis Estates and why, then defines the properties that every later addition, including every AI feature, must conform to. Those properties come in two kinds: **three invariants that are never traded**, and **four ranked characteristics that are**. Each has a definition, the reason it matters on this estate, a concrete fitness function that can be measured, and the components that carry it. The AI documents in `docs/ai/` cite this file when they show conformance.

## Base architecture options

| Option | Description | Fit to the constraints | Verdict |
|---|---|---|---|
| A. Cloud-centric | Thin devices publish straight to a cloud IoT hub; all logic, validation and alerting run in the cloud | Fails on patchy connectivity. Gates, animal alarms and keeper tools stop the moment the link drops. Cheapest to build, most fragile to run | Rejected |
| B. Edge-heavy | Everything runs on the estate; the cloud is a backup and reporting target only | Robust under partition, but expensive to scale to 15,000 visitors a day, slow to evolve, and it starves analytics and AI of data and elastic compute | Rejected |
| C. Edge-first hybrid, event-driven | Zone gateways run operational welfare alerts, ticket validation and buffering; hard-wired panels carry containment and duress; the cloud owns ticketing, analytics, AI and long-term data. Everything else is an event over MQTT and an event backbone | Meets every constraint. Each side degrades gracefully without the other. Fixed-cost edge, elastic cloud | **Recommended** |

The recommended option is recorded in [ADR-001](../adr/ADR-001-edge-first-hybrid-architecture.md).

## The three invariants

An invariant does not appear in the ranking, because ranking implies a rate of exchange and there is none. If a proposed design weakens one of these, it is rejected — it is not scored against cost, speed or convenience. This is the difference between a property the estate *prefers* and one it *depends on*.

### Invariant 1: Life safety

| | |
|---|---|
| Definition | The detection, alerting, dispatch and response path for containment breach and keeper duress carries no dependency on software, network, cloud or AI |
| Why it matters here | The estate holds venomous and poisonous animals. Everything else in this repository can be late; this cannot be wrong |
| Fitness functions | Quarterly drill with the cloud unreachable, and one a year with the zone gateway physically powered off, both meeting the response times in [06-safety-case](06-safety-case.md). No model inference, no cloud call and no software-actuated door or egress control anywhere on the path |
| Carried by | Hard-wired alarm panel, fixed duress stations, on-zone pagers, estate radio, named on-duty roles and a drilled procedure ([ADR-016](../adr/ADR-016-local-incident-response.md)) |

### Invariant 2: Data integrity

| | |
|---|---|
| Definition | Tickets, payments and animal records are never silently altered or double-counted, and are never lost to a backhaul failure. Loss of the durable local store itself is a bounded, declared exception, not a silent one |
| Why it matters here | A double-charged family or a lost vet record is a refund, a complaint or a dead animal. Store-and-forward replays events, so consumers must be idempotent |
| Fitness functions | Replay after an outage is idempotent: every scan carries a unique `scan_id` and a scan delivered twice is counted once, so occupancy and revenue never double-count. Payment reconciliation with the provider balances daily. Animal records are append-only with a full audit trail. AI never writes directly to these stores |
| Carried by | Idempotent consumers keyed on event sequence and device identifier, QoS 1 with deduplication, payment provider as the system of record for money, append-only welfare records, human-in-the-loop for every AI recommendation |

**The one bounded exception.** Store-and-forward makes an event durable locally before anything depends on it, which is what defeats the intermittent link. It does not defeat destruction of that local storage: if a gateway's disk is physically destroyed, its unforwarded queue is lost. The recovery point is how far behind the bridge was — seconds on a healthy link, up to hours of one zone's telemetry during an outage. It is bounded, detected and reconciled rather than silent, because ticket scans carry unique identifiers and surface as a reconciliation gap ([05-site-core-and-spare](../explainers/05-site-core-and-spare.md#the-spare-gateway), [fault catalogue](../fault-catalogue.md)). Stating it is the difference between an invariant and a slogan.

**What this invariant does not claim.** It is about *records*, not about admission decisions. A partitioned gateway cannot see other gateways' scan logs, so offline cross-gateway ticket re-use is possible; that is an availability-versus-fraud trade made deliberately under characteristic 1, and it is stated in full in the [known limitations](../limitations.md#the-availability-versus-fraud-trade). What integrity guarantees is that the resulting scans reconcile exactly once when the link returns.

### Invariant 3: Security and privacy

| | |
|---|---|
| Definition | Devices and services hold least privilege; personal data is collected with consent and minimised; video never leaves the estate |
| Why it matters here | Guests include families and children. Footfall must be measured without tracking individuals. Enclosure cameras and keeper voice notes are sensitive |
| Fitness functions | People counters store no identifiers. No video frame crosses the cloud bridge except sampled, blurred, reviewed audit frames. PII is redacted before any external model call **on every tier**, tested by routing a seeded payload to Tier 1a, Tier 1b and Tier 2 and asserting the redaction on each. Consent is re-checked immediately before any send, not only at scoring time. Every device has a unique certificate and a topic ACL. Payment card data never touches estate systems |
| Carried by | Per-device mTLS certificates and broker ACLs, edge-only vision, **deterministic PII redaction in the model access layer applied before candidate selection** ([ADR-012](../adr/ADR-012-llm-observability-and-kill-switches.md)), consent service, PCI scope delegated to the payment provider |

**Why redaction sits in the library and not in the catalogue.** Tier 2 is self-hosted outside the catalogue on purpose ([ADR-006](../adr/ADR-006-multi-provider-portfolio.md)). A control implemented only by the catalogue would therefore be bypassed by the failover the portfolio exists to provide, and this invariant would hold on a good day and fail during an incident — which is the definition of not being an invariant. Content filtering may degrade at Tier 2; redaction may not.

## The four ranked characteristics

These are genuine trade-offs, and ranking them is how conflicts get settled. When two conflict, the higher one wins. Availability under partition leads because a locked gate is a worse day than a late dashboard — but note that it leads *among these four only*, and never overrides an invariant.

### 1. Availability under partition

| | |
|---|---|
| Definition | Every function that a guest, keeper or ride operator needs in the moment keeps working when the estate cannot reach the cloud, and when one zone cannot reach another |
| Why it matters here | WiFi is patchy and backhaul will drop. Gates must admit ticket holders, enclosure alarms must reach keepers, and rides must be operable regardless |
| Fitness functions | Gate validation succeeds with backhaul down for 8 hours (target 100 percent of valid tickets admitted). An operational welfare alert reaches a keeper device within 30 seconds with the cloud unreachable. Guest app shows map, tickets and cached content with no connectivity. Zone gateway buffers 72 hours of telemetry without loss |
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

### 4. Elastic scalability

| | |
|---|---|
| Definition | The cloud tier scales with demand; the edge tier scales by adding zones and devices |
| Why it matters here | Visitors must triple in three years. Summer weekends will be several times a winter weekday. The estate cannot afford idle hardware sized for the peak |
| Fitness functions | Cloud services sustain 3x current event volume and 3x concurrent guest app sessions with no design change. A new zone is added by deploying one gateway and a topic prefix. Batch AI workloads run off-peak without affecting interactive latency |
| Carried by | Managed event backbone, stateless domain services on Kubernetes or serverless, model calls made in-process so there is no shared component to scale, per-zone gateways with a uniform image |

## Cost transparency is a constraint, not a fifth characteristic

Cost transparency is a **constraint on how the invariants and the four characteristics are met**. It makes the cost of each design choice visible and bounded, but it cannot justify weakening least privilege, consent, audit retention or any life-safety control.

| | |
|---|---|
| Definition | The cost of every feature is visible, including AI tokens, and budgets are enforced |
| Why it matters here | The alternative is the garden gnome business. AI spend can run away through abuse or a price change, and management needs to see what each feature costs |
| Fitness functions | Cost per capability is visible daily and reprices within one hour of a price-sheet change. Budget alerts fire at 120 percent of plan and automatic routing shifts at 150 percent. Per-user rate limits cap guest chat spend |
| Carried by | Model registry price sheet, token metering from the model access layer, a budget consumer that owns the hard cap, per-feature inference-profile cost tags |
| **What it may never do** | A budget cap may only degrade a feature to its designed non-AI fallback. It may not weaken a guardrail, bypass an eval gate, reduce audit retention, or touch anything on the life-safety path. This is why the hard cap flips a capability to Tier 3 rather than, say, routing to an unevaluated cheaper model |

## Fitness function scorecard

A fitness function that is never evaluated is a wish. This section states, for every fitness function above, **what is known today and what is not** — because this is a design submission with no running system, and pretending otherwise would be the worst thing in this repository.

Four verdicts are used, and the distinction between them is the point:

| Verdict | Meaning |
|---|---|
| **Arithmetic** | Settled now by calculation from the design. The working is shown or linked, and a reader can check it |
| **Inspection** | Settled now by reading the design or the configuration. Structural, not measured |
| **Qualified** | Does not hold as literally stated. The gap is named |
| **Unproven** | Cannot be settled without a running system. The metric, dataset and threshold are specified so the first measurement has something to be scored against |

### Settled by arithmetic

| Fitness function | Working | Verdict |
|---|---|---|
| Zone gateway buffers 72 hours of telemetry without loss | Busiest zone is 2.8 events/s; 72 hours spans ~30 operating hours; 302,000 events at 150–300 bytes is **45–91 MB against a 1 TB SSD** — four orders of magnitude of headroom ([capacity](03-edge-zone.md#capacity-what-those-devices-actually-generate)) | **Pass**, with the observation that 72 hours is a policy choice, not a hardware limit |
| Cloud services sustain 3x current event volume with no design change | 5,000 visitors produce 330,000 events/day; **tripling visitors produces 400,000, a factor of 1.21, not 3** — only the scan row scales with visitors. A literal 3x of *events* would be 28/s, still trivial for one managed instance | **Pass**, and the fitness function is a stricter test than the brief's growth target actually requires |
| An 8-hour outage backlog drains without operator involvement | 8h × 7/s × 300 B ≈ **60 MB**; under a minute at 10 Mbit/s even when throttled to protect live traffic | **Pass** |
| Signed ticket fits a QR code that scans from a phone screen | A 120–150 byte token Base32-encodes to **192–240 characters**. Base32 uses QR alphanumeric mode, whose capacity at error-correction level M is 221 (v8) and 262 (v9) | **Pass, with release test** |

**QR capacity is settled by the encoding mode.** [Signed-ticket-format](../implementation/signed-ticket-format.md) emits Base32, which falls entirely within QR alphanumeric mode. A 192-character token fits version 8 at EC-M; the 240-character maximum fits version 9 at EC-M. The implementation must retain the payload-size check and scanner/phone-screen release test, because capacity alone does not prove reliable scanning in the estate's lighting.

### Settled by inspection

These are structural properties. A reader can confirm each by reading the design; none of them requires a measurement.

| Fitness function | How a reader confirms it |
|---|---|
| No model inference, no cloud call and no software-actuated egress on the life-safety path | Every arrow on the [safety case](06-safety-case.md) response diagram is solid and terminates in hardware or a person. There is no model, no service and no door actuator on it |
| AI never writes directly to ticketing, payments or animal records | No AI component appears as a writer in the [container view](02-containers.md); the coupling rules state it, and [ADR-011](../adr/ADR-011-human-in-the-loop.md) governs it |
| A model swap for any capability is a registry change with zero code deployments | The capability map is a git artefact rendered to the parameter store and polled with a short TTL ([ADR-005](../adr/ADR-005-model-access-and-capability-contracts.md)). There is no code path that names a model |
| Replay after an outage is idempotent | Every scan carries a unique `scan_id` and consumers deduplicate on it ([signed-ticket-format](../implementation/signed-ticket-format.md)). The property is in the event schema, not in an operational procedure |
| Payment card data never touches estate systems | There is no card data path into any estate component; the payment provider holds it ([PCI scope](../explainers/03-pci-scope.md)) |
| No raw video crosses the cloud bridge | Vision runs on the gateway and publishes counts and confidence ([ADR-010](../adr/ADR-010-edge-vision-no-cloud-video.md)). The egress test that *proves* it is unproven; the absence of a publisher is inspectable now |
| Budget caps may only degrade a feature to Tier 3 | The hard cap flips the same feature flag the manual kill switch uses ([ADR-007](../adr/ADR-007-model-registry-and-cost-policies.md)); there is no code path from a budget event to a guardrail or an eval gate |

### Qualified, or failing as stated

The two that do not hold as literally written. Both are deliberate, and both are stated here rather than only in the document that benefits from them.

| Claim | Status | Where it is argued |
|---|---|---|
| "A ticket cannot be used twice for the same admission" (requirement F1.3) | **Qualified.** A permitted re-entry or ride/day-pass use is not a second admission. For a single-use scope, the local log prevents repeats at one gateway; a second gateway can admit before cloud reconciliation, for seconds online or for a partition. This is a chosen trade of bounded fraud against the first ranked characteristic | [known limitations](../limitations.md#the-availability-versus-fraud-trade), [ADR-003](../adr/ADR-003-offline-verifiable-signed-tickets.md) |
| "A revoked ticket is refused at the gate within a defined sync window" (F1.4) | **Holds only when the window is met.** During an outage longer than the sync window, a revoked ticket is admitted until the snapshot syncs | [signed-ticket-format](../implementation/signed-ticket-format.md#revocation) |

### Unproven until a system exists

These need hardware, an estate and a season of data. What the design owes them today is a specified metric, a named data source and a threshold — so that the first measurement is a verdict rather than an opinion. Every row below has all three, in [validation](../validation.md) and [eval-harness](../implementation/eval-harness.md).

| Fitness function | First opportunity to measure |
|---|---|
| 100 percent of valid tickets admitted with the backhaul down for 8 hours | Phase 1 partition drill |
| Enclosure alarm reaches a keeper device within 30 seconds, cloud unreachable | Phase 1 drill |
| Containment and duress response times, cloud off and gateway powered off | Phase 1 commissioning drill, then quarterly and annually |
| Device silence detected within 5 minutes | Phase 1, once last-will keepalives are configured against real devices |
| Occupancy available at 15-minute granularity within 2 minutes of arrival | Phase 2 |
| Welfare anomaly precision and recall against vet verdicts | Phase 2 gate — blind replay over historical periods containing known incidents |
| Piranha count interval coverage against manual counts | Phase 2 gate — three months of monthly cross-checks |
| Forecast MAPE against a seasonal-naive baseline | Phase 3 gate — a full season including a wet weekend and a bank holiday |
| Guide groundedness, citation coverage and refusal correctness | Phase 3 — golden sets exist as specifications now; scores exist only after a model runs against them |
| Kill switch propagates estate-wide inside a minute | Phase 3 drill |
| Cost per capability visible daily and repricing within an hour | Phase 3, once metering runs against a real bill |

**No number in this repository is presented as a measured result.** Where a figure appears, it is either arithmetic from the design (and shows its working), a published list price (and says so), or a threshold that something must later clear (and says which document defines the test).

## Conformance rule

Any new component, including every AI addition, must state in its own document how it upholds the three invariants and the four ranked characteristics. A deviation from a ranked characteristic may be justified and recorded in an ADR. **A deviation from an invariant may not**: there is no ADR that makes it acceptable for an AI feature to sit on the life-safety path, write directly to an animal record, or send raw video to the cloud. The AI overview at [00-ai-overview.md](../ai/00-ai-overview.md) carries the conformance table for the AI additions.

## Related

- [ADR-001 Edge-first hybrid architecture](../adr/ADR-001-edge-first-hybrid-architecture.md)
- [ADR-002 MQTT topology and store-and-forward](../adr/ADR-002-mqtt-topology-and-store-and-forward.md)
- [ADR-003 Offline-verifiable signed tickets](../adr/ADR-003-offline-verifiable-signed-tickets.md)
- [ADR-005 Model access through a hyperscaler catalogue](../adr/ADR-005-model-access-and-capability-contracts.md)
- [ADR-007 Model registry and cost policies](../adr/ADR-007-model-registry-and-cost-policies.md)
- [ADR-012 LLM observability and kill switches](../adr/ADR-012-llm-observability-and-kill-switches.md)
- [ADR-013 Privacy-preserving footfall and consent](../adr/ADR-013-privacy-preserving-footfall-and-consent.md)
- [AI overview and conformance table](../ai/00-ai-overview.md)
