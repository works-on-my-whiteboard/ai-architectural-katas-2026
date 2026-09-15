# MQTT topic scheme and payload envelope

This document defines how every device, gateway and cloud bridge on the estate names its messages, which quality of service each class of signal uses, and the payload envelope that makes replay and deduplication possible. It is the contract that lets a new device type, a new zone or a new consumer join without changing anything else, which is how the platform earns its evolvability characteristic.

## Topic scheme

```
vde/{site}/{zone}/{asset-type}/{asset-id}/{signal}
```

| Segment | Values | Notes |
|---|---|---|
| `site` | `main` today; future sites get their own value | Keeps the door open for a second estate |
| `zone` | `z1` to `z8` as defined in [03-edge-zone.md](../architecture/03-edge-zone.md) | Brokers are per zone; the bridge maps the prefix unchanged |
| `asset-type` | See the table below | One word, lower case |
| `asset-id` | Stable identifier from the asset register, for example `E12`, `R07`, `G03` | Never reused |
| `signal` | Signal name, may have sub-levels such as `water/ph` | Lower case, slash separated |

Example: `vde/main/z4/enclosure/E12/water/ph`

## Asset types and signals

| Asset type | Signals | Rate | QoS | Retained |
|---|---|---|---|---|
| `gate` | `scan`, `status`, `heartbeat` | Per scan; heartbeat every 60 s | scan 1, status 1, heartbeat 0 | status and heartbeat yes, scan no |
| `counter` | `count/in`, `count/out`, `occupancy`, `heartbeat` | Every 15 s or on change | 1 | occupancy yes |
| `enclosure` | `water/ph`, `water/do`, `water/ammonia`, `water/salinity`, `water/turbidity`, `water/temp`, `air/temp`, `air/humidity`, `door/state`, `heartbeat` | Every 30 s to 5 min depending on the signal; door on change | 1 | latest value yes |
| `feeder` | `dispense`, `remaining`, `fault`, `heartbeat` | Per dispense; remaining every 15 min | dispense 1, fault 1 | remaining yes |
| `camera` | `inference/count`, `inference/activity`, `inference/queue`, `audit/frame`, `heartbeat` | Count per feeding and hourly; activity every 5 min; queue every 60 s | 1 | latest count yes, frame no |
| `ride` | `vibration/summary`, `cycle`, `anomaly`, `status`, `heartbeat` | Summary every 10 s; cycle per ride cycle; anomaly on change | summary 0, cycle 1, anomaly 1, status 1 | status and anomaly yes |
| `gateway` | `status`, `queue/depth`, `link/state`, `heartbeat` | Every 60 s | 1 | yes |
| `alarm` | `raised`, `acknowledged`, `cleared` | On event | 1 | current alarms yes |
| `config` (cloud to edge) | `rules`, `keys`, `revocations`, `models/{model-id}`, `knowledge/{pack-id}` | On change | 1 | yes |

QoS rule of thumb: anything that changes a record or a count is QoS 1 and consumers deduplicate; high-rate summaries that are re-sent every few seconds are QoS 0 because the next one supersedes the last. QoS 2 is not used; its cost is not worth it when every consumer is idempotent anyway.

## Retained messages and last will

- Retained: the latest value on state topics (`status`, `occupancy`, `remaining`, latest water readings, current alarms, all `config` topics). A tablet, dashboard or restarted consumer sees current state immediately without waiting for the next sample.
- Last will: every device and gateway registers a last-will message on its `status` topic with `{"state":"offline"}`, retained. The broker publishes it if the client disconnects ungracefully. The local rule engine treats a last-will or a missed heartbeat for 5 minutes as a device-silent alarm.
- Heartbeats are QoS 0 and not persisted beyond the retained flag; they exist to detect silence, not to be replayed.

## Payload envelope

Every message is a JSON object with the same envelope. Signal-specific fields go under `value`.

```json
{
  "v": 1,
  "device_id": "E12-kit-01",
  "asset_id": "E12",
  "seq": 104233,
  "ts": "2026-09-09T14:22:51.120Z",
  "signal": "water/ph",
  "value": 7.42,
  "unit": "pH",
  "quality": "good",
  "meta": {"fw": "2.3.1", "calibrated_at": "2026-08-30"}
}
```

Schema sketch:

| Field | Type | Required | Meaning |
|---|---|---|---|
| `v` | integer | yes | Envelope version. Consumers reject unknown major versions |
| `device_id` | string | yes | The physical publisher. One asset may have several devices |
| `asset_id` | string | yes | The estate asset the reading belongs to |
| `seq` | integer | yes | Monotonic per device, persisted across restarts. Dedupe key with `device_id` |
| `ts` | RFC 3339 string | yes | Time at the device. Gateways add `gw_ts` if the device clock is untrusted |
| `signal` | string | yes | Mirrors the topic so a payload is self-describing when logged |
| `value` | number, string, boolean or object | yes | The reading. Objects for compound signals such as a count with confidence: `{"count": 143, "low": 138, "high": 149, "model": "fishcount-v7"}` |
| `unit` | string | for numeric values | SI or the domain unit |
| `quality` | `good`, `suspect`, `bad`, `calibrating` | yes | Set by the device or gateway; suspect values are stored but not alarmed on |
| `meta` | object | no | Firmware, calibration, model version and similar |

Compound signals such as ticket scans and audit frames extend `value` with their own fields; the scan payload is defined in [signed-ticket-format.md](signed-ticket-format.md). Audit frames carry a reference to an object stored on the gateway plus a blurred JPEG under a size cap, not raw video.

## Cloud bridge

The zone bridge forwards topics unchanged and adds two things:

| Bridge topic | Direction | Purpose |
|---|---|---|
| `vde/{site}/{zone}/#` | Edge to cloud | All zone traffic, QoS preserved, from the store-and-forward queue in order |
| `vde/{site}/{zone}/config/#` | Cloud to edge | Rules, keys, revocations, model artifacts, knowledge packs; retained so an offline gateway catches up |
| `vde/{site}/{zone}/gateway/{gw-id}/ack` | Cloud to edge | Acknowledgement of replayed batches so the queue can drain |

Authentication is per-gateway mTLS to the cloud hub and per-device certificates to the zone broker, with ACLs restricting each device to its own `asset-id` prefix and each gateway to its own zone prefix.

## Why not Sparkplug B by default

Sparkplug B gives a standard birth and death certificate model, a compact protobuf encoding and a built-in sequence number, and it is well supported by industrial brokers. It was not chosen as the default because its topic namespace is fixed around group, edge node and device and does not express the estate's zone and asset vocabulary naturally, its metric-centric payload is awkward for compound values such as counts with confidence intervals and ticket scans, and the JSON envelope above is readable by every tool the team and the judges will use. If a device vendor ships Sparkplug B only, the gateway translates it into this scheme; the two can coexist behind the bridge.

## Related

- [03-edge-zone.md](../architecture/03-edge-zone.md)
- [04-data-flow.md](../architecture/04-data-flow.md)
- [Signed ticket format](signed-ticket-format.md)
- [ADR-002 MQTT topology and store-and-forward](../adr/ADR-002-mqtt-topology-and-store-and-forward.md)
- [ADR-010 Edge vision, no cloud video](../adr/ADR-010-edge-vision-no-cloud-video.md)
- [AI-1 Piranha counting](../ai/ai-01-piranha-counting.md)
