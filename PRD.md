# Von Digitalis Estates: Problem Statement and Product Requirements

This document restates the kata brief as a problem statement and turns it into a product requirements document (PRD). It is the "what and why" that the architecture in [README.md](README.md) and [docs/](docs/) answers. Anything in the brief is quoted or paraphrased faithfully; anything we inferred is marked as an assumption.

---

## Part 1: Problem statement

### Background

After a bizarre gardening accident, the 204th in line to the Von Digitalis estates has become the 72nd Countess Von Digitalis. The estates are large and sprawling and need help to become profitable. The family's previous business, making unfortunately highly explosive garden gnomes, is no longer viable. The Countess wants digital solutions that help her monetise parts of the estate.

### What the estate has

| Asset | Detail from the brief |
|---|---|
| Amusement park | An extensive, historically important collection of 18th-century rides, 40 in total. Recently passed safety inspections after asbestos, broken glass and garden gnomes were removed |
| Exotic animal collection | Previously private, now opening to the public. Over 200 exotic and poisonous animals across 55 displays and enclosures, a mix of aquatic and land-based, including a jumping piranha collection |
| Carnivorous plant collection | At risk of being sold if visitor growth targets are missed |
| Visitors | 5,000 a day on average, with the expectation and hope of at least 15,000 a day within three years |

### What the Countess needs

1. A system that lets people buy tickets, including family passes, to access the estates.
2. An understanding of how popular different parts of the park are, so the estate knows where to improve things and where to invest and deploy staff.
3. Careful monitoring of the exotic animal collection: animal health, how much and how well they are eating, and, for the jumping piranha collection, population levels.
4. A way to grow the number of visitors, get more returning visitors, and make the estates more profitable. Otherwise it is back to the garden gnome business.

### Biggest business challenges, in the Countess's words

- "We have no real idea of what parts of the estates are most popular, so it is difficult to know where to invest and deploy staff."
- "Looking after the animals is costly, even more so if they get sick. So we want healthy and happy animals."
- "We want to get more returning visitors too, but are not sure how."

### Technical context and constraints

| Constraint | Detail |
|---|---|
| Connectivity | WiFi coverage on the park is patchy |
| Cloud | Cloud services may be used, but there must be a way of getting information from the estate to the cloud |
| Hardware | Assume a budget for MQTT-capable hardware devices that can be installed throughout the park |

### The ask

Produce a new, comprehensive, modern architecture for the Von Digitalis estates, with a focus on how AI could be used to solve the Countess's problems, both for the company and for its customers.

### How the work will be judged

| Criterion | What it means for this project |
|---|---|
| Innovative use of AI in the solution | AI where it changes an outcome, for the estate and for guests |
| Suitability of the solution given the constraints | Works with patchy WiFi, an estate-to-cloud path, MQTT devices, 3x growth |
| Appropriate levels of detail | Not too much, not too little: enough to build from at several zoom levels, without drowning the judges in detail they do not need |
| Dealing with uncertainty in the world of AI technology | Models change: the best model or provider today may not be the best tomorrow, or may shut down. Money changes: providers reprice, and the estate's own AI budget may shrink or grow. The design must absorb both without a rewrite |
| Validation and verification of AI results | GenAI is non-deterministic; show how you confirm it works and how you detect misbehaviour in production |

### Deliverables and schedule

| Deliverable | Note |
|---|---|
| Overview | A short narrative describing how the team solved the problems of the estates: the conventional architecture (ticketing, edge zones, messaging, data platform) and how AI is used within it. AI is one part of the solution, not the whole of it |
| Diagrams | Comprehensive views of the whole system (context, containers, deployment, data flow) plus a targeted view for each use of AI. Ordinary patterns such as event-driven messaging, store and forward, CQRS and offline-first clients appear alongside AI patterns such as capability-based model selection, RAG and human-in-the-loop |
| ADRs | For any significant decision, AI or not (for example MQTT and zone gateways, offline ticket validation, model access, model portfolio), each with trade-off analysis in the standard form: Title, Status, Context, Decision, Consequences |
| Implementation details | Optional, where pertinent |
| Video | Five minutes, semi-finalists only |
| Repository | A GitHub repo with all documentation and visuals and a simple README for the judges |




