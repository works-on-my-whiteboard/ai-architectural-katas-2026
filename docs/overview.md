# Overview

The Von Digitalis Estates need to sell more tickets, keep 200 exotic animals healthy, and learn where their 5,000 daily visitors actually go, on a site where WiFi is unreliable. We designed an edge-first, event-driven platform: MQTT devices and zone gateways keep gates, operational welfare alerts and keeper tools working when the cloud is unreachable; hard-wired panels independently carry containment and duress; and a cloud tier owns ticketing, analytics and AI. AI is used where it changes an outcome: on-device vision counts the piranhas, anomaly models flag sick or under-fed animals before a vet bill arrives, forecasting places staff where crowds will be, and a grounded guest guide helps a first-time family find its way. Whether better staffing and a better guide actually lift return visits is the estate's commercial bet; this architecture is what makes that bet measurable and reversible rather than what guarantees it. Every generative model is reached by naming a capability that configuration resolves to an entry in a single model catalogue, backed by a priced model registry, a portfolio ordered by failure domain and evaluation-gated promotion, so a better model, a price rise or a vendor shutdown is a configuration change, not a rewrite. Every AI output is measured before release and monitored in production, with a human in the loop wherever an animal, a ride or a price is involved.

## How to read this repository

1. [README.md](../README.md) for the map, the one-view diagram and the submission criteria index.
2. [Walkthroughs](walkthroughs.md) for three scenarios end to end, and the [delivery plan](delivery-plan.md) for what is built first and what unlocks the rest.
3. [Architecture](architecture/05-characteristics.md) for the invariants and ranked characteristics every addition must respect, then the context, container, edge and [cloud deployment](architecture/08-cloud-deployment.md) views.
4. [AI overview](ai/00-ai-overview.md) and the eight targeted use cases.
5. [Dealing with uncertainty in AI](uncertainty.md) for models, prices and vendors.
6. [Validation and verification](validation.md) for how we know it works.
7. [ADR index](adr/README.md) for every decision and its trade-offs.
8. [Implementation notes](implementation/model-access-config.md) and the [fault catalogue](fault-catalogue.md) for configuration shapes and designed failure behaviour.

## How AI helps the company

- Fewer vet bills — **a hypothesis with a test, not a promise.** The claim is that feeding, water and activity data carry a detectable signal before a keeper would notice. Whether that holds for these species is unknown until a season of data exists, so it is measured by blind replay over historical periods containing known incidents, with lead time reported as a distribution. If it does not beat fixed thresholds, the estate keeps the thresholds. The morning brief puts whatever is found into plain language with the evidence attached.
- A reliable piranha census without draining tanks or guessing, with a confidence interval and monthly manual cross-checks.
- Staff where the crowds will be, from a demand forecast and a roster optimiser, explained in a daily ops brief.
- Early warning on 18th-century rides from vibration baselines, prioritising what inspectors look at first without touching the inspection regime.
- Support triage, incident summaries and keeper voice notes turned into structured records, saving staff time.
- AI spend that is visible per feature, capped, and never a single point of failure.

## How AI helps guests

- A guide that answers grounded questions about animals, rides, history and accessibility, and refuses safely on medical and safety topics.
- A personalised itinerary for the family's ages and time budget, with alerts when a favourite ride has no queue.
- Shorter queues overall because staffing follows the forecast.
- Personalised post-visit recaps and fair off-peak offers that make coming back easy.
- Privacy by design: no WiFi sniffing, consent before personalisation, and camera video that never leaves the estate.
