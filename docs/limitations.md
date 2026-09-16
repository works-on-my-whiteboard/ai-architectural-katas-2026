# Known limitations and accepted trade-offs

The important trade-offs that would otherwise be scattered across the design. Each has its reasoning recorded; none is an oversight.

| Limitation | Consequence | Why it is accepted |
|---|---|---|
| **Single-use entitlement cannot be enforced globally at the edge** | A product with a single-use admission scope can be admitted at two zone gateways before reconciliation. An allowed re-entry or day-pass ride scan is not fraud and is recorded as such | Availability outranks fraud here: the cross-zone exposure is seconds while connected and up to the outage length while partitioned. It is reconciled, not silently ignored ([item below](#the-availability-versus-fraud-trade)) |
| **A refund issued during an outage is honoured at the gate** | A revoked ticket admits until the revocation snapshot syncs | Same trade. The exposure is hours of refunds, not systemic |
| **An offline kiosk has delegated ticket-issuance authority** | A stolen or compromised kiosk can mint only its bounded walk-up allocation until its key is revoked | The key sits in a secure element, has its own `kid`, narrow product scope, operating-day validity and daily cap, and replays a tamper-evident journal for reconciliation ([ticket format](implementation/signed-ticket-format.md#offline-kiosk-delegation)) |
| **Tier 1a and Tier 1b share one account and control plane** | Every generative feature degrades together in a catalogue-wide outage | Stated rather than dressed up as two vendors. Real independence lives at Tier 2, which is why it is funded ([ADR-006](adr/ADR-006-multi-provider-portfolio.md)) |
| **Tier 2 is a small warm pool, not full standby capacity** | A catalogue outage at peak exceeds Tier 2 headroom; the overflow lands on Tier 3 while Tier 2 scales | Full hot standby costs more than the traffic it protects ([uncertainty](uncertainty.md)) |
| **The estate-wide occupancy view is the one thing a partition really costs** | Ops sees seven live zone views instead of one estate view | Each zone view is correct; only the aggregate is late |
| **A second gateway failure before the first is replaced leaves a zone uncovered** | One cold spare serves seven zones; recovery is hours, not seconds | Seven hot standbys are not justified by the ranked characteristics ([05-site-core-and-spare](explainers/05-site-core-and-spare.md)) |
| **Welfare anomaly value is unproven until a season of data exists** | Phase 2 may conclude that fixed thresholds are as good | That is a legitimate outcome, and the phase gate is designed to produce it rather than hide it |
| **Automatic incident telemetry is absent when both the gateway and cloud fail** | The incident commander opens the controlled radio/paper log, which is entered later as a reconstructed record with its missing telemetry declared | The response path is allowed to fail neither recording nor safety: the manual log is the durable record-of-last-resort ([safety case](architecture/06-safety-case.md)) |
| **Cost figures are illustrative list prices, not quotes** | Actual spend will differ | The live price sheet is in the registry, not in prose ([uncertainty](uncertainty.md#worked-cost-model)) |
| **Device counts are design estimates, not a bill of materials** | A site survey will move them | Stated wherever they appear ([03-edge-zone](architecture/03-edge-zone.md)) |

## The availability-versus-fraud trade

Offline ticket verification is the decision this architecture is built on, and it has one honest cost. A gateway that cannot reach the cloud cannot know what other gateways have seen, so during a partition the estate trades a bounded amount of ticket fraud for the ability to keep admitting paying guests.

| Property | Online | Partitioned |
|---|---|---|
| Signature, window and zone check | Enforced locally | Enforced locally — needs only cached validation material and a clock |
| Re-use at the **same** gateway | A single-use scope is refused; an explicitly allowed re-entry is recorded | Same policy, from the local scan log |
| Re-use at a **different** gateway | A single-use scope can be admitted until cloud reconciliation, normally seconds later | **Can be admitted for the whole partition.** Detected on reconciliation, not synchronously at the gate |
| Revoked ticket | Refused once the current snapshot has reached the gateway | Admitted until the revocation snapshot syncs |
| Replay to the cloud | n/a | Idempotent on `scan_id`: a scan delivered twice is counted once, so occupancy and revenue are not double-counted |

The exposure scales with the reconciliation delay and is visible after the fact, because every scan carries its gateway identifier and admission scope. The alternative — putting a synchronous cloud lookup in every scan — fails the estate's first-ranked characteristic on the estate's most common failure. Recorded in [ADR-003](adr/ADR-003-offline-verifiable-signed-tickets.md), with the token design in [signed-ticket-format](implementation/signed-ticket-format.md).

## Related

- [05-characteristics](architecture/05-characteristics.md#fitness-function-scorecard), the verdict on every fitness function, including the two that are qualified by the trades above
- [ADR-003](adr/ADR-003-offline-verifiable-signed-tickets.md), the offline ticket decision these trades follow from
- [ADR-006](adr/ADR-006-multi-provider-portfolio.md), the model portfolio and why Tier 2 is a warm pool
- [walkthroughs](walkthroughs.md), the first two trades seen from the gate and the enclosure
- [uncertainty](uncertainty.md), what is genuinely unknown about models, prices and vendors
- [fault catalogue](fault-catalogue.md), the operational response to the corresponding failures