---

## Part 2: Product requirements document

### Vision

A digital platform that lets the Von Digitalis estates sell access, understand their visitors, keep their animals healthy, and grow from 5,000 to 15,000 daily visitors profitably, on a site where connectivity cannot be trusted, with AI applied wherever it changes an outcome and never as a single point of failure.

### Goals

| ID | Goal | Measure of success (target within 3 years unless stated) |
|---|---|---|
| G1 | Sell tickets and family passes reliably | 100% of valid tickets admitted at the gate with the cloud link down; zero double admissions |
| G2 | Know what is popular | Occupancy and dwell time per attraction available in 15-minute buckets for every day of operation |
| G3 | Healthy, happy animals at lower cost | Welfare anomalies flagged before a keeper would have noticed; reduction in emergency vet call-outs (baseline to be set in year one) |
| G4 | Reliable piranha population count | Automated count within its stated confidence interval of the monthly manual count |
| G5 | Grow visitors and returning visitors | 15,000 visitors a day; measurable rise in the share of returning visitors and memberships |
| G6 | Improve profitability | Revenue per visitor and staff cost per visitor tracked park-wide and per attraction; AI spend visible per feature |

### Non-goals

- Replacing the ride safety inspection regime with automation. AI is advisory for rides.
- Automating any action on an animal (feeding changes, medication). AI recommends; a keeper decides.
- Building our own foundation models. We consume models from a catalogue by naming a capability, and own only small task-specific models.
- Tracking individual visitors without consent. Popularity is measured anonymously.
- Handling card payments in-house. A payment provider carries PCI scope.
- Deciding to retire, close or restore a ride. The value-per-attraction view informs that decision; the Countess and management make it.

### Personas

| Persona | Who they are | What they need |
|---|---|---|
| Guest | A family or individual visiting for the day, often with a phone on poor signal | Buy tickets easily, get in without fuss, know what to see and how long the queues are, have a reason to come back |
| Gate attendant | Staff at entrance and ride gates | Validate a ticket in under two seconds whether or not the network is up |
| Keeper | Cares for a set of enclosures, works outdoors with a tablet | Know which animals need attention this morning and why, record observations quickly, be alerted when something is wrong |
| Vet | Visits on schedule or on call | A clear history per animal: feeding, environment, observations, previous treatment |
| Operations manager | Runs staffing, queues and incidents for the day | Know where crowds are and will be, place staff, see incidents and their status |
| Ride engineer or inspector | Maintains 18th-century rides | Know which ride to look at first, with the evidence |
| Marketing and commercial lead | Grows visitors and revenue | Segments, campaign performance, pricing suggestions with guardrails, membership uptake |
| The Countess and management | Owns the estate's future | Whether the estate is on track for 15,000 visitors a day and profitability, and what to invest in; which attractions earn their keep and which are popular but cost more than they return |

### Functional requirements

Each requirement has an identifier, a priority (Must, Should, Could) and acceptance criteria. Requirements map to the needs in Part 1.

#### F1. Ticketing and access

| ID | Requirement | Priority | Acceptance criteria |
|---|---|---|---|
| F1.1 | Guests can buy day tickets online and at the gate | Must | Purchase completes in under a minute; ticket delivered to the app, wallet or print |
| F1.2 | Family passes cover a defined group | Must | One purchase issues a group token; each member is admitted individually; group size is enforced |
| F1.3 | Tickets are validated at entrance and ride gates | Must | Validation in under two seconds; works offline; a ticket cannot be used twice for the same admission |
| F1.4 | Refunds, cancellations and revocation | Must | A revoked ticket is refused at the gate within a defined sync window even when the link is intermittent |
| F1.5 | Memberships and annual passes | Should | Renewals and member benefits are supported by the same validation path |
| F1.6 | Timed entry and capacity limits | Could | Sales can be capped per time slot when the park approaches capacity |

