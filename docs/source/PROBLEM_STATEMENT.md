# Von Digitalis Estates: Problem Statement

Architectural Katas 2026: AI-Assisted Software Architecture

## 1. Background

After a bizarre gardening accident, the 204th in line to the Von Digitalis estates has become the 72nd Countess Von Digitalis. The family business of making highly explosive garden gnomes is no longer viable. The estates are large, sprawling, and unprofitable. The Countess wants digital solutions to monetise them. If visitor numbers do not grow, the family may be forced to sell the carnivorous plant collection or return to the gnome business.

## 2. The estate

| Asset | Detail |
|---|---|
| Amusement park | 40 rides from an 18th century, historically important collection. Recently passed safety inspection after asbestos, broken glass, and garden gnomes were removed. |
| Exotic animal collection | Over 200 exotic and poisonous animals, aquatic and land based, across 55 displays and enclosures. Previously private, now opening to the public. Includes a jumping piranha collection. |
| Visitors | 5,000 a day today. Target of at least 15,000 a day within three years. |

## 3. Stated requirements

These come directly from the brief.

### Functional
- F1. Sell tickets for estate access, including family passes.
- F2. Understand which parts of the park are popular, to guide investment and staff deployment.
- F3. Track animal health.
- F4. Track how much and how well animals eat.
- F5. Track population levels of the jumping piranha collection.
- F6. Grow visitor numbers from 5,000 to 15,000 a day within three years.
- F7. Increase the number of returning visitors.
- F8. Make the estates more profitable.
- F9. Reduce the cost of animal care; sick animals are expensive.

### Constraints
- C1. Wifi coverage across the park is patchy.
- C2. Cloud services may be used, but the architecture must show how data gets from the estate to the cloud.
- C3. A budget exists for MQTT-capable hardware devices installed throughout the park.

### Business challenges (as stated by the Countess)
- No real idea which parts of the estate are most popular, so investment and staffing are guesswork.
- Animal care is costly, more so when animals get sick. Healthy, happy animals are the goal.
- Returning visitors are wanted, but there is no idea how to get them.

### Mandated focus
- AI must be central to the solution, for both the estate and its visitors.

## 4. Inferred requirements

These are not in the brief. Each is justified by the stated facts and must be labelled as inferred in the deliverables. Items marked (scope) are candidates for this submission; the rest are documented and deferred.

### Rides and physical assets
- I1. Ride telemetry and predictive maintenance. 18th century mechanisms that just scraped through inspection are a safety and downtime risk. (scope)
- I2. Ride availability status for staff and visitors.
- I3. Queue length and wait time estimation per ride. Feeds F2, staffing, and visitor experience. (scope)
- I4. Capacity and throughput limits per ride and enclosure at 3x current load.

### Ticketing and access
- I5. Offline ticket validation at gates and rides when wifi drops. Follows from C1 and F1. (scope)
- I6. Fraud and duplicate-use prevention.
- I7. Payments, refunds, and PCI scope.
- I8. Ticket types beyond family: single day, season, timed entry, member.
- I9. Daily capacity caps so growth to 15,000 does not overwhelm the park.
- I10. Zone access and re-entry; the animal collection may be separately ticketed.

### Visitor analytics
- I11. Footfall and dwell time per zone, anonymised. This is how F2 is actually answered. (scope)
- I12. Privacy and consent for any visitor tracking, with opt-in for personalisation.
- I13. Correlation of popularity with weather, day, events, and pricing.

### Animal welfare
- I14. Enclosure environment monitoring: water quality, temperature, humidity, light. Prerequisite for F3. (scope)
- I15. Escape and containment monitoring for venomous and poisonous species. Public safety.
- I16. Feeding schedules and food stock management. Supports F4.
- I17. Vet records, medication, and incident logging.
- I18. Alerting and escalation to keepers with human confirmation before action. (scope)
- I19. Regulatory reporting for exotic animal licences.

### Staff operations
- I20. Staff deployment recommendations driven by popularity data. Direct answer to F2.
- I21. Incident and emergency handling: ride fault, animal escape, medical, lost child.
- I22. Staff mobile tooling that works offline.

### Growth and revenue
- I23. Marketing, CRM, and loyalty for returning visitors. Answers F7. (scope)
- I24. Dynamic or seasonal pricing. Answers F8.
- I25. Secondary revenue: food, retail, events, memberships, sponsorship.
- I26. Guest engagement: personalised itineraries, concierge, wayfinding. (scope)

### Platform and non-functional
- I27. Edge computing with store-and-forward buffering. Follows from C1 and C2. (scope)
- I28. MQTT topic design, device provisioning, firmware updates, device security. Follows from C3.
- I29. Data retention and lifecycle for telemetry.
- I30. Bounded cloud and AI spend. This is a small estate, not a hyperscaler.
- I31. Differentiated availability targets: gates and safety systems high, analytics lower.
- I32. Evolvability, since AI providers and models will change.
- I33. Observability across edge and cloud.
- I34. Network segregation: enclosure controls must not be reachable from guest wifi.

### AI specific (implied by the judging criteria)
- I35. Explicit statement of where AI is deliberately not used.
- I36. Human-in-the-loop for high-stakes decisions: animal health, pricing, safety.
- I37. Eval datasets, offline golden tests, online drift and quality monitoring.
- I38. Fallback behaviour when a model or provider degrades or fails.
- I39. Per-use-case cost monitoring for AI.

## 5. Deliverables

Submitted as a GitHub repo with a README the judges can navigate. Judges cannot speak to the team; the deliverables are the only channel.

1. Overview: a short narrative of how the team used AI to solve the estate's problems.
2. Diagrams: comprehensive and targeted views for each AI use case. Simple boxes, arrows, and words. Provide a key if shapes carry meaning.
3. ADRs: one per AI-related implementation, including trade-off analysis. Format: title, status, context, decision, consequences.
4. Optional: pertinent implementation details.
5. Semi-finalists only: a five-minute video describing the team's approach.

## 6. Judging criteria

- Innovative use of AI in the solution.
- Suitability of the solution given the constraints.
- Appropriate level of detail.
- Dealing with uncertainty in AI: model quality shifting, provider price changes, provider shutdown.
- Whether the architectural characteristics of the additions match the existing architecture.
- Validation and verification of AI results, given that GenAI is non-deterministic. How misbehaviour is detected in production.

## 7. Schedule

| Milestone | Date |
|---|---|
| Solutions due in GitHub repo | 11:59pm ET, Wednesday 16 September 2026 |
| Semifinalists announced | Monday 5 October 2026 |
| Semifinalist videos due | 11:59pm ET, Monday 12 October 2026 |
| Winners announced | Wednesday 21 October 2026 |

## 8. Scope decision

The full requirements landscape above is larger than one submission can design well. Items marked (scope) form the target for this submission. Everything else is acknowledged and deferred, with the reason recorded in the README. Demonstrating judgment about what not to build is part of the assessment.
