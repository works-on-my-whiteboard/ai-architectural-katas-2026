# ADR-010: Edge computer vision with no cloud video

## Status

Accepted, 2026-09-16. Supersedes / Superseded by: none.

## Context

Counting the jumping piranha population, estimating queue lengths and measuring animal activity all benefit from cameras. Video is the heaviest data the estate will produce, the backhaul is unreliable, and cameras in public areas raise privacy questions. Privacy is an architectural invariant.

Forces:

- A single tank camera at modest resolution produces more data per day than every other sensor in the zone combined.
- Counting must keep working when the backhaul is down.
- Guests in the background of a queue camera have not consented to cloud processing.
- Models improve and must be updateable in the field.

### Alternatives considered

| Option | Summary | Why not (or why partially) |
|---|---|---|
| Stream video to a cloud vision service | Cameras push RTSP or clips to the cloud; a hosted model counts | Bandwidth the estate does not have, counts stop during outages, guest footage leaves the site, and a vendor owns the capability |
| Periodic still uploads | Snapshots every few minutes to the cloud | Cheaper than video but still cloud-dependent and misses feeding-time bursts when fish cluster |
| Inference on the zone gateway, results only to the cloud | Chosen | Bandwidth, availability and privacy solved; models are the estate's own |

## Decision

Cameras connect to the zone gateway over the wired zone network. The gateway's GPU-class module runs owned detection and tracking models (ONNX, YOLO-class). Only inference results leave the zone: counts with confidence intervals, activity indices, queue estimates, plus a small number of sampled frames for audit, with people-containing frames blurred at the edge before sampling.

Video is retained on the gateway in a short rolling buffer for keeper review and model audit, then overwritten. No video is stored in the cloud.

Models are versioned in the estate's own model registry and pushed to gateways over MQTT command topics with a signed manifest; a gateway keeps the previous version and rolls back if the new model's confidence distribution shifts abnormally.

## Consequences

### Positive

- Counting and activity monitoring continue through any partition.
- Backhaul carries kilobytes of results instead of gigabytes of video.
- Guest footage never leaves the estate; the privacy story is simple to explain.
- The vision capability is immune to vendor changes.

### Negative

- Each vision zone needs a GPU-class module, which is the most expensive edge component.
- Model updates and rollbacks in the field are harder than a cloud deploy.
- Training data must be collected and labelled from the estate's own tanks and queues.

### Trade-off analysis

| Quality attribute | Effect | Mitigation |
|---|---|---|
| Bandwidth | Reduced by orders of magnitude | Results and sampled frames only |
| Availability | Local inference | Rolling buffer for catch-up counts after a gateway restart |
| Privacy | Strong | Edge blurring, no cloud video, short retention |
| Cost | GPU module per vision zone | Only aquatic and queue zones get GPU modules |
| Operability | Field model updates | Signed manifests, automatic rollback |

## Related

- [ADR-001](ADR-001-edge-first-hybrid-architecture.md), [ADR-002](ADR-002-mqtt-topology-and-store-and-forward.md), [ADR-004](ADR-004-classic-ml-vs-genai-selection.md), [ADR-013](ADR-013-privacy-preserving-footfall-and-consent.md)
- [Piranha counting](../ai/ai-01-piranha-counting.md), [Edge zone](../architecture/03-edge-zone.md)
