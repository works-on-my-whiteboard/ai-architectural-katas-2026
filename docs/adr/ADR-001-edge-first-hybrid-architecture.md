# ADR-001: Edge-first, event-driven hybrid architecture

## Status

Accepted, 2026-09-16. Supersedes / Superseded by: none.

## Context

The estate must sell and validate tickets, monitor 200+ animals across 55 enclosures, understand where 5,000 (rising to 15,000) daily visitors go, and keep 40 historic rides safe. WiFi on the park is patchy, cloud services are permitted, and there is a budget for MQTT-capable devices installed throughout the park.

Forces:

- Gates, alarms and keeper tools must keep working when the link to the cloud drops. A queue of families at a dead turnstile is a lost day.
- Visitor numbers are expected to triple. The cloud can absorb that elastically; hardware on the estate cannot be re-bought every season.
- Analytics, ticketing, CRM and generative AI all want central data and elastic compute.
- The judges score suitability to constraints and whether AI additions match the base architecture. The base must therefore be explicit about its characteristics.

### Alternatives considered

| Option | Summary | Why not (or why partially) |
|---|---|---|
| A. Cloud-centric | Thin devices publish straight to a cloud IoT hub; all logic runs in the cloud | Fails the connectivity constraint outright. Every gate scan, alarm and keeper action becomes a cloud round trip on a link that is known to be unreliable |
| B. Edge-heavy | Everything runs on estate hardware; the cloud is backup only | Robust in the field, but 3x growth means buying and operating more on-site compute, analytics tooling is weak, and access to frontier AI models is awkward |
| C. Edge-first hybrid, event-driven | Zone gateways run local validation, alarms and buffering; the cloud owns ticketing, analytics, AI and long-term data | Chosen. Each side degrades gracefully without the other |

## Decision

Adopt option C. The estate is divided into six to eight zones (entrance, rides north and south, aquatic house, terrestrial house, gardens and plants, back-of-house). Each zone has a ruggedised gateway running a local MQTT broker, a time-series buffer, an alarm rule engine, a ticket validation cache and, where vision is needed, a GPU-class module. Gateways bridge to a cloud IoT hub over private LTE/5G or fibre with store-and-forward.

The cloud tier is event-driven: an IoT hub feeds an event backbone, stream processing produces hot-path metrics, and an open-format data lake plus time-series store hold history. Domain services (Ticketing and Payments, Guest Identity and Consent, Park Operations, Animal Welfare, Guest Engagement, Analytics) each own their data and communicate through events. Generative models are not a service of their own: they are reached from a hyperscaler model catalogue through a shared client library, so the cloud tier gains a dependency rather than a component ([ADR-005](ADR-005-model-access-and-capability-contracts.md)).

The ranked architectural characteristics are: availability under partition, evolvability, observability, data integrity, elastic scalability, cost transparency, security and privacy. Every later decision, including every AI addition, is tested against this list.

```mermaid
flowchart LR
  subgraph Edge["Estate edge, per zone"]
    DEV[MQTT devices] --> GW["Zone gateway: broker, buffer, rules, ticket cache, vision"]
    GW --> KT[Keeper tablet]
  end
  GW -- "MQTT bridge, store and forward" --> HUB[Cloud IoT hub]
  subgraph Cloud
    HUB --> BUS[Event backbone] --> SVC[Domain services and lake]
    SVC --> CAT[Model catalogue]
  end
```

## Consequences

### Positive

- Ticket validation, safety alarms and keeper workflows continue during a cloud partition.
- The cloud tier scales with visitor growth; the edge tier scales by adding a zone.
- Video and raw sensor streams stay on the estate, which reduces bandwidth and privacy exposure.
- The characteristics list gives the team and the judges a concrete conformance test.

### Negative

- Two deployment surfaces (edge and cloud) mean two release pipelines, two monitoring views and more operational skill.
- Gateway hardware is a capital cost per zone and must be maintained in outdoor and humid environments.
- Eventual consistency between edge and cloud must be designed for in ticketing and animal records.

### Trade-off analysis

| Quality attribute | Effect | Mitigation |
|---|---|---|
| Availability | Strongly improved at the edge | Local rule engine and ticket cache; tested partition drills |
| Simplicity | Reduced by two tiers | Shared observability stack; gateways as identical images |
| Cost | Higher up-front hardware spend | Standardised gateway build; cloud pay-as-you-grow |
| Consistency | Eventual between tiers | Idempotent events, deduplication on reconnect |
| Evolvability | Improved by service and event boundaries | Contracts on topics and events |

## Related

- [ADR-002](ADR-002-mqtt-topology-and-store-and-forward.md), [ADR-003](ADR-003-offline-verifiable-signed-tickets.md), [ADR-010](ADR-010-edge-vision-no-cloud-video.md)
- [Context](../architecture/01-context.md), [Containers](../architecture/02-containers.md), [Edge zone](../architecture/03-edge-zone.md), [Data flow](../architecture/04-data-flow.md), [Characteristics](../architecture/05-characteristics.md)
