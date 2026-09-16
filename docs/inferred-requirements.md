# Inferred requirements: disposition of I1 to I39

The brief separates what the Countess actually asked for from what a competent team would infer from the facts she gave. It then requires that anything inferred is **labelled as inferred in the deliverables**, and states that judgment about what *not* to build is part of the assessment ([problem statement §4 and §8](source/PROBLEM_STATEMENT.md)).

This register is that label. Every inferred item I1 to I39 gets exactly one disposition, with the architecture's response and the reason.

| Disposition | Meaning |
|---|---|
| **In scope** | Designed in this submission, with a document, an owner and evidence |
| **Guardrail** | The architecture constrains how this is done rather than building a product for it. A boundary, not a backlog item |
| **Deferred** | Deliberately not built. The reason is stated and a named interface or record is preserved so a later decision can add it without rewriting the core |

Nothing is silently dropped. A deferred row is a decision with a reason, which is the point the brief is testing.

## Rides and physical assets

| ID | Inferred requirement | Disposition | Response and why |
|---|---|---|---|
| I1 | Ride telemetry and predictive maintenance | **In scope** | [AI-6](ai/ai-06-ride-condition-monitoring.md) and F6.1: non-invasive vibration sensing per ride baseline, advisory only. 18th-century mechanisms that narrowly passed inspection are a safety and downtime risk, but the inspection regime keeps clearance authority ([ADR-011](adr/ADR-011-human-in-the-loop.md)) |
| I2 | Ride availability status for staff and visitors | **In scope** | F5.1 shows live ride status in the guest app from retained MQTT status topics; F6.5 records closures and downtime. Availability must be truthful including when the feed is stale, so status carries its own freshness |
| I3 | Queue length and wait time estimation | **In scope** | F2.2 and [AI-3](ai/ai-03-crowd-flow-and-staffing.md), reported with a stated confidence, and separating ride queue time (a cost) from enclosure viewing time (a value) |
| I4 | Capacity and throughput limits at 3x load | **In scope** | Two halves. Throughput is *measured* per attraction as riders or visitors per operating hour ([ADR-015](adr/ADR-015-occupancy-grain-and-operating-hour-normalisation.md)), and F1.6 can cap sales per slot. It is then *modelled forward* by [AI-8](ai/ai-08-simulation-gym.md), whose first question is which zones break first at 15,000 visitors a day and what the binding constraint is in each. Calibrated from observed operation rather than asserted from a spreadsheet |
| I5 | Offline ticket validation at gates and rides | **In scope** | [ADR-003](adr/ADR-003-offline-verifiable-signed-tickets.md) and [signed-ticket-format.md](implementation/signed-ticket-format.md). Direct consequence of C1 and F1; admission cannot wait for WiFi |
| I6 | Fraud and duplicate-use prevention | **Guardrail** | Signed tickets, a local scan journal and idempotent cloud reconciliation on ticket and scan identifiers. A partitioned estate cannot promise global instant uniqueness, so conflicts are recorded and reviewed rather than claimed impossible |
| I7 | Payments, refunds and PCI scope | **In scope** as a boundary; vendor selection deferred | [PCI scope explainer](explainers/03-pci-scope.md): card data reaches the payment provider directly and the estate stores an opaque reference. F1.4 covers refunds and revocation. Choosing and negotiating the provider is commercial work, listed out of scope in the PRD |
| I8 | Ticket types beyond family | **In scope** | F1.5 and F1.6 reuse one entitlement and validation path, so a new product is a catalogue entry rather than a new gate behaviour |
| I9 | Daily capacity caps | **Deferred** to the timed-entry phase | F1.6 is a Could. The mechanism (cap sales per slot) is designed and the occupancy grain to drive it exists; the commercial decision to cap a day's sales is the Countess's, and capping revenue before measuring demand would be premature |
| I10 | Zone access and re-entry; separately ticketed collection | **Guardrail** | The signed ticket carries an admission scope, so a gate makes a local decision about which zones a credential admits and whether re-entry is allowed. Whether the animal collection is separately ticketed is a pricing decision the architecture does not pre-empt |

## Ticketing, analytics and privacy

| ID | Inferred requirement | Disposition | Response and why |
|---|---|---|---|
| I11 | Footfall and dwell time per zone, anonymised | **In scope** | F2.1 and F2.2 via anonymous counters and gate scans in 15-minute buckets ([ADR-015](adr/ADR-015-occupancy-grain-and-operating-hour-normalisation.md)). This is how F2 is actually answered, and it needs no visitor identity |
| I12 | Privacy and consent, opt-in for personalisation | **In scope** | [ADR-013](adr/ADR-013-privacy-preserving-footfall-and-consent.md): counters store no identifiers; personalisation requires granular consent. Admission must never be a consent event |
| I13 | Popularity correlated with weather, day, events, pricing | **In scope** | F2.3 and [AI-3](ai/ai-03-crowd-flow-and-staffing.md) take these as forecast features. Popularity without context tells the Countess what happened, not what to do |

