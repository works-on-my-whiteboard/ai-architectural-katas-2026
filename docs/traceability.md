# Traceability: brief to requirement to design to evidence

Every claim in this submission must be followable from the brief to the thing that proves it. This document is that index. One row per requirement: where it came from, where it is designed, which decision governs it, and what would demonstrate it works.

**Origin** distinguishes what the Countess asked for from what we inferred, as the brief requires. `I1`–`I39` are inferred and dispositioned in [inferred-requirements.md](inferred-requirements.md).

### Two numbering schemes, and how to tell them apart

The brief's nine goals and our requirements catalogue both number from F1, and they do **not** line up — brief F5 is the piranha census, while requirement group F5 is guest growth and retention. Rather than renumber source material that is reproduced unchanged, the convention across this repository is:

| Form | Means | Example |
|---|---|---|
| **"brief F5"** — always with the word *brief* | One of the nine goals in [PROBLEM_STATEMENT.md](source/PROBLEM_STATEMENT.md) | brief F5 = track piranha population levels |
| **`F5.2`** — always with a dot and a sub-number | A row in [requirements-and-constraints.md](source/requirements-and-constraints.md), and a row in the table below | F5.2 = guest guide |

A bare `F5` with neither a dot nor the word "brief" is a defect; there should be none left. The Origin column below uses the first form, the Req column the second.

## Functional requirements

