# Explainers

Concept notes. Each one takes a single term that appears throughout the architecture and answers *what is this, why is it here, and what would break without it*.

These are deliberately separate from the rest of the repository. [docs/architecture/](../architecture/) specifies the system, [docs/adr/](../adr/) records the decisions and their trade-offs, and these notes explain the ideas underneath both. Nothing here is normative: if an explainer and a specification disagree, the specification is right and the explainer needs fixing.

| # | Explainer | Answers |
|---|---|---|
| 01 | [Zone gateways](01-zone-gateways.md) | Why the estate is divided into zones, what runs on the box in each one, and why not one gateway or fifty-five |
| 02 | [The estate-to-cloud path](02-estate-to-cloud-path.md) | What the brief means by getting information from the estate to the cloud, and why it is a different problem from patchy WiFi |
| 03 | [PCI scope](03-pci-scope.md) | What PCI DSS scope is, how it is pushed to a payment provider, and why a scope boundary is an architectural decision |
| 04 | [Device connectivity and cable runs](04-device-connectivity.md) | What PoE, LoRaWAN and radio runs are, the distance limits on each, and why those limits draw the zone boundaries |
| 05 | [Z8, the site core and the spare gateway](05-site-core-and-spare.md) | Why one zone has no devices, in what sense the zones are independent, and where the single point of failure really is |

Short definitions of these and other terms are in the [PRD glossary](../../PRD.md#glossary).
