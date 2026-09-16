# Fault catalogue

This is an index, not a new set of promises. It brings the designed failure behaviour from the architecture, AI and delivery documents into one page so that an operator can find the fallback, owner and drill.

| Fault | Immediate operating mode | Primary evidence or drill | Source |
|---|---|---|---|
| Estate-to-cloud backhaul cut | Gates validate locally; operational alerts stay local; zone buffers store and forward | Eight-hour admission and replay drill | [Walkthrough 1](walkthroughs.md#1-gate-admission-during-an-eight-hour-outage) |
| Site-core or cloud outage | Seven zones operate independently; dashboards and cloud sales become stale or pause | 72-hour buffer and cellular-failover tests | [Z8 and spare](explainers/05-site-core-and-spare.md) |
| Zone gateway failure, disk recoverable | Affected zone is restored from the Vision-class cold spare; the buffer moves with the disk; no containment or duress function is lost | Replacement and gateway-off safety drill | [Edge zone](architecture/03-edge-zone.md), [Safety case](architecture/06-safety-case.md) |
| Zone gateway failure, disk destroyed | As above, but that zone's unforwarded queue is lost. Recovery point is how far behind the bridge was: seconds on a healthy link, up to hours of one zone's telemetry during an outage. Scans carry unique identifiers, so the loss appears as a reconciliation gap rather than a silent miscount, and the gap is declared on the affected records | Reconciliation exercise with a deliberately truncated queue; confirm the gap is reported, not absorbed | [Z8 and spare](explainers/05-site-core-and-spare.md#the-spare-gateway), [Data flow](architecture/04-data-flow.md) |
| Local broker or operational-rule failure | Ticket validator and other independent edge functions continue where available; operational alerts use observation and the controlled procedure until restored | Device-health monitoring and operations drill | [MQTT ADR](adr/ADR-002-mqtt-topology-and-store-and-forward.md) |
| Gateway, cloud or AI failure during a containment or duress event | Alarm panel, sounder, beacon, pagers and radio remain independent of estate software | Quarterly containment and duress drill, including gateway powered off | [Safety case](architecture/06-safety-case.md) |
| Alarm panel or circuit fault | Panel supervision raises an audible, locally annunciated fault distinguishable from an alarm; the gateway independently flags panel silence as a second pair of eyes | Quarterly supervision test on a containment contact and a panic-station circuit | [Safety case](architecture/06-safety-case.md#layers-2-3-and-4-share-the-alarm-panel) |
| Alarm panel lost entirely | Layers 2–4 go together. Declared degraded-safety stop-work condition: the affected venomous area goes under manual observation on the pre-digital radio procedure until safety engineering restores and witnesses a panel test | Annual loss-of-panel drill, once with the gateway running and once with it also off | [Safety case](architecture/06-safety-case.md) |
| Cross-zone ticket re-use during a partition | Same-gateway repeat is denied; cross-zone single-use conflict is reconciled and flagged later | Reconciliation test with replayed scan logs | [Ticket format](implementation/signed-ticket-format.md#scan-dedupe-and-re-entry) |
| Refund or issuer-key revocation during a partition | Last retained revocation snapshot applies; stale refunds may admit until reconnect | Key-rotation and revocation exercise | [Ticket format](implementation/signed-ticket-format.md#revocation) |
| Offline kiosk compromise | Only a secure-element key's short-lived, capped walk-up scope can be minted; journal reconciliation disables the issuer key on anomalies | Kiosk cap, journal and issuer-revocation test | [Offline kiosk delegation](implementation/signed-ticket-format.md#offline-kiosk-delegation) |
| Model-specific or regional Tier 1 failure | Circuit breaker routes to Tier 1b | Shadow, canary and breaker test | [ADR-006](adr/ADR-006-multi-provider-portfolio.md) |
| Catalogue or primary-account failure | Tier 2 serves warm-pool capacity; surplus traffic uses Tier 3 | Monthly warm-pool and Tier-3 exercise | [Uncertainty](uncertainty.md#failure-scenarios) |
| Tier 2 failure or budget hard cap | Calling service uses its Tier 3 non-AI fallback | Kill-switch and budget-cap drill | [AI overview](ai/00-ai-overview.md#failure-scenarios) |
| Model, prompt or retrieval regression | Promotion is blocked, rolled back or killed by registry state | Golden set, shadow and canary results | [Validation](validation.md), [ADR-008](adr/ADR-008-evaluation-gated-promotion.md) |
| Incomplete telemetry or model drift | Decision is marked low coverage or falls back to rules/manual judgement | Per-use-case coverage and drift checks | [Validation](validation.md), [AI overview](ai/00-ai-overview.md) |

## Operating rule

No fault may turn a degraded feature into an unbounded one: a ticket fallback is controlled and reconciled, a budget fallback cannot weaken a guardrail, and an AI fallback never reaches a safety, payment or animal-record write path.

## Related

- [Walkthroughs](walkthroughs.md), the three primary drills
- [Delivery plan](delivery-plan.md), evidence gates by phase
- [Safety case](architecture/06-safety-case.md), the independent life-safety path
- [Uncertainty](uncertainty.md), model and provider failure handling
