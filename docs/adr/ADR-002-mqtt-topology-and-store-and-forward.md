# ADR-002: MQTT topology, QoS and store-and-forward

## Status

Accepted, 2026-09-16. Supersedes / Superseded by: none.

## Context

The brief provides a budget for MQTT-capable devices. Gate scanners, people counters, enclosure sensor kits, smart feeders, cameras and ride sensors will all publish over MQTT. The park's WiFi is patchy and the backhaul to the cloud is not guaranteed. Sensor data feeds operational welfare and maintenance alerts (which must fire locally) and analytics (which can tolerate delay). Venomous containment and keeper duress are separately hard-wired safety circuits, not MQTT rules ([ADR-016](ADR-016-local-incident-response.md)).

Forces:

- Operational alerts about water chemistry or a ride vibration spike must reach keepers and ops without the cloud.
- Analytics wants every reading eventually, in order, without duplicates inflating counts.
- Exactly-once delivery is expensive and fragile on constrained devices.
- Topic design decides how easy it is to add a zone, an enclosure or a new sensor type.

### Alternatives considered

| Option | Summary | Why not (or why partially) |
|---|---|---|
| Devices publish directly to the cloud hub | No local broker; each device holds its own cloud credentials and buffer | Constrained devices buffer poorly, credential management across hundreds of devices is fragile, and local alarms would still need a local subscriber |
| QoS 2 end to end | Exactly-once semantics on every hop | Higher latency and broker state on hardware that is already limited; most consumers can be made idempotent more cheaply |
| HTTP polling or REST from devices | Devices call service endpoints | No fan-out, no retained state, no last-will, poor fit for intermittent links |
| Local broker per zone with QoS 1 and bridging | Chosen | Local fan-out for alarms, buffered bridge to cloud, idempotent consumers absorb duplicates |

## Decision

Each zone gateway runs a local MQTT broker (Mosquitto or EMQX Edge). Devices connect to the zone broker over wired Ethernet or PoE where fixed, and over LoRaWAN via a gateway adapter for low-rate sensors. Guest WiFi is never used for devices.

Topic scheme: `vde/{site}/{zone}/{asset-type}/{asset-id}/{signal}`, for example `vde/main/aquatic/enclosure/E12/water/ph`. Payloads carry a device timestamp, a monotonic sequence number and a schema version. Sparkplug B is permitted where a self-describing payload standard is wanted.

QoS 1 is used on every hop with persistent sessions and retained last-known values. Every device sets a last-will message so the gateway can raise a "device silent" alarm. The zone broker bridges to the cloud IoT hub with a local disk queue; when the backhaul drops, messages accumulate and drain in order on reconnect. Cloud consumers deduplicate on `(asset-id, sequence)`.

Operational welfare and maintenance rules run on the gateway against the local broker and publish to `vde/{site}/{zone}/alarm/...`, which keeper tablets subscribe to over the zone network. They cannot create, suppress or delay a containment or duress alarm.

```yaml
# Design-level topic and QoS policy
topics:
  telemetry: vde/{site}/{zone}/{asset-type}/{asset-id}/{signal}
  alarm:     vde/{site}/{zone}/alarm/{severity}/{asset-id}
  command:   vde/{site}/{zone}/cmd/{asset-id}
qos: 1
session: persistent
retain: last-known-value on telemetry
last_will: required on every device
bridge:
  target: cloud-iot-hub
  queue: on-disk, drain in order on reconnect
```

## Consequences

### Positive

- Local fan-out means operational alerts are independent of the cloud.
- A single topic convention makes onboarding a new enclosure a configuration change.
- QoS 1 with idempotent consumers is cheap on devices and robust on flaky links.
- Retained values and last-will give operators an instant picture of device health.

### Negative

- Duplicates are possible and every cloud consumer must be idempotent.
- Ordering across zones is not guaranteed; only within a device's sequence.
- Disk queues on gateways need sizing and monitoring so a long outage does not overflow.

### Trade-off analysis

| Quality attribute | Effect | Mitigation |
|---|---|---|
| Reliability | Good on intermittent links | Persistent sessions, disk-backed bridge queue |
| Consistency | At-least-once, not exactly-once | Sequence numbers and deduplication in stream processing |
| Simplicity | Improved by one topic convention | Topic linting in gateway config validation |
| Latency | Local operational alerts in milliseconds; cloud analytics delayed during outages | Rules on the gateway |
| Capacity | Queue growth during outages | Sized queues, alert at 70 percent, oldest-analytics-first eviction policy |

## Related

- [ADR-001](ADR-001-edge-first-hybrid-architecture.md), [ADR-010](ADR-010-edge-vision-no-cloud-video.md), [ADR-013](ADR-013-privacy-preserving-footfall-and-consent.md)
- [Edge zone](../architecture/03-edge-zone.md), [Data flow](../architecture/04-data-flow.md), [MQTT topics](../implementation/mqtt-topics.md)