| Req | Priority | Origin | Designed in | Decision | Evidence |
|---|---|---|---|---|---|
| F1.1 Buy day tickets | Must | Brief F1 | [02-containers](architecture/02-containers.md), [signed-ticket-format](implementation/signed-ticket-format.md), [PCI scope](explainers/03-pci-scope.md) | [ADR-003](adr/ADR-003-offline-verifiable-signed-tickets.md) | Purchase completes under a minute; audit that no card data reaches estate systems |
| F1.2 Family passes | Must | Brief F1 | [signed-ticket-format](implementation/signed-ticket-format.md) | [ADR-003](adr/ADR-003-offline-verifiable-signed-tickets.md) | Group token admits each member once; group size enforced at the gate |
| F1.3 Validate tickets offline | Must | Brief F1, I5 | [03-edge-zone](architecture/03-edge-zone.md), [signed-ticket-format](implementation/signed-ticket-format.md) | [ADR-003](adr/ADR-003-offline-verifiable-signed-tickets.md), [ADR-002](adr/ADR-002-mqtt-topology-and-store-and-forward.md) | **Qualified, not fully satisfied.** Partition drill: valid tickets admitted with backhaul down, and single-use re-use refused at the same gateway. The acceptance criterion's literal "cannot be used twice" does **not** hold across disconnected gateways — a deliberate availability-versus-fraud trade, stated in [limitations](limitations.md#the-availability-versus-fraud-trade) and given a verdict in the [scorecard](architecture/05-characteristics.md#qualified-or-failing-as-stated) |
| F1.4 Refunds and revocation | Must | I7 | [signed-ticket-format](implementation/signed-ticket-format.md) | [ADR-003](adr/ADR-003-offline-verifiable-signed-tickets.md) | Revoked ticket refused within the sync window with an intermittent link |
| F1.5 Memberships | Should | I8 | [signed-ticket-format](implementation/signed-ticket-format.md), [02-containers](architecture/02-containers.md) | [ADR-003](adr/ADR-003-offline-verifiable-signed-tickets.md) | Renewal and member benefit validate on the same path as a day ticket |
| F1.6 Timed entry and capacity | Could | I9 | [02-containers](architecture/02-containers.md) | [ADR-015](adr/ADR-015-occupancy-grain-and-operating-hour-normalisation.md) | Slot cap prevents oversale; gate backstop test |
| F2.1 Measure occupancy | Must | Brief F2, I11 | [ADR-015](adr/ADR-015-occupancy-grain-and-operating-hour-normalisation.md), [04-data-flow](architecture/04-data-flow.md), [03-edge-zone](architecture/03-edge-zone.md) | [ADR-015](adr/ADR-015-occupancy-grain-and-operating-hour-normalisation.md), [ADR-013](adr/ADR-013-privacy-preserving-footfall-and-consent.md) | Sample count reconciliation; closed-hours fixture proves a repaired ride is not read as unpopular |
| F2.2 Queue length and dwell | Should | I3 | [AI-3](ai/ai-03-crowd-flow-and-staffing.md) | [ADR-015](adr/ADR-015-occupancy-grain-and-operating-hour-normalisation.md) | Stated confidence is calibrated on held-out days; queue time and viewing time reported separately |
| F2.3 Forecast demand | Should | I13 | [AI-3](ai/ai-03-crowd-flow-and-staffing.md) | [ADR-004](adr/ADR-004-classic-ml-vs-genai-selection.md) | Forecast error reported against actuals and against a naive-seasonal baseline |
| F2.4 Recommend staffing | Should | I20 | [AI-3](ai/ai-03-crowd-flow-and-staffing.md) | [ADR-011](adr/ADR-011-human-in-the-loop.md) | Manager accept, edit or reject is logged with a reason; no roster publishes itself |
| F2.5 Heatmaps and investment reporting | Must | Brief F2 | [ADR-015](adr/ADR-015-occupancy-grain-and-operating-hour-normalisation.md), [02-containers](architecture/02-containers.md) | [ADR-013](adr/ADR-013-privacy-preserving-footfall-and-consent.md) | Query audit proves the reporting surface reads aggregates only, never identifiers |
| F2.6 Value per attraction | Should | Brief F2, F8 | [07-attraction-economics](architecture/07-attraction-economics.md) | [ADR-015](adr/ADR-015-occupancy-grain-and-operating-hour-normalisation.md) | One asset's period figure recomputed by hand from source records; low-coverage assets excluded from the ranking |
| F3.1 Enclosure environment | Must | Brief F3, I14 | [03-edge-zone](architecture/03-edge-zone.md), [mqtt-topics](implementation/mqtt-topics.md) | [ADR-002](adr/ADR-002-mqtt-topology-and-store-and-forward.md) | Species band fixture fires on two consecutive out-of-band readings, not on one spike |
| F3.2 Monitor feeding | Must | Brief F4, I16 | [03-edge-zone](architecture/03-edge-zone.md), [mqtt-topics](implementation/mqtt-topics.md) | [ADR-002](adr/ADR-002-mqtt-topology-and-store-and-forward.md) | Dispensed against scheduled weight deviation flagged; missed dispense detected |
| F3.3 Keeper and vet records | Must | Brief F3, I17, I22 | [03-edge-zone](architecture/03-edge-zone.md), [02-containers](architecture/02-containers.md) | [ADR-011](adr/ADR-011-human-in-the-loop.md) | Offline outbox drill: a full shift logged with no signal, then synced with no loss or duplication |
| F3.4 Welfare anomalies | Should | Brief F3, F9, I18 | [AI-2](ai/ai-02-welfare-anomaly-and-brief.md) | [ADR-004](adr/ADR-004-classic-ml-vs-genai-selection.md), [ADR-011](adr/ADR-011-human-in-the-loop.md) | Blind historical replay over periods containing known incidents; keeper acceptance rate tracked |
| F3.5 Daily keeper brief | Should | I18 | [AI-2](ai/ai-02-welfare-anomaly-and-brief.md) | [ADR-009](adr/ADR-009-rag-over-fine-tuning.md), [ADR-011](adr/ADR-011-human-in-the-loop.md) | Every statement cites its data point; faithfulness scored on a golden set |
| F3.6 Safety alarms offline | Must | I15, I18 | [03-edge-zone](architecture/03-edge-zone.md), [06-safety-case](architecture/06-safety-case.md) | [ADR-016](adr/ADR-016-local-incident-response.md), [ADR-001](adr/ADR-001-edge-first-hybrid-architecture.md) | Quarterly drill with the cloud unreachable, and one a year with the zone gateway physically powered off |
| F4.1 Population estimate | Must | Brief F5 | [AI-1](ai/ai-01-piranha-counting.md) | [ADR-010](adr/ADR-010-edge-vision-no-cloud-video.md), [ADR-004](adr/ADR-004-classic-ml-vs-genai-selection.md) | Interval coverage against the manual count; a failed scene publishes no estimate rather than a bad one |
| F4.2 Manual reconciliation | Must | Brief F5 | [AI-1](ai/ai-01-piranha-counting.md) | [ADR-010](adr/ADR-010-edge-vision-no-cloud-video.md) | Monthly manual count recorded and compared; disagreement trended, not discarded |
| F4.3 Video stays on estate | Must | I12 | [AI-1](ai/ai-01-piranha-counting.md) | [ADR-010](adr/ADR-010-edge-vision-no-cloud-video.md) | Network egress test proves no frame crosses the bridge except sampled reviewed frames |
| F5.1 Guest app | Must | I26 | [02-containers](architecture/02-containers.md), [04-data-flow](architecture/04-data-flow.md) | [ADR-001](adr/ADR-001-edge-first-hybrid-architecture.md) | Airplane-mode journey: tickets, map and cached content all usable |
| F5.2 Guest guide | Should | I26 | [AI-4](ai/ai-04-guest-guide.md) | [ADR-009](adr/ADR-009-rag-over-fine-tuning.md), [ADR-005](adr/ADR-005-model-access-and-capability-contracts.md) | Groundedness and refusal-correctness suites gate every release; see [validation](validation.md) |
| F5.3 Personalised itinerary | Should | I26 | [AI-4](ai/ai-04-guest-guide.md) | [ADR-009](adr/ADR-009-rag-over-fine-tuning.md) | Itinerary replanned against changing queue data in simulation |
| F5.4 Re-engagement | Should | Brief F7, I23 | [AI-5](ai/ai-05-retention-and-revenue.md) | [ADR-013](adr/ADR-013-privacy-preserving-footfall-and-consent.md) | Consent checked at send time, not at scoring time; performance measured against a holdout |
| F5.5 Off-peak pricing | Could | Brief F8, I24 | [AI-5](ai/ai-05-retention-and-revenue.md) | [ADR-011](adr/ADR-011-human-in-the-loop.md) | Floor and cap guardrail test; no per-person price; human approval logged |
| F5.6 Feedback capture | Must | I37 | [AI-4](ai/ai-04-guest-guide.md), [validation](validation.md) | [ADR-012](adr/ADR-012-llm-observability-and-kill-switches.md) | A rating traces through to the quality signal that can kill the feature |
| F6.1 Ride condition advisory | Should | Brief F9, I1 | [AI-6](ai/ai-06-ride-condition-monitoring.md) | [ADR-011](adr/ADR-011-human-in-the-loop.md), [ADR-004](adr/ADR-004-classic-ml-vs-genai-selection.md) | Historical replay; engineer confirmation at inspection; ride never stopped automatically |
| F6.2 Support triage | Could | I21 | [AI-7](ai/ai-07-company-copilots.md) | [ADR-011](adr/ADR-011-human-in-the-loop.md) | Routing accuracy on a labelled set; a person confirms before routing |
| F6.3 Keeper voice notes | Could | I17 | [AI-7](ai/ai-07-company-copilots.md) | [ADR-011](adr/ADR-011-human-in-the-loop.md) | Field-level extraction accuracy; nothing enters a record unconfirmed |
| F6.4 AI cost visibility | Must | I30, I39 | [uncertainty](uncertainty.md), [model-access-config](implementation/model-access-config.md) | [ADR-007](adr/ADR-007-model-registry-and-cost-policies.md), [ADR-014](adr/ADR-014-caching-and-batch-cost-levers.md) | Daily reconciliation of the metered ledger against the provider bill; threshold alert test |
| F6.5 Maintenance log | Should | Brief F8, I1 | [07-attraction-economics](architecture/07-attraction-economics.md), [02-containers](architecture/02-containers.md) | [ADR-015](adr/ADR-015-occupancy-grain-and-operating-hour-normalisation.md) | Offline work-order drill; cost attributes to the correct asset and nowhere else |