#### F2. Popularity and operations

| ID | Requirement | Priority | Acceptance criteria |
|---|---|---|---|
| F2.1 | Measure occupancy per attraction and area | Must | Counts available in 15-minute buckets, derived from gate scans and anonymous counters; popularity is expressed per operating hour so a ride closed for repair is not mistaken for an unpopular one |
| F2.2 | Estimate queue length and dwell time | Should | Queue estimates published to ops and to guests with a stated confidence. Dwell time is reported as queue time and viewing time separately: for a ride, queue time is a cost to the guest; for an enclosure, viewing time is value to the guest. Long queue plus short viewing marks a bottleneck; long viewing plus low occupancy marks a hidden gem |
| F2.3 | Forecast demand | Should | Day-ahead and week-ahead forecasts by area, with forecast error reported against actuals |
| F2.4 | Recommend staffing | Should | A roster suggestion per day with an explanation; managers can accept, edit or reject |
| F2.5 | Heatmaps and investment reporting | Must | Management can see the most and least popular areas over any period |
| F2.6 | Value per attraction | Should | For each ride and enclosure, popularity (riders or visitors per operating hour) is shown against operating cost: maintenance hours, parts, downtime and staff. A ranked list of best and worst return over any period, so a popular ride that costs more than it earns is visible and a cheap, well-liked ride is not overlooked |

#### F3. Animal welfare

| ID | Requirement | Priority | Acceptance criteria |
|---|---|---|---|
| F3.1 | Monitor enclosure environment | Must | Temperature, humidity and water quality (pH, dissolved oxygen, ammonia, salinity, turbidity where relevant) recorded per enclosure |
| F3.2 | Monitor feeding quantity and quality | Must | Food dispensed and leftover recorded per feeding; deviations flagged |
| F3.3 | Record keeper observations and vet records | Must | Keepers can log observations on a tablet offline; records sync when connected |
| F3.4 | Detect welfare anomalies early | Should | Anomalies flagged with the supporting data; keepers can accept or reject each flag; acceptance rate tracked |
| F3.5 | Daily keeper brief | Should | A plain-language morning brief per enclosure, citing the data points behind every statement |
| F3.6 | Safety alarms work offline | Must | Environmental and safety alarms fire on the zone network with the cloud unreachable |

#### F4. Piranha population

| ID | Requirement | Priority | Acceptance criteria |
|---|---|---|---|
| F4.1 | Automated population estimate | Must | An estimate with a confidence interval per feeding; a trend over time |
| F4.2 | Manual reconciliation | Must | Monthly manual count recorded and compared with the estimate; disagreement tracked |
| F4.3 | Video stays on the estate | Must | No raw video leaves the zone; only counts, confidence and a small number of sampled frames |

#### F5. Guest growth and retention

| ID | Requirement | Priority | Acceptance criteria |
|---|---|---|---|
| F5.1 | Guest app with tickets, map and live information | Must | Works with poor signal: tickets, map and core content cached on the device |
| F5.2 | Guest guide | Should | Answers grounded questions about animals, rides, history and accessibility with citations; refuses medical and safety advice safely |
| F5.3 | Personalised itinerary | Should | A plan for the family's ages and time budget; updates when queues change |
| F5.4 | Re-engagement | Should | Post-visit recaps and offers sent only to guests who consented; performance measured |
| F5.5 | Off-peak pricing suggestions | Could | Suggestions with floors and caps; a human approves any change; no per-person price discrimination |
| F5.6 | Feedback capture | Must | Guests can rate guide answers and the visit; ratings feed quality monitoring |

#### F6. Profitability and internal efficiency

