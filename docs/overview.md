# Overview

The Von Digitalis Estates need to sell more tickets, keep 200 exotic animals healthy, and learn where their 5,000 daily visitors actually go, on a site where WiFi is unreliable. We designed an edge-first, event-driven platform: MQTT devices and zone gateways keep gates, alarms and keeper tools working when the cloud is unreachable, and a cloud tier owns ticketing, analytics and AI. AI is used where it changes an outcome: on-device vision counts the piranhas, anomaly models flag sick or under-fed animals before a vet bill arrives, forecasting places staff where crowds will be, and a grounded guest guide turns a first visit into a return visit. Every generative model is reached by naming a capability that configuration resolves to an entry in a single model catalogue, backed by a priced model registry, a portfolio ordered by failure domain and evaluation-gated promotion, so a better model, a price rise or a vendor shutdown is a configuration change, not a rewrite. Every AI output is measured before release and monitored in production, with a human in the loop wherever an animal, a ride or a price is involved.

## How to read this repository

1. [README.md](../README.md) for the map and the judges' criteria index.
2. [Architecture](architecture/05-characteristics.md) for the ranked characteristics every addition must respect, then the context and container views.
3. [AI overview](ai/00-ai-overview.md) and the seven targeted use cases.
4. [Dealing with uncertainty in AI](uncertainty.md) for models, prices and vendors.
5. [Validation and verification](validation.md) for how we know it works.
6. [ADR index](adr/README.md) for every decision and its trade-offs.
7. [Implementation notes](implementation/model-access-config.md) if you want to see the shapes of the configuration.

## How AI helps the company

- Fewer vet bills: welfare anomalies are flagged days earlier from feeding, water and activity data, and the keeper morning brief puts them in plain language with the evidence attached.
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

## How the team used AI tooling to produce this submission

We used AI assistants to draft and red-team ADRs, to generate first-pass Mermaid diagrams from written descriptions, and to build the token cost model with current list prices. Every decision, trade-off and diagram was reviewed and edited by the team, and several AI-suggested options were rejected on the way to the ones recorded here. We treated the assistant the way this architecture treats its models: useful, measured, and never the final authority.
