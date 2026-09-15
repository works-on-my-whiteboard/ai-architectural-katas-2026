# Device connectivity and cable runs

*What PoE, LoRaWAN and radio runs are, the physical limits on each, and why those limits — not the org chart — draw the zone boundaries.*

---

## The short answer

A **run** is a single cable from a switch to one device. The phrase "two zones keep the PoE and radio runs short" is a statement about physics: copper and radio both have distance limits, and the zone boundaries are drawn so that no device is further from its gateway than its link type can carry.

This is the least glamorous constraint in the architecture and one of the most decisive. It is the reason there are two ride zones rather than one.

## Power over Ethernet

**PoE** carries power and data on the same cable. That is why it is the default for fixed devices here: a vibration sensor bolted to an 18th-century ride needs no separate mains supply, no electrician, and no listed-building argument about running power to it. One cable does both.

The limit is hard and comes from the Ethernet specification: **100 metres** per run — conventionally 90 m of fixed horizontal cable plus up to 10 m of patch leads at each end. Past it, two separate things fail:

| What fails | Why |
|---|---|
| Data | Signal degrades beyond the specified channel length; the link becomes unreliable or does not come up at all |
| Power | Voltage drops over copper. A camera at the far end of an over-long run may not have enough power to boot, even if the data link is holding |

The second failure is the nastier one, because it looks like a faulty device rather than a cabling mistake.

So every PoE device must sit within 100 m of a switch, and that switch must reach the zone gateway. This is the constraint that sizes a zone.

## LoRaWAN

For sensors that are kilometres from anything, PoE is not an option at any zone size. **LoRaWAN** trades bandwidth for range: a few kilometres in open ground, years of life on a battery, and immunity to the patchy WiFi because it uses licence-free sub-gigahertz radio rather than 2.4 GHz.

The trade is real. LoRaWAN carries **kilobits per second, not megabits**, and in Europe the sub-gigahertz bands are duty-cycle limited, so a device may only transmit for a small fraction of the time. That is fine for a soil moisture reading every fifteen minutes. It is useless for a camera, and useless for high-rate ride vibration data.

This is why Z6 Gardens and Plants is described as LoRaWAN territory while the ride zones are not: the gardens have sparse, slow, distant sensors, which is exactly the shape LoRaWAN fits.

## Radio runs and the gateway backhaul

The same distance logic applies one level up, to the links carrying each zone gateway back to the site core: private LTE or 5G, or engineered point-to-point radio where fibre does not exist.

Radio range is not a hard cliff like the 100 m Ethernet limit, but the further the link, the weaker the signal, the more interference matters, and the more likely something breaks line of sight — a building, a mature tree, or a ride. Shorter engineered links are simply more predictable, which is the property the design wants. The connectivity table is explicit that licensed or engineered links are chosen because they are predictable and public WiFi is not.

## The link types, side by side

| Link | Used for | Reach | Why |
|---|---|---|---|
| PoE and Ethernet | Fixed devices in buildings: scanners, sensor kits, feeders, cameras | 100 m per run | Power and data on one cable; no dependence on WiFi |
| LoRaWAN | Low-rate outdoor sensors in Z6 and outlying enclosures | Kilometres | Long range, years of battery, immune to patchy WiFi. Low bandwidth |
| Private LTE or 5G, point-to-point radio | Zone gateway to site core where fibre does not exist | Site-scale | Licensed or engineered links are predictable |
| Fibre | Site core to cloud, and to buildings where it exists | Long | Primary backhaul. Carries no power |
| Guest WiFi | The guest app only | — | Never on the critical path. The app is offline-first |

The last row is a deliberate exclusion, not an omission. The brief says WiFi coverage is patchy, so no device and no critical function is allowed to depend on it.

## Why this splits the rides in two

Forty rides on a sprawling estate cover a lot of ground. Each ride zone carries roughly seventy-four devices: twenty scanners, ten people counters, four queue cameras, twenty vibration sensors and twenty cycle counters.

A single gateway for all forty would leave devices at the far end hundreds of metres away, well outside a 100 m run. The alternatives are both worse:

- **Chained switches.** A relay of switches to cover the distance. Every hop is another box to power, patch, monitor and eventually replace, and another single point of failure between a device and its gateway.
- **Long fibre to remote switch cabinets.** It works and it is sometimes necessary, but it means trenching fibre across a historic estate, and fibre carries no power, so there is still a powered cabinet with PoE switches at the far end.

Splitting into Rides North and Rides South **halves the maximum distance** from any device to its gateway. Each gateway sits near the middle of its twenty rides, and every device gets a short, direct run. The zone boundary is doing the job a pile of extra hardware would otherwise have to do.

## What draws a zone boundary

Read the zone table with this in mind and the boundaries stop looking arbitrary:

| Driver | Zones it explains |
|---|---|
| Cable distance alone | Z2 and Z3, the north-south ride split. Same function, two zones, purely geography |
| Criticality plus specialist hardware | Z4 and Z5. Welfare alarms must fire locally, and vision counting needs a GPU module |
| Distance too great for cable at any zone size | Z6. The answer is not another zone but a different radio technology |
| Scan rate and independence | Z1. Highest throughput, and it must admit guests with nothing else working |

Z6 is the instructive one. The gardens cannot be solved by drawing a smaller zone, because the sensors are spread over kilometres and are battery-powered. The link type changes instead.

## A useful test

When someone proposes a zone boundary, ask:

**Can every device in this zone reach its gateway on one run within the limit of its link type?**

If not, there are exactly three moves: split the zone, change the link type, or accept a chain of switches and the failure modes that come with it. The design here uses the first two and avoids the third.

## Where this is specified

- [03-edge-zone.md](../architecture/03-edge-zone.md) — the zone table with the reason each zone exists, per-zone device estimates, the connectivity table and the wiring diagram for one zone
- [ADR-001](../adr/ADR-001-edge-first-hybrid-architecture.md) — the decision to divide the estate into six to eight zones
- [ADR-002](../adr/ADR-002-mqtt-topology-and-store-and-forward.md) — devices connect to the zone broker over wired Ethernet or PoE where fixed, and over LoRaWAN via a gateway adapter for low-rate sensors; guest WiFi is never used for devices

Related explainers: [zone gateways](01-zone-gateways.md), the machine at the end of every one of these runs; [the estate-to-cloud path](02-estate-to-cloud-path.md), the link that starts where these end.