| ID | Requirement | Priority | Acceptance criteria |
|---|---|---|---|
| F6.1 | Ride condition advisory | Should | Vibration anomalies per ride baseline surfaced to engineers as a priority list; inspection regime unchanged |
| F6.5 | Maintenance log per ride and enclosure | Should | Work orders, downtime, parts and labour cost recorded against the asset; feeds the cost side of F2.6 and the per-attraction view of G6 |
| F6.2 | Support triage and incident summaries | Could | Support requests categorised and routed; incident summaries drafted for human review |
| F6.3 | Keeper voice notes to records | Could | Spoken observations transcribed into structured records that the keeper confirms |
| F6.4 | AI cost visibility | Must | Spend per AI feature visible daily; budgets and alerts in place |

### Non-functional requirements

These are the ranked architectural characteristics the whole platform, including every AI addition, must honour. The full definitions and fitness functions are in [docs/architecture/05-characteristics.md](docs/architecture/05-characteristics.md).

| Rank | Characteristic | Requirement |
|---|---|---|
| 1 | Availability under partition | Gates, alarms and keeper tools work for at least a full operating day with the cloud unreachable; data is buffered and replayed without loss |
| 2 | Evolvability | Services, models and AI providers can be replaced behind stable contracts without rewriting features |
| 3 | Observability | Every device, service and model call is traced and metered; AI quality signals are monitored in production |
| 4 | Data integrity | No lost or double-counted tickets, payments or animal records; consumers are idempotent |
| 5 | Elastic scalability | The cloud tier scales to 3x current load; the edge scales by adding zones |
| 6 | Cost transparency | Cost per feature, including AI tokens, is visible and capped |
| 7 | Security and privacy | Least privilege on devices; consent before personalisation; no WiFi sniffing; video never leaves the estate |

Additional non-functional requirements:

| Area | Requirement |
|---|---|
| Latency | Gate validation under 2 seconds; guest guide answer under 4 seconds at p95; alarms within 5 seconds locally |
| Growth | Support 15,000 visitors a day with headroom; ticket sales peaks before holidays |
| Privacy and compliance | GDPR-style handling of personal data; data minimisation; retention limits per data class |
| Payment security | PCI scope held by the payment provider; no card data on estate systems |
| AI safety | Human in the loop for any decision affecting an animal, a ride or a price; grounded answers only for guests |
| AI resilience | At least two independent providers per generative capability plus a self-hosted fallback and a non-AI floor; the same degradation path is used when budget, not availability, is the constraint |
| Maintainability | Infrastructure as code; versioned prompts, models and configuration; automated evals in CI |

### Constraints

| Constraint | Source | Effect on the design |
|---|---|---|
| Patchy WiFi | Brief | No critical function depends on WiFi; wired, LoRaWAN and private backhaul for devices |
| Estate-to-cloud path required | Brief | Zone gateways bridge MQTT to the cloud with store-and-forward |
| MQTT device budget | Brief | MQTT is the device protocol for scanners, counters, sensors, feeders and ride sensors |
| Historic rides | Brief | Sensors must be non-invasive; safety remains with inspectors |
| Poisonous animals | Brief | Keeper safety alarms are local and deterministic |
| AI market uncertainty | Judges' criteria | Models are configuration, not code; prices and providers can change without a rewrite |

### Assumptions

| Assumption | Why we made it |
|---|---|
| The estate can install wired or LoRaWAN connectivity to fixed devices and a private backhaul per zone | The brief funds MQTT devices; devices need a path to a gateway |
| Cellular or fibre backhaul is available at least intermittently | The brief allows cloud use and requires an estate-to-cloud path |
| Keepers and gate staff can carry tablets or scanners | Needed for offline observation logging and validation |
| Guests are willing to install an app or use a mobile web app | Needed for guide, itinerary and wallet tickets; printed tickets remain available |
| A payment provider is acceptable | Keeps PCI scope off the estate |
| Maintenance work on rides and enclosures can be logged digitally, even if today it is on paper | Needed to set popularity against cost per attraction (F2.6); without it the estate only knows what is popular, not what is worth it |
| Average ticket revenue makes AI spend a small fraction of income | Used to argue that AI cost is a governance problem, not an existential one |