## Constraints

| Constraint | Origin | Addressed in | Decision |
|---|---|---|---|
| C1 Patchy WiFi | Brief | [03-edge-zone](architecture/03-edge-zone.md), [zone gateways](explainers/01-zone-gateways.md), [device connectivity](explainers/04-device-connectivity.md) | [ADR-001](adr/ADR-001-edge-first-hybrid-architecture.md) |
| C2 Estate-to-cloud path | Brief | [04-data-flow](architecture/04-data-flow.md), [estate-to-cloud path](explainers/02-estate-to-cloud-path.md), [site core and spare](explainers/05-site-core-and-spare.md) | [ADR-002](adr/ADR-002-mqtt-topology-and-store-and-forward.md) |
| C3 MQTT device budget | Brief | [mqtt-topics](implementation/mqtt-topics.md), [03-edge-zone](architecture/03-edge-zone.md) | [ADR-002](adr/ADR-002-mqtt-topology-and-store-and-forward.md) |
| C4 Historic rides | Brief context | [AI-6](ai/ai-06-ride-condition-monitoring.md) | [ADR-011](adr/ADR-011-human-in-the-loop.md) |
| C5 Poisonous animals | Brief context | [06-safety-case](architecture/06-safety-case.md), [03-edge-zone](architecture/03-edge-zone.md) | [ADR-016](adr/ADR-016-local-incident-response.md) |
| C6 AI market uncertainty | Judging criteria | [uncertainty](uncertainty.md) | [ADR-005](adr/ADR-005-model-access-and-capability-contracts.md), [ADR-006](adr/ADR-006-multi-provider-portfolio.md), [ADR-007](adr/ADR-007-model-registry-and-cost-policies.md) |