## Animal welfare

| ID | Inferred requirement | Disposition | Response and why |
|---|---|---|---|
| I14 | Enclosure environment monitoring | **In scope** | F3.1: per-enclosure kits for temperature, humidity and water chemistry, with species bands. Prerequisite for every welfare claim the estate makes |
| I15 | Escape and containment monitoring for venomous species | **In scope** | [ADR-016](adr/ADR-016-local-incident-response.md) and [06-safety-case.md](architecture/06-safety-case.md): deterministic local detection, hard-wired alerting, keeper duress stations and a drilled human response, with no dependency on cloud, WiFi or AI. Public safety in a collection that is newly public |
| I16 | Feeding schedules and food stock management | **Split.** Feeding **in scope**; stock management **deferred** | F3.2 records dispensed and leftover weight per feeding and flags deviations. Procurement, valuation and stock levels need a supplier and finance owner; the feeding-instance records preserve the interface for it |
| I17 | Vet records, medication and incident logging | **In scope** as records; statutory medication system **deferred** | F3.3 keeps append-only observations and vet records, and [ADR-016](adr/ADR-016-local-incident-response.md) adds the incident record. A controlled-drug register is a regulated clinical system, not something to improvise in an architecture kata |
| I18 | Alerting and escalation with human confirmation | **In scope** | F3.6 local alarms, [ADR-011](adr/ADR-011-human-in-the-loop.md) human decision, [ADR-016](adr/ADR-016-local-incident-response.md) role-based dispatch. A keeper, not a model, owns the consequential act |
| I19 | Regulatory reporting for exotic animal licences | **Deferred** | Jurisdiction and regulator-approved templates were not supplied and cannot be guessed. The append-only welfare and incident records with full provenance are the preserved interface, so producing a return later is a reporting job rather than a data-capture project |

## Staff operations

| ID | Inferred requirement | Disposition | Response and why |
|---|---|---|---|
| I20 | Staff deployment recommendations from popularity data | **In scope** | F2.4 and [AI-3](ai/ai-03-crowd-flow-and-staffing.md) produce an explained roster suggestion a manager accepts, edits or rejects. Direct answer to the Countess's "staffing is guesswork", without publishing a roster automatically |
| I21 | Incident and emergency handling | **Split.** Animal containment **in scope**; medical, lost child and ride fault **guardrail** | [ADR-016](adr/ADR-016-local-incident-response.md) covers containment and keeper safety. The estate's existing procedures own the rest, and [06-safety-case.md](architecture/06-safety-case.md) states explicitly that this does not become a general emergency-command platform — a system trusted for everything urgent gets used for things it was never validated for |
| I22 | Staff mobile tooling that works offline | **In scope** | F3.3 keeper tablets on the zone network with an offline outbox ([03-edge-zone](architecture/03-edge-zone.md)). Care evidence must be capturable where the signal is worst |

## Growth and revenue

| ID | Inferred requirement | Disposition | Response and why |
|---|---|---|---|
| I23 | Marketing, CRM and loyalty for returning visitors | **In scope** | F5.4 and [AI-5](ai/ai-05-retention-and-revenue.md), consent-gated at send time and measured against a holdout. Answers F7, and measurement is what stops it becoming spend without evidence |
| I24 | Dynamic or seasonal pricing | **Guardrail** | F5.5 is a Could, and is bounded: aggregate off-peak suggestions with floors and caps, a human approves, and no per-person price discrimination. Individual dynamic pricing is unfair and unauditable on a family day out |
| I25 | Secondary revenue: food, retail, events, sponsorship | **Deferred** | Named out of scope in the PRD. Per-attraction cost and capacity reporting (F2.6, F6.5) preserves the data a later commercial decision needs. Safe, reliable operations come before new revenue lines |
| I26 | Guest engagement: itineraries, concierge, wayfinding | **In scope** | F5.1, F5.2, F5.3 and [AI-4](ai/ai-04-guest-guide.md), offline-first with cached map and tickets, grounded answers with citations, and a guest-controlled plan |

## Platform and non-functional