### Success metrics

| Metric | Baseline | Target |
|---|---|---|
| Daily visitors | 5,000 | 15,000 within three years |
| Returning visitor share | Unknown, to be measured in year one | Rising quarter on quarter |
| Gate validation success with link down | Not possible today | 100% of valid tickets |
| Welfare flags accepted by keepers | Not applicable | Above an agreed precision threshold, tracked monthly |
| Piranha count agreement with manual count | Manual only | Within the stated interval |
| Forecast error | None | Falling month on month after launch |
| AI availability | None | Guest-facing features degrade gracefully with any single provider down |
| AI spend per feature | Unknown | Within budget, with alerts at 120% of plan |
| Attractions with both popularity and cost data | None | All 40 rides and 55 enclosures by end of year one, so every attraction has a return figure |

### Out of scope for this submission

- Detailed vendor selection and commercial negotiation.
- Physical installation plans, cabling routes and radio surveys.
- Food, retail and car parking systems, except where ticketing touches them.
- Detailed UI design for the guest and keeper apps.

### Risks

| Risk | Impact | Mitigation |
|---|---|---|
| Backhaul is worse than assumed | Delayed analytics, stale cloud data | Edge autonomy for everything critical; buffers sized for a full day |
| Model provider changes price or shuts down | Cost spike or outage of GenAI features | Capability contracts in configuration, priced registry, a self-hosted tier outside the catalogue, non-AI fallback |
| The estate's own AI budget is cut | Features must run on less or be switched off | Per-feature budget caps, cheaper model tiers selectable in configuration, every AI feature has a non-AI floor so switching it off degrades rather than breaks |
| Maintenance logs are incomplete or stay on paper | Value per attraction is wrong or missing; a popular ride looks cheaper than it is | Simple work-order capture on the engineer's tablet, offline like the keeper app; cost figures show their data coverage |
| GenAI gives a wrong or unsafe answer to a guest | Reputational harm | Grounded answers with citations, refusal topics, evals before release, monitoring in production, kill switch |
| Welfare model misses or over-flags | Vet cost or keeper fatigue | Human in the loop, accept and reject labels, threshold alarms remain |
| Privacy complaint about tracking | Legal and reputational harm | Anonymous counters, consent-based personalisation, video stays on the estate |
| Visitor growth outpaces capacity | Queues and poor experience | Forecasting, staffing recommendations, timed entry option |

### Glossary