## Deliverables the brief asked for

| Deliverable | Where it is |
|---|---|
| Overview narrative of how AI solves the estate's problems | [docs/overview.md](overview.md), with the comprehensive AI map in [00-ai-overview](ai/00-ai-overview.md) |
| Comprehensive and targeted diagrams, one per AI use case | [architecture/](architecture/) for the system views; [ai/ai-01](ai/ai-01-piranha-counting.md) to [ai-08](ai/ai-08-simulation-gym.md) each carry their own targeted view |
| One ADR per AI-related implementation, with trade-off analysis | [adr/README.md](adr/README.md); every ADR carries an alternatives table and a trade-off analysis table |
| Pertinent implementation details (optional) | [implementation/](implementation/): [MQTT topics](implementation/mqtt-topics.md), [signed ticket format](implementation/signed-ticket-format.md), [model access config](implementation/model-access-config.md), [eval harness](implementation/eval-harness.md) |
| Inferred requirements labelled as inferred | [inferred-requirements.md](inferred-requirements.md), all of I1 to I39 |
| Judgment about what not to build | The **Guardrail** and **Deferred** rows of [inferred-requirements.md](inferred-requirements.md), plus "Out of scope" in the [PRD](../PRD.md) |

## Cross-cutting capabilities

Three capabilities serve many requirements rather than one, so they appear here instead of owning a row above.

| Capability | Serves | Designed in | Decision | Evidence |
|---|---|---|---|---|
| Model promotion gate | Every GenAI capability behind F3.5, F5.2, F5.3, F5.4, F6.2 and F6.3: no model reaches production, and no model is replaced, without passing it | [eval-harness](implementation/eval-harness.md), [validation](validation.md) | [ADR-008](adr/ADR-008-evaluation-gated-promotion.md) | Golden-set score at or above the capability's minimum before promotion; shadow at 2-5% then canary at 10% with automatic rollback on signal regression; a promotion is a registry status change with no code deployment, which is itself the test |
| Simulation gym | I4 capacity at 3x load; informs F1.6 slot caps, F2.4 staffing mix, F2.6 investment choices, F6.5 maintenance windows, and the delivery sequence of the programme itself | [AI-8](ai/ai-08-simulation-gym.md) | [ADR-017](adr/ADR-017-simulation-gym.md) | Blind historical back-test with interval coverage; adversarial test that a profitable but unsafe scenario is rejected before ranking; predicted-against-actual recorded per scenario |
| Return per attraction | F2.6 and F6.5 directly; supplies the cost side of F2.5 investment reporting | [07-attraction-economics](architecture/07-attraction-economics.md) | [ADR-015](adr/ADR-015-occupancy-grain-and-operating-hour-normalisation.md) | One asset's period figure recomputed by hand; low-coverage assets excluded from the ranking |

The gym models the estate as it *might be*; [AI-3](ai/ai-03-crowd-flow-and-staffing.md) forecasts the estate as it *is*; [07-attraction-economics](architecture/07-attraction-economics.md) reports the estate as it *was*. Keeping those three separate is deliberate: only the last one produces the figures an investment decision is defended with.

## Judging criteria

The README carries the [criteria index](../README.md#submission-criteria-index). This document demonstrates appropriate detail — every requirement has a design and a named piece of evidence — and validation and verification of AI results, where the Evidence column is deliberately a test, a drill or a measurement rather than an assertion.

## Related

- [Inferred requirements](inferred-requirements.md), the disposition of everything inferred
- [PRD](../PRD.md), requirement wording, priorities and acceptance criteria
- [Validation](validation.md), how AI results are verified and misbehaviour is detected
- [Walkthroughs](walkthroughs.md), three pieces of that evidence followed end to end
- [Delivery plan](delivery-plan.md), which phase produces which evidence
- [Problem statement](source/PROBLEM_STATEMENT.md), the source of every stated requirement
