# ADR-016: Local, role-based incident response for venomous containment and keeper safety

## Status

Accepted, 2026-09-16. Supersedes / Superseded by: none.

## Context

The estate holds over 200 exotic and poisonous animals across 55 displays and enclosures. The collection was private; it is now opening to the public and the plan is to put 15,000 people a day alongside it. Two events on this estate have a margin measured in minutes rather than hours: a venomous animal out of containment, and a keeper envenomated inside an enclosure.

[ADR-001](ADR-001-edge-first-hybrid-architecture.md) puts deterministic operational welfare rules on the zone gateway. Containment is different: a venomous enclosure door or breach contact must sound directly through the alarm panel, even when that gateway has failed. [ADR-011](ADR-011-human-in-the-loop.md) keeps AI out of any action on an animal. None of that answers the questions an inspector will ask first: **who responds, how does a keeper who cannot use a phone raise an alarm, and how is a zone isolated without trapping the guests inside it?**

Forces:

- The person who most needs to raise the alarm may be envenomated, one-handed, or on the floor of an enclosure. A phone in a pocket is not a control.
- WiFi is patchy by the brief's own statement, so a response path that assumes connectivity fails precisely when it is needed.
- Guests are now inside the collection. Stopping movement into a zone is necessary; stopping movement out of it is illegal and lethal.
- Automatic containment that locks doors could trap a keeper or a guest, and would have to be certified as a safety function.
- An escape alarm that fires on every routine door-open is an alarm nobody answers.
- Welfare and dangerous-species licensing are inspected regimes. Accountability must sit with a named person and be evidenced.

### Alternatives considered

| Option | Summary | Why not (or why partially) |
|---|---|---|
| App and cloud incident management | Alarms and response coordinated through the staff app and a cloud incident service | Depends on WiFi, a charged phone and a reachable cloud. Fails in the exact conditions it exists for |
| Manual procedure only | Paper procedure, radio, no instrumentation | Robust, and it is the floor beneath everything below. But it detects nothing, times nothing and evidences nothing |
| Automated containment | Breach detection drives door interlocks and zone lockdown | Certification burden of a safety function, and an unacceptable risk of trapping a person. Rejected outright |
| Deterministic local detection, hard-wired alerting, human radio-led response, software as recorder only | Chosen | Slower than automation and needs drilled people and dedicated wiring, but it is the only option that holds with the network, the cloud, the gateway and the AI all absent |

## Decision

**The life-safety path contains no software the estate wrote, and no network the estate shares.**

1. **Containment detection and alerting are directly hard-wired.** Door and breach contacts drive the local alarm panel, which drives the sounder, beacon, on-zone pagers and radio dispatch. A safety-owned local maintenance bypass is the only way to suppress a planned door-open; it is not gateway configuration. No cloud round trip, model, inference, broker or estate-written software is on this path.

2. **Duress is hard-wired and independent of the gateway.** Every venomous enclosure and keeper work area has a fixed panic station wired directly to the local alarm panel and to estate radio dispatch. It works with the gateway powered off, the broker stopped and the backhaul cut. It is reachable at floor level. This is the control for the keeper who cannot operate a device, and it is deliberately not a button in an app.

3. **Response is human and role-based.** On-duty named roles — keeper lead, qualified first-aid responder, incident commander — are dispatched by radio, which is the estate's most dependable channel. Escalation to the veterinary holder of antivenom and to emergency services follows a documented, drilled procedure. The incident commander, a person, declares the all-clear.

4. **Isolation stops entry, never egress.** Declaring a zone isolated closes public entry routes and halts new admissions at the affected gates. Egress remains fail-safe open under the certified access-control and life-safety system, which is not controlled by the zone gateway, the cloud or any estate-written service. A software fault cannot lock a guest or a keeper in.

5. **Estate-written software's role is bounded to record and inform.** It receives a non-blocking copy of a panel event, stores it when possible, and presents context to staff. It never actuates containment, gates the response, suppresses an alarm, or becomes the only path — the sounder and panic station are wired, and the radio procedure stands if every computer on the estate is dead.

6. **AI has no role in this path.** Not in detection, not in triage, not in escalation, not in the all-clear. This is recorded here so that the boundary is a decision rather than an omission, and it is restated in [the AI overview](../ai/00-ai-overview.md#where-ai-is-deliberately-not-used).

7. **Every incident produces an append-only record.** When the gateway is available it captures what fired, when, who acknowledged, elapsed times, actions taken, outcome and all-clear authority. When it is not, the incident commander opens the controlled radio/paper log and enters it later as a reconstructed record, explicitly declaring missing telemetry. It feeds the welfare record ([ai-02](../ai/ai-02-welfare-anomaly-and-brief.md)) and dangerous-species licence reporting, and it is the evidence base for tuning thresholds.

8. **Alarm discipline is part of the design.** An authorised, local alarm-panel bypass covers a planned maintenance door-open; it is not gateway configuration and expires under the safety procedure. A containment contact whose false-alarm rate breaches its agreed ceiling is a defect investigated through the safety-change process, not silenced locally.

Threshold values, antivenom stock and location, responder rosters, radio dispatch procedure, and lone-worker wearable duress devices are owned by the estate's safety, veterinary and HR functions. They are named in [06-safety-case.md](../architecture/06-safety-case.md) as decisions for those owners, with a safe default stated for each; this ADR fixes the architecture, not the clinical or staffing values.

## Consequences

### Positive

- The response works with the cloud, the backhaul, WiFi, the gateway and every AI feature simultaneously unavailable.
- A keeper who cannot reach or operate a device can still raise an alarm.
- Egress is structurally protected from software failure, because software is not in that path.
- Incident timing and acknowledgement become evidence, which is what an inspector and a licensing regime ask for.
- The AI boundary is explicit: no model participates in containment, duress, dispatch or the all-clear.

### Negative

- Dedicated wiring, panic stations, sounders, beacons and pagers are capital cost that the MQTT device budget does not cover.
- The response is as good as the drills. An undrilled procedure is a document, not a control.
- No automatic containment means a breach relies on trained people arriving, which is slower than an interlock would be.
- Radio-first means radio coverage and battery discipline become an operational obligation of their own.

### Trade-off analysis

| Quality attribute | Effect | Mitigation |
|---|---|---|
| Availability under partition | Strongly improved; the path has no network dependency | Hard-wired alerting and radio; quarterly drill with the gateway powered off |
| Safety | Strongly improved, and certifiable because the safety function is not software we wrote | Certified access control retains egress; deterministic rules only |
| Response time | Slower than automated containment | Drilled roles, on-zone pagers, panic stations at floor level |
| Cost transparency | Capital cost outside the device budget, stated rather than absorbed | Costed as safety infrastructure, not as IoT |
| Observability | Improved; incidents are timed and evidenced | Append-only incident record with acknowledgement and elapsed times |
| Data integrity | Improved | Incident records append-only, same discipline as welfare records ([ADR-011](ADR-011-human-in-the-loop.md)) |

## Related

- [06-safety-case.md](../architecture/06-safety-case.md), the layered design view and its failure modes
- [ADR-001](ADR-001-edge-first-hybrid-architecture.md), [ADR-002](ADR-002-mqtt-topology-and-store-and-forward.md), [ADR-011](ADR-011-human-in-the-loop.md)
- [03-edge-zone.md](../architecture/03-edge-zone.md), local alarm rules and the zone network
- [05-characteristics.md](../architecture/05-characteristics.md), availability under partition and security
- [Inferred requirements](../inferred-requirements.md), dispositions for I15, I18 and I21