| Term | Meaning |
|---|---|
| Occupancy | How many people are at an attraction or in an area at a given moment, derived from anonymous entry and exit counts. Answers "where are the crowds" and drives staffing and queue estimates |
| Occupancy bucket | The unit in which occupancy is stored and reported: a fixed 15-minute window per attraction and area holding entries, exits, occupancy at the end of the window, peak occupancy within it, and open-or-closed status. Fifteen minutes matches how quickly staff can be moved, smooths the batch nature of ride cycles, and keeps records small enough to buffer on a zone gateway during an outage. Popularity is riders or visitors per operating hour, so closed buckets are excluded rather than counted as zero. Rationale in [ADR-015](docs/adr/ADR-015-occupancy-grain-and-operating-hour-normalisation.md) |
| Dwell time | How long a visitor stays at an attraction, from arriving to leaving. Answers "how engaging is this". Measured anonymously as the average gap between the entry count and exit count rising, so no individual is tracked. For rides it is queue time plus ride duration; for enclosures it is viewing time |
| Zone gateway | The on-estate computer in each park zone that runs the local MQTT broker, buffers data, evaluates alarms, validates tickets and runs edge vision |
| MQTT | Message Queuing Telemetry Transport. A lightweight publish-and-subscribe messaging protocol built for small devices on unreliable networks. A device publishes a short message to a named topic (for example `park/zone3/ride12/vibration`), a broker receives it, and every subscriber to that topic gets a copy; devices never talk to each other directly. Each message carries a delivery guarantee chosen by the sender (at most once, at least once, exactly once), so a ticket scan can be exactly-once while a temperature reading is fire-and-forget. A broker can run on the zone gateway, so devices keep publishing when the cloud link is down. It runs on microcontrollers costing a few dollars, which is why the brief funds it. In this architecture MQTT is the device-to-gateway layer only; cloud services, the guest app and AI features sit above it |
| Estate-to-cloud path | The network link that carries data off the estate grounds to cloud services, commonly called the backhaul. It is a separate concern from the in-park connectivity that carries a reading from a device to its zone gateway: good coverage inside a rural park says nothing about whether there is a usable link out of it, which is why the brief allows cloud services but requires a stated way of getting information from the estate to the cloud. Here the path is private LTE/5G or fibre per zone, and it is assumed intermittent rather than absent. Every zone gateway bridges to the cloud hub through a local disk queue, so a dropped link fills a buffer instead of losing events and the queue drains in order on reconnect, with cloud consumers deduplicating on asset and sequence ([ADR-002](docs/adr/ADR-002-mqtt-topology-and-store-and-forward.md)). Nothing critical waits on it: gate entry validates signed tickets offline ([ADR-003](docs/adr/ADR-003-offline-verifiable-signed-tickets.md)), vision runs on the gateway so the path carries counts rather than video ([ADR-010](docs/adr/ADR-010-edge-vision-no-cloud-video.md)), and gateways buffer 72 hours of telemetry. The link-up and link-down cases are traced in [04-data-flow.md](docs/architecture/04-data-flow.md) |
| Store and forward | Buffering messages locally while the link is down and replaying them when it returns |
| PCI scope | PCI DSS is the Payment Card Industry Data Security Standard, the rulebook the card networks impose on anyone who handles cardholder data; it is a contractual condition of being allowed to accept cards, not a law. Scope is the set of systems that touch that data. A system in scope carries annual assessment, vulnerability scanning, network segmentation, encryption and key management, access logging and documented evidence of all of it, and connected systems are pulled in with it. The estate keeps scope off its own systems: the guest's browser sends card details straight to the payment provider through the provider's hosted form, and the estate receives and stores only an opaque payment reference ([ADR-003](docs/adr/ADR-003-offline-verifiable-signed-tickets.md)). Card data never reaches estate hardware, the cloud tier or any backup, which is a fitness function of the security and privacy characteristic ([05-characteristics.md](docs/architecture/05-characteristics.md)). The provider is then an external dependency with its own failure mode: online sales pause, gate sales fall back to a standalone terminal with deferred settlement, and issued tickets are unaffected because validation is offline ([01-context.md](docs/architecture/01-context.md)) |
| Model access layer | The shared client library every service uses to call a model: it resolves a capability to a model from configuration, applies the guardrail policy and cost tag, meters the call and walks the fallback tiers. Not a service, and not on any critical path of its own |
| Model catalogue | A hyperscaler's managed collection of models behind one API (AWS Bedrock, Google Vertex AI, Azure AI Foundry). The estate's single integration point for generative models |
| Capability | What a feature needs from a model, for example chat.guide, declared instead of a specific model |
| Model registry | The catalogue of models, providers, prices, eval scores and status that drives routing |
| Golden set | A human-labelled dataset used to evaluate an AI capability before release |
| Shadow and canary | Sending a copy of traffic to a candidate model for offline scoring, then a small share of live traffic, before promotion |
| Tier 3 fallback | The non-AI behaviour a feature falls back to when no model is available or budget is exhausted |

---

Related: [README.md](README.md) for the repository map, [docs/overview.md](docs/overview.md) for the narrative, [docs/planning/solutioning.md](docs/planning/solutioning.md) for the analysis of the brief.
