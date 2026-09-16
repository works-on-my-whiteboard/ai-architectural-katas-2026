# The estate-to-cloud path

*What the brief means by "a way of getting information from the estate to the cloud", and why it is a separate problem from patchy WiFi.*

---

## The short answer

The estate-to-cloud path is the network link that carries data **off the estate grounds** to cloud services. The common name for it is the **backhaul**.

The brief states the constraint plainly:

> Cloud services may be used, but there must be a way of getting information from the estate to the cloud.

## Two hops, not one

The reason this is called out separately from "WiFi coverage on the park is patchy" is that they are different problems on different hops.

| Hop | Covers | The constraint |
|---|---|---|
| Device to zone gateway | Inside the park. A pH probe in the aquatic house to the gateway in the same building | WiFi is patchy, so use PoE, Ethernet and LoRaWAN instead |
| Zone gateway to cloud | Off the property, to a cloud region | **This is the estate-to-cloud path** |

A large rural estate does not automatically have a usable internet connection. Excellent coverage inside the park gets a reading to a gateway fifty metres away; it does nothing whatsoever to get that reading to a cloud region three hundred kilometres away. The brief splits the two deliberately.

## What the criterion is really testing

It is blocking the hand-wave. The common failure in this kind of submission is to draw a sensor, an arrow, and a cloud logo, and never say what the arrow *is*. Naming the path forces two answers:

1. **What physically carries it?** Fibre to the estate, private LTE or 5G, engineered point-to-point radio, cellular, satellite?
2. **What happens when it drops?** Rural links fail. A design that stops working when the link is down fails the suitability criterion regardless of how elegant the rest is.

The second question is the one that shapes the architecture.

## The answer this architecture gives

**Physically:** fibre from the site core to the cloud where it exists, with cellular failover at the core as a second path for the whole site. Within the estate, private LTE or 5G, or engineered point-to-point radio, carries each zone gateway back to the site core (Z8) where the backhaul is aggregated. Guest WiFi is never part of this.

**Logically:** the path is assumed **intermittent rather than absent**. The stated assumption is that cellular or fibre backhaul is available *at least intermittently*, and every design decision downstream treats a drop as normal operation rather than an incident.

That assumption produces four concrete responses:

| Response | Effect | Where |
|---|---|---|
| Disk-queued MQTT bridge | Each zone gateway bridges to the cloud hub through a local disk queue sized for 72 hours. A drop fills a buffer; a reconnect drains it in publication order. Consumers deduplicate on device identifier and sequence number | ADR-002 |
| Offline ticket validation | Entry does not use the path at all. Ed25519-signed tickets verify locally against cached keys and a revocation list | ADR-003 |
| Edge vision | Vision models run on the zone gateway. The path carries counts, confidence and sampled frames — kilobytes — instead of video — gigabytes | ADR-010 |
| Edge operational rules | Water chemistry and feeder faults are evaluated locally and page a keeper directly; containment contacts use the separate hard-wired alarm panel | 03-edge-zone.md |

The shape of all four is the same: **nothing critical waits on the path.**

## The failure case, concretely

The sequence diagram in [04-data-flow.md](../architecture/04-data-flow.md) traces it. Backhaul down for several hours: devices keep publishing to the local broker, operational alerts keep firing, gates keep admitting, the gateway keeps producing 15-minute occupancy buckets, and the bridge queue grows. Containment and duress stay on their independent hard-wired path. Backhaul restored: the queue drains in order, the cloud stream processor recomputes what it can from replayed raw events and trusts gateway-computed buckets otherwise.

The result is that an outage leaves **a complete history that arrived late**, not a gap. That distinction is how the data-integrity invariant is upheld under a partition.

## What it costs

Assuming an intermittent link is not free. The trade-offs worth stating:

- **Analytics are stale during an outage.** Dashboards, forecasts and the guest app's live occupancy show the last known state. This is accepted: the alternative is dashboards that are fresh and gates that are shut.
- **Gateways need disk and the buffer needs sizing.** 72 hours at peak rate per zone is a real capacity decision, and it is wrong if the outage is longer.
- **Reconciliation is real work.** Idempotent consumers and deduplication keys exist because replay is expected, not exceptional.

The registered risk is that the backhaul turns out to be worse than assumed. The mitigation is that everything critical is already edge-autonomous and the buffers are sized for a full operating day, so a worse link degrades analytics freshness rather than operations.

## Where this is specified

- [03-edge-zone.md](../architecture/03-edge-zone.md) — the connectivity table, link by link
- [04-data-flow.md](../architecture/04-data-flow.md) — the link-up and link-down traces
- [ADR-002](../adr/ADR-002-mqtt-topology-and-store-and-forward.md) — the bridge, the queue, the deduplication
- [ADR-003](../adr/ADR-003-offline-verifiable-signed-tickets.md) — why entry does not need the path
- [ADR-010](../adr/ADR-010-edge-vision-no-cloud-video.md) — why the path never carries video

Related explainers: [zone gateways](01-zone-gateways.md), the machines that make the path optional.
