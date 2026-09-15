# ADR-015: Fifteen-minute occupancy buckets normalised per operating hour

## Status

Accepted, 2026-09-10. Supersedes / Superseded by: none.

## Context

Goal G2 and requirement F2.1 ask for occupancy per attraction and area. Every gate scan and every people-counter tick is a timestamped event; the question is at what grain the estate stores, computes and reports them. That grain is used by the heatmaps (F2.5), queue and dwell estimates (F2.2), the demand forecast (F2.3), the staffing recommendation (F2.4) and the value-per-attraction view (F2.6). If each consumer picked its own grain the views would disagree with each other and the forecast would train on a different signal from the one managers see.

Forces:

- Operations managers act on a timescale of roughly a quarter hour: moving a staff member between zones or opening a second gate takes about that long.
- Guests deciding where to walk next want a queue estimate that is fresh but stable, not one that flickers.
- Ride throughput is lumpy. A ride admits a batch of riders per cycle, so counts at a grain shorter than a few cycles are mostly noise.
- WiFi is patchy and zone gateways must keep working and later catch up. Whatever is stored must survive store-and-forward and duplicate delivery (ADR-002).
- Rides are 18th-century and will close for repair. A closed ride produces zero counts, and a naive popularity figure would read that as unpopularity.
- The hot-path target is occupancy on the dashboard within 2 minutes of events arriving (data-flow, characteristics).

### Alternatives considered

| Option | Summary | Why not (or why partially) |
|---|---|---|
| Raw events, aggregate at query time | Store every scan and tick, let dashboards sum on demand | Query cost grows with visitors (3x in three years); every consumer re-derives its own numbers and they drift apart; replaying an outage window is expensive |
| 1-minute buckets | Fine-grained rollup | Ride cycles make counts spike and drop within a minute; a single counter glitch dominates; 15x the rows and bridge traffic for no operational gain |
| 5-minute buckets | Compromise grain | Still noisy for low-throughput rides and enclosures; managers cannot act within 5 minutes anyway; can be produced later by re-bucketing raw events if a specific analysis needs it |
| Hourly buckets | Coarse rollup | Loses the shape of the day: a queue that builds and collapses within 40 minutes disappears; forecast and staffing have too little signal |
| 15-minute buckets, normalised per operating hour | Chosen | Matches the timescale on which people act; smooths ride cycles; small idempotent records; the convention in footfall and retail analytics so figures compare with off-the-shelf tools |

## Decision

Occupancy, entries, exits and queue estimates are computed per attraction and per area in fixed 15-minute buckets aligned to the clock (10:00 to 10:15, 10:15 to 10:30, and so on), in local park time.

Each bucket is a small record keyed by `(asset_id, bucket_start)` holding entries, exits, occupancy at the end of the bucket, peak occupancy within the bucket, an open-or-closed status and a data-coverage flag. Occupancy is a running level maintained on the gateway (previous level plus entries minus exits), not recomputed per bucket; the peak is the highest value that level reached inside the window, so a crowd that arrives and leaves within one bucket is still visible to staffing and safety views even though the end-of-bucket level has recovered. The record is idempotent: a bucket recomputed or re-sent after an outage overwrites the previous value rather than adding to it. Raw scans and ticks are retained for ticketing and for occasional re-bucketing, but no analytics consumer reads them directly.

Buckets are computed on the zone gateway from local MQTT traffic and forwarded to the cloud hub. When the backhaul is down the gateway keeps producing buckets and drains them in order on reconnect (ADR-002). The cloud stream processor recomputes buckets from replayed raw events where it has them and trusts gateway buckets otherwise, so an outage leaves a complete history rather than a gap.

Popularity is expressed per operating hour, not per calendar hour. Each bucket carries the attraction's status from the maintenance log (F6.5): open, closed for repair, closed by schedule, or closed by weather. Popularity for any period is riders or visitors divided by the hours the attraction was actually open. Closed buckets are excluded from the denominator and shown as closed on heatmaps, never as zero.

Dwell time is derived from the same two series and needs no further sensing. No individual is tracked (ADR-013), so the system cannot time any one person's visit. Instead it treats the attraction like a bathtub: entries are the tap, exits are the drain, and the delay between a rise in the entry series and a matching rise in the exit series is how long people stayed. Worked example: if the entry counter jumps by 40 at 11:00 and the exit counter jumps by 40 at 11:05, the estimate for that group is five minutes; if a further 50 enter around 11:03 and 50 leave around 11:20, that group's estimate is about 17 minutes. The bucket reports the average across groups, weighted by group size, together with a confidence that is high when flows are steady and low when they are irregular, because the method cannot tell apart individuals who left early from those who left late within the same group. For rides the estimate is queue time plus ride duration; for enclosures it is viewing time (F2.2).

All downstream views (F2.2 to F2.6) and the demand forecast consume the same buckets. Any other grain is derived from these, never computed independently.

## Consequences

### Positive

- One grain, one set of numbers: heatmap, forecast, roster and value-per-attraction all agree.
- Roughly 40 records per attraction per day instead of thousands of events, so storage, bridge traffic and dashboard queries stay flat as visitors triple.
- Store-and-forward is simple because buckets are small and idempotent; a duplicate bucket is harmless.
- A ride shut for repair is not mistaken for an unpopular one, so investment and retirement decisions (a non-goal for AI, a goal for management) rest on the right figure.
- Fifteen-minute figures are what footfall vendors and retail benchmarks publish, so the estate can compare itself with the outside world.

### Negative

- Events inside a bucket are invisible to analytics except through the peak field. A surge that starts and ends within 10 minutes shows as a raised peak with an unchanged end level, but not its exact timing or duration.
- The forecast and queue estimate cannot resolve finer than 15 minutes without a separate raw-event path.
- Operating-hour normalisation depends on the maintenance log being kept up to date. A closure that is not logged looks like an empty ride.
- Bucket boundaries are arbitrary: a ride cycle straddling 10:15 is split across two buckets.
- Dwell time is a statistical estimate, not a measurement. When groups overlap or leave in a scattered way, the entry-to-exit lag blurs and the confidence drops; the figure is never exact for any one visitor.

### Trade-off analysis

| Quality attribute | Effect | Mitigation |
|---|---|---|
| Timeliness | Bucket completes up to 15 minutes after an event | Partial current bucket is published as it accumulates, marked provisional; hot-path target of 2 minutes still met |
| Accuracy | Sub-bucket dynamics lost | Peak occupancy field captures the worst moment in the window; raw events retained for ad hoc re-bucketing; queue estimate uses counter pairs within the bucket |
| Scalability | Flat with visitor growth | Fixed record count per attraction per day |
| Resilience | Survives partition and duplicates | Idempotent key, gateway-side computation, ordered replay |
| Correctness of popularity | Depends on closure data | Closure status defaults to "unknown" and is flagged in coverage; engineers log closures on the same offline tablet as work orders |

## Related

- [ADR-002](ADR-002-mqtt-topology-and-store-and-forward.md), [ADR-013](ADR-013-privacy-preserving-footfall-and-consent.md), [ADR-011](ADR-011-human-in-the-loop.md)
- [Crowd flow](../ai/ai-03-crowd-flow-and-staffing.md), [Data flow](../architecture/04-data-flow.md), [Characteristics](../architecture/05-characteristics.md)
- PRD: G2, F2.1 to F2.6, F6.5, glossary entry "Occupancy bucket"