| ID | Inferred requirement | Disposition | Response and why |
|---|---|---|---|
| I27 | Edge computing with store-and-forward | **In scope** | [ADR-001](adr/ADR-001-edge-first-hybrid-architecture.md) and [ADR-002](adr/ADR-002-mqtt-topology-and-store-and-forward.md). The direct consequence of C1 and C2, and the spine of the whole design |
| I28 | MQTT topic design, provisioning, firmware, device security | **In scope** | [mqtt-topics.md](implementation/mqtt-topics.md) covers the topic scheme, envelope and the device lifecycle: per-device certificates, topic ACLs, signed firmware and revocation. A few hundred cheap devices are a security perimeter, not an assortment of trusted sensors |
| I29 | Data retention and lifecycle for telemetry | **Guardrail** | Retention class per data type is stated in the PRD non-functional requirements and enforced by [ADR-013](adr/ADR-013-privacy-preserving-footfall-and-consent.md). Actual periods are a legal decision; the architecture makes the class explicit so the value can be set without redesign |
| I30 | Bounded cloud and AI spend | **In scope** | [ADR-007](adr/ADR-007-model-registry-and-cost-policies.md), [ADR-014](adr/ADR-014-caching-and-batch-cost-levers.md) and the [worked cost model](uncertainty.md#worked-cost-model). This is a small estate, and the runaway risk is abuse and loops rather than list prices |
| I31 | Differentiated availability targets | **In scope** | [Availability under partition](architecture/05-characteristics.md#1-availability-under-partition): gates and alerts high, analytics lower. Availability follows consequence — a late dashboard is acceptable, a late alert is not |
| I32 | Evolvability as AI providers and models change | **In scope** | [ADR-005](adr/ADR-005-model-access-and-capability-contracts.md), [ADR-006](adr/ADR-006-multi-provider-portfolio.md) and [uncertainty.md](uncertainty.md). Models are configuration, not code |
| I33 | Observability across edge and cloud | **In scope** | [Observability](architecture/05-characteristics.md#3-observability), [ADR-012](adr/ADR-012-llm-observability-and-kill-switches.md), device last-will and heartbeat. An edge failure must be visible before it quietly corrupts an operating decision |
| I34 | Network segregation | **Guardrail** | [03-edge-zone](architecture/03-edge-zone.md) and the [security-and-privacy invariant](architecture/05-characteristics.md#invariant-3-security-and-privacy): guest WiFi is never on a critical path, and enclosure controls are not reachable from it. [06-safety-case.md](architecture/06-safety-case.md) makes the same statement for the safety path |

## AI specific, implied by the judging criteria

| ID | Inferred requirement | Disposition | Response and why |
|---|---|---|---|
| I35 | Explicit statement of where AI is deliberately not used | **In scope** | [AI overview](ai/00-ai-overview.md#where-ai-is-deliberately-not-used) and [06-safety-case.md](architecture/06-safety-case.md). The strongest responsible-AI control is a specific, checkable boundary, not an ethics paragraph |
| I36 | Human-in-the-loop for high-stakes decisions | **In scope** | [ADR-011](adr/ADR-011-human-in-the-loop.md): a named role owns every decision about an animal, a ride or a price, and the accept-or-reject becomes a training label |
| I37 | Eval datasets, golden tests, drift and quality monitoring | **In scope** | [validation.md](validation.md), [eval-harness.md](implementation/eval-harness.md), [ADR-008](adr/ADR-008-evaluation-gated-promotion.md). Non-deterministic output needs both pre-release and production evidence |
| I38 | Fallback when a model or provider degrades | **In scope** | [ADR-006](adr/ADR-006-multi-provider-portfolio.md) tiers and [uncertainty.md](uncertainty.md) playbooks. Every generative feature has a designed, tested non-AI floor, so degradation is a known experience rather than an invented answer |
| I39 | Per-use-case cost monitoring for AI | **In scope** | F6.4 and [ADR-007](adr/ADR-007-model-registry-and-cost-policies.md): every call metered and priced from the registry sheet, tagged by capability. An owner can see whether a feature still earns its cost |

## Summary

Thirty-nine inferred items, each with exactly one primary disposition. Four are split, and the row says which half went where.

| Disposition | Count | IDs |
|---|---|---|
| **In scope** | 31 | I1, I2, I3, I4, I5, I7\*, I8, I11, I12, I13, I14, I15, I16\*, I17\*, I18, I20, I21\*, I22, I23, I26, I27, I28, I30, I31, I32, I33, I35, I36, I37, I38, I39 |
| **Guardrail** | 5 | I6, I10, I24, I29, I34 |
| **Deferred** | 3 | I9, I19, I25 |

\* Split: I7 designs the PCI boundary and defers vendor selection; I16 records feeding and defers stock management; I17 keeps records and defers a statutory medication register; I21 covers animal containment and leaves medical, lost-child and ride-fault events to existing estate procedure.

The nine items the brief itself marked "(scope)" — I1, I3, I5, I11, I14, I18, I23, I26, I27 — are all in scope, which is the floor the brief set. The work beyond that floor is there because the facts demanded it: **I15**, because a venomous collection that is newly open to the public needs a safety case rather than a sensor; and **I35 to I39**, because the judging criteria ask directly about where AI stops, how its output is validated, what happens when a provider fails, and what it costs. **I4** is answered twice over — measured from operation and modelled forward in the [simulation gym](ai/ai-08-simulation-gym.md) — because "which zone breaks first at 15,000 a day" is the question the whole growth target rests on.

## Related

- [Traceability](traceability.md), requirement by requirement, from the brief to the evidence
- [PRD](../PRD.md), the stated requirements and their acceptance criteria
- [Problem statement](source/PROBLEM_STATEMENT.md), the source these are inferred from
