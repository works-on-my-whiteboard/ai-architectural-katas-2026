# ADR-013: Privacy-preserving footfall and consent-based personalisation

## Status

Accepted, 2026-09-16. Supersedes / Superseded by: none.

## Context

The estate wants to know which parts of the park are popular and to bring visitors back with personalised offers. The cheapest way to count people is to sniff the WiFi and Bluetooth identifiers of their phones; the most effective way to personalise is to track everything a guest does. Both create legal exposure, reputational risk and a data store that is attractive to attackers. Security and privacy is an invariant of the base architecture: it does not participate in the ranking and is not traded against cost or convenience ([05-characteristics](../architecture/05-characteristics.md)).

Forces:

- Popularity analytics need counts and flows, not identities.
- Personalisation needs some individual history, which must be lawful and expected.
- Guests bring children; family passes make this explicit.
- The patchy WiFi makes phone-based sensing unreliable anyway.

### Alternatives considered

| Option | Summary | Why not (or why partially) |
|---|---|---|
| WiFi and Bluetooth probe sniffing | Passive collection of device identifiers at each attraction | Identifies individuals without consent, unreliable with randomised addresses, legally questionable in several jurisdictions |
| App location tracking for everyone | Continuous GPS from the guest app | Requires the app, drains batteries, tracks people who only wanted a ticket |
| Anonymous counters plus consented app data | Chosen | Counts from hardware that cannot identify anyone; personalisation only for guests who opt in |

## Decision

Footfall and occupancy come from anonymous sources: gate scan events (a ticket ID, not a person), time-of-flight or thermal people counters at attraction entrances, and edge-blurred queue cameras that emit counts only. None of these can identify a guest.

Personalisation uses only data the guest has consented to in the app: visit history tied to their account, favourites, and app interactions. Consent is granular (analytics, personalised offers, notifications) and revocable; revocation deletes the personal profile and leaves only aggregate counts.

Data minimisation applies throughout: tickets carry no names; the guest guide redacts personal details before any model call; traces are redacted; retention periods are set per data class and enforced automatically.

## Consequences

### Positive

- Popularity analytics work for every visitor, app or not, and never identify anyone.
- The estate can explain its data practice in one sentence to a guest or a regulator.
- The breach blast radius is small because there is little identifying data to steal.

### Negative

- Individual journeys through the park are not reconstructable, so some flow analytics are coarser.
- Personalisation reach is limited to consenting app users, which slows the returning-visitor programme.
- Counters and blurring hardware cost more than a sniffer.

### Trade-off analysis

| Quality attribute | Effect | Mitigation |
|---|---|---|
| Privacy | Strongly improved | Anonymous sensing, granular consent, minimisation |
| Analytical richness | Reduced at the individual level | Zone-to-zone flow estimated from aggregate counts and timings |
| Reach of personalisation | Limited to opted-in guests | Clear value exchange in the app; family pass holders prompted at purchase |
| Cost | Counters and edge blurring | Shared with queue estimation hardware |

## Related

- [ADR-010](ADR-010-edge-vision-no-cloud-video.md), [ADR-012](ADR-012-llm-observability-and-kill-switches.md)
- [Crowd flow](../ai/ai-03-crowd-flow-and-staffing.md), [Retention and revenue](../ai/ai-05-retention-and-revenue.md), [Guest guide](../ai/ai-04-guest-guide.md)
