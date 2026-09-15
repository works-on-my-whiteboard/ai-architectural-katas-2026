# Von Digitalis Estates: Brief Analysis and Solutioning


## 1. What the brief is really asking

The slides tell a joke, but the judges' criteria are precise. Read together, the brief is testing six things:

| Judges' criterion | What they are actually checking | Where it must show up |
|---|---|---|
| Innovative use of AI | AI applied where it changes an outcome, not bolted on. A mix of classic ML and GenAI, edge and cloud. | One targeted view + ADR per AI use |
| Suitability given constraints | Patchy WiFi, edge-to-cloud path, MQTT hardware budget, 3x visitor growth | Base architecture, edge design |
| Appropriate level of detail | C4-style zoom levels. Enough to build from, not a vendor catalogue | Diagrams, optional implementation notes |
| Dealing with uncertainty in AI | Best model changes, prices change, provider shuts down | A dedicated "model uncertainty" section + ADRs |
| Characteristics match existing architecture | You must first *define* the base architecture's characteristics, then prove every AI addition conforms | Explicit conformance table |
| Validation and verification of AI results | Non-deterministic output; how you know it works pre-prod and how you detect misbehaviour in prod | Eval + observability section |

Two things are easy to miss:

1. **"Existing architecture" does not exist in the brief.** The kata expects you to design the core (non-AI) system first, name its architectural characteristics, and then show the AI additions inherit them. If the core is offline-tolerant and event-driven, an AI feature that needs a synchronous cloud round trip is a mismatch.
2. **"Overview: how the team used AI to solve the problems"** is ambiguous. It probably means AI in the solution, but a short paragraph on how the team itself used AI tooling (drafting ADRs, generating diagrams, red-teaming the design) costs nothing and hits the "innovative use" criterion for the company *and* its customers.

Required deliverables: short overview narrative, comprehensive and targeted diagrams for each AI use, ADRs (Title / Status / Context / Decision / Consequences with trade-offs), optional implementation details, a navigable README.

---

## 2. Requirements matrix

### Business context

| Fact | Value | Implication |
|---|---|---|
| Visitors today | 5,000 / day | Modest scale; correctness and resilience matter more than raw throughput |
| Visitors target | 15,000 / day within 3 years | Design for 3x with headroom; elastic cloud, fixed-cost edge |
| Rides | 40, 18th century, recently passed inspection | Safety is deterministic and human-owned; AI is advisory only |
| Animals | 200+ across 55 displays and enclosures, aquatic and land, some poisonous | Per-enclosure sensing; keeper safety; vet costs are the pain point |
| Piranha collection | Population count required | Computer vision counting problem |
| Failure mode | Sell the carnivorous plant collection | Revenue growth and cost control are both first-class goals |

### Functional needs

| ID | Need | Notes |
|---|---|---|
| F1 | Sell tickets including family passes | Online, at gate, offline validation at entry |
| F2 | Understand popularity of park areas | Occupancy, dwell time, queue length per attraction; staffing decisions |
| F3 | Monitor animal health, feeding quantity and quality | Sensors, feeders, cameras, keeper observations, vet records |
| F4 | Check piranha population levels | Automated counts with confidence, cross-checked by humans |
| F5 | Grow visitors and returning visitors | Engagement, personalisation, memberships, re-engagement |
| F6 | Make the estate more profitable | Pricing, staffing, energy, vet cost avoidance, upsell |

### Constraints

| ID | Constraint | Design response |
|---|---|---|
| C1 | Patchy WiFi | Nothing critical depends on WiFi. Wired or LoRaWAN for devices, private backhaul for gateways, offline-first apps |
| C2 | Cloud allowed, but need an estate-to-cloud path | Edge gateways with store-and-forward, MQTT bridging over cellular or fibre |
| C3 | Budget exists for MQTT-capable devices | MQTT is the device protocol; topic schema and QoS are design decisions |

### Implicit requirements the judges will look for

- Privacy of visitors (no MAC-address sniffing for footfall; consent-based personalisation; GDPR-style data handling).
- Animal and staff safety alarms that work with the cloud link down.
- Payment security (PCI scope pushed to a payment provider).
- Cost transparency for AI spend.
- Evolvability: models and vendors swapped without code changes.

---

## 3. Base architecture (the thing AI must conform to)

### 3.1 Options considered

| Option | Description | Fit to constraints | Verdict |
|---|---|---|---|
| A. Cloud-centric | Thin devices publish straight to a cloud IoT hub; all logic in cloud | Fails on patchy connectivity: gates, alarms and keeper tools stop when the link drops | Rejected |
| B. Edge-heavy | Everything runs on-premise; cloud is backup only | Robust, but expensive to scale 3x, hard to evolve, poor analytics and AI access | Rejected |
| C. Edge-first hybrid, event-driven | Zone gateways run local safety, validation and buffering; cloud owns analytics, ticketing, AI, and long-term data | Meets every constraint; each side degrades gracefully without the other | **Recommended** |

### 3.2 Architectural characteristics (rank these and put them in the README)

1. **Availability under partition.** Gates, alarms and keeper tools work with the cloud unreachable.
2. **Evolvability.** Services, models and vendors are replaceable behind stable contracts.
3. **Observability.** Every device, service and model call is traced and metered.
4. **Data integrity.** Tickets, payments and animal records are never lost or double-counted.
5. **Elastic scalability.** Cloud tier scales to 3x; edge tier scales by adding zones.
6. **Cost transparency.** Per-feature cost is visible, including AI tokens.
7. **Security and privacy.** Least privilege on devices, consent-driven personal data.

These become the conformance test for every AI addition (section 6).

### 3.3 Edge tier

- **Zones.** Split the estate into 6 to 8 zones (entrance, rides north/south, aquatic house, terrestrial house, gardens and plants, back-of-house). Each zone gets a ruggedised gateway: industrial PC, local MQTT broker (Mosquitto or EMQX Edge), local time-series buffer, rule engine for alarms, ticket validation cache. Aquatic and CV zones get a GPU-class module (Jetson-class) for on-device vision.
- **Devices (MQTT).** Ride gate scanners with QR readers, anonymous people counters (time-of-flight or thermal, not WiFi sniffing), enclosure sensor kits (temperature, humidity, water pH, dissolved oxygen, ammonia, salinity, turbidity), smart feeders with load cells, cameras (video stays local; only inference results and sampled frames leave the zone), ride vibration and cycle-count sensors.
- **Connectivity.** PoE or Ethernet to fixed devices, LoRaWAN for low-rate sensors, private LTE/5G or point-to-point radio for gateway backhaul. Visitor WiFi is for guests only and never on the critical path.
- **Store and forward.** Persistent MQTT sessions, retained last-known values, bridge to cloud with local queue. Alarms evaluated locally and delivered to keeper devices over the zone network.
- **Offline ticketing.** Tickets are signed QR tokens (Ed25519) carrying ticket ID, type, validity window and family group size. Gates verify signatures offline and hold a small synced revocation list; scans are deduplicated on reconnect. A family pass is a group token whose members scan individually.


### 3.4 Cloud tier

- **Ingestion.** Cloud MQTT/IoT hub (AWS IoT Core, Azure IoT Hub, or EMQX Cloud) bridged from zone gateways.
- **Event backbone.** Kafka or a managed equivalent. Stream processing (Flink or equivalent) for hot-path metrics and alerts. Data lake in open table format (Iceberg/Delta) plus a time-series store and an analytics warehouse.
- **Domain services** (containers on Kubernetes or serverless, each owning its data):
  - Ticketing and Payments (payment provider handles PCI).
  - Guest Identity, CRM and Consent.
  - Park Operations and Occupancy.
  - Animal Welfare (records, feeding, vet, alerts).
  - Guest Engagement (app backend, notifications, recommendations).
  - Analytics and BI.
  - **Model access layer and model catalogue** (section 5).
- **Applications.** Guest app (offline-first PWA or native, tickets in wallet), keeper tablet app (offline-first), operations dashboard, gate firmware.
- **Cross-cutting.** Identity and access, OpenTelemetry tracing, feature flags, secrets, infrastructure as code, CI/CD.

---

## 4. AI use cases

### How we choose the AI type (candidate ADR)

- **Numbers or images in, a number or a class out** → classic ML or plain rules.
- **Language in or language out** → GenAI.

Classic models are artifacts we own, so vendor churn cannot break them. Every GenAI call names a capability that configuration resolves to a model (section 5), and every GenAI feature has a non-AI fallback.

### The use cases at a glance

| ID | Use case | AI type | Runs where | Fallback if AI unavailable |
|---|---|---|---|---|
| AI-1 | Piranha and aquatic population counting | Computer vision, owned | Edge (aquatic zone GPU) | Last confirmed count; monthly manual count |
| AI-2 | Welfare anomalies and keeper daily brief | Anomaly detection, owned + GenAI | Edge for alarms, cloud for briefs | Threshold alarms; templated report |
| AI-3 | Crowd flow, demand and staffing | Forecasting and optimisation, owned + GenAI | Cloud | Heatmap dashboard, manual rostering |
| AI-4 | Guest guide and personalised itinerary | GenAI (RAG) | Cloud, content cached in app | Keyword FAQ search, static map |
| AI-5 | Retention, pricing and re-engagement | Classic ML + GenAI | Cloud, batch | Segment rules, fixed pricing |
| AI-6 | Ride condition monitoring | Anomaly detection, owned. Advisory only | Edge | Scheduled inspections (unchanged) |
| AI-7 | Staff copilots: support triage, incident summaries, voice notes | GenAI | Cloud, batch or interactive | Manual process |

GenAI appears in AI-2, AI-3, AI-4, AI-5 and AI-7, and in every one of them it only writes or reads language. The numbers underneath it always come from a model we own.

### AI-1 Piranha counting (innovation showcase)

- **How it works.** Fixed cameras at several angles per tank. A small detection model (YOLO-class) plus multi-object tracking counts continuously, with the headline count taken at feeding time when the fish cluster.
- **What it outputs.** A count with a confidence interval, never a bare single number.
- **How we know it works.** Keepers count manually once a month. We track precision and recall, and alert on drift when the interval widens or the gap to the manual count crosses a threshold.
- **Privacy.** Video never leaves the zone. Only counts, confidence and a few sampled frames for audit.

### AI-2 Welfare anomalies and daily brief

Vet cost is the pain point the brief names (F3), and it scales with how late a problem is found. The aim is a shorter list of the right things to look at, earlier in the day.

- **Inputs.** Per-enclosure sensors (temperature, humidity, and water chemistry on aquatic displays), feeder load-cell deltas of dispensed versus leftover, a camera activity index computed on the gateway (movement, never identity), keeper observations, vet records.
- **Two tiers.** Safety-critical conditions (water chemistry out of range, feeder jammed, sensor silent) are plain rules on the zone gateway, so alarms fire with the cloud unreachable. Learned anomaly detection runs in the cloud, where the history lives.
- **Models.** One anomaly model per species group, each learning its own enclosure's baseline rather than a species textbook. Start with statistical bands and seasonal decomposition, since the cycles are strongly daily; move to isolation forest only where the simple bands prove noisy.
- **The brief.** Overnight, GenAI turns each enclosure's structured anomalies into a plain-language morning brief, requested as a capability rather than as a named model. Every claim cites the data point behind it, and a claim that cannot is dropped. Batch and off-peak, so it is cheap and tolerates a slow link.
- **The loop.** Keepers accept or reject each flagged item. Those verdicts retrain the models and score the brief, and a falling accept rate is the early warning that something has drifted. Alert volume per keeper per day is tracked as hard as precision, because alert fatigue is the realistic failure mode.
- **Fallback.** Link down, alarms continue locally and the brief waits. GenAI down or over budget, a templated numeric report of anomalies, values and baselines.
- **The limit.** AI never changes a feeding schedule and never medicates. It recommends, a keeper decides, and the recommendation is stored apart from the welfare record.

### AI-3 Crowd flow and staffing

- **Inputs.** Occupancy per attraction in 15-minute buckets from gate scans and people counters, plus queue length estimates and dwell time.
- **Forecast and roster.** Demand forecast by day, weather, school holidays and events. An optimiser suggests rosters, and GenAI writes the daily ops brief explaining why.
- **How we know it works.** Forecast error tracked against actuals; roster suggestions compared with what ops actually did and with the queues that resulted.

### AI-4 Guest guide (customer-facing GenAI)

- **Knowledge.** RAG over curated content: animals, rides, history, accessibility, food, safety. Live inputs are queue lengths, show times and weather.
- **Personalisation.** An itinerary for a given family composition and time budget, plus push alerts when a favourite ride is quiet.
- **Guardrails.** Grounded answers with citations required, refusal on safety and medical topics, PII redaction, per-user rate limits.
- **Offline.** The app caches the knowledge base and a static map; on a poor link the chat degrades to keyword search.

### AI-5 Retention and revenue

Two goals: get more visitors to come back (F5) and make quiet days pay (F6). Three separate jobs, all running overnight in the cloud, none of them touching a guest while they are in the park.

- **Work out who is likely to return.** A model we own scores each guest on how likely they are to visit again, and on whether a membership or family pass would suit them, from their visit history and app activity. Only guests who have consented are scored; everyone else is treated as an anonymous group.
- **Suggest a price for a quiet day.** Starting from the AI-3 demand forecast, a model estimates how much a lower midweek price would lift bookings, and proposes one. It can only move within set floors and caps, it sets one price for that day for everybody rather than a personal price per guest, and the price is published in advance. A revenue manager approves or rejects every suggestion; nothing reaches the ticketing system on its own.
- **Write the follow-up message.** Overnight, GenAI drafts each consented guest a short recap of their visit and picks one offer from a catalogue marketing has already approved. It may only use facts from that guest's own visit record and offers from that catalogue. Every draft is checked against both before sending, and a draft that fails the check is dropped and reported, never sent.
- **How we know the messages work.** A small random group of guests, chosen permanently, never gets a personalised message; they keep receiving the plain template. Because return rates move with the weather and the season anyway, the only honest measure of whether personalisation is earning its keep is the gap between that group and everyone else. We watch returns, redemptions and unsubscribes across both.
- **How we turn it off.** If the personalised messages stop beating the untouched group, or unsubscribes rise, a config switch sends everyone the plain template again. Pricing has its own switch back to the fixed calendar. Neither needs a code change.

### AI-6 Ride condition monitoring

The 40 rides are 18th century and have just passed inspection. The inspection regime does not change at all. The only gap is that inspectors have no way to tell which ride deserves attention first, and this fills exactly that gap and nothing more.

- **What is measured.** Vibration sensors on each ride's main bearings and drive, plus a counter for how many times it has run, reporting over MQTT to the rides gateway.
- **What the model does.** Each ride gets its own model, because no two of these machines are alike, and a new rattle only means something relative to how that particular ride normally sounds. It learns that ride's usual signature, allowing for temperature and load, and flags when the signature shifts.
- **What comes out.** A condition score per ride and a ranked list for the inspectors, each entry showing the evidence: which reading moved, when, and by how much.
- **The limit.** The model cannot stop a ride, change its schedule or sign anything off. Only an inspector takes a ride out of service, and the scheduled inspections happen on their own calendar whether or not the model has flagged anything. All it changes is the order of the queue.
- **How we know it works.** Every inspection outcome is recorded against the score the model gave that ride, so over time we can see whether a high score actually predicted a problem. If the model is unavailable, inspectors fall back to raw threshold alarms and the fixed schedule, which is where they are today.

---

## 5. Dealing with AI uncertainty: models, prices, vendors

This is the section you asked about most, and it maps directly to a judges' criterion.

**The principle: models are configuration, not code.** Everything below exists so that a better model, a price rise or a provider disappearing is an operational event, not a rewrite.

### 5.1 Options for how GenAI is integrated

| Option | Description | Strengths | Weaknesses | Verdict |
|---|---|---|---|---|
| A. Direct SDK, single vendor | Services call one provider's SDK | Fastest to build; full access to provider features | Total dependency; price and shutdown risk unmitigated; no central metering | Rejected |
| B. Hyperscaler catalogue, capability map in config | Bedrock, Vertex or Azure AI Foundry as the only integration; a shared client resolves capability to model from configuration | One bill, IAM, private networking, committed-use pricing, data residency; the catalogue's unified API *is* the adapter layer, so many model families cost nothing to reach; nothing added to the guest path | One account and one control plane in front of every AI feature; lags first-party APIs and some models never arrive; hard cost caps and per-guest rate limits must be built | **Recommended** |
| C. Library abstraction | A framework (LangChain-style) wraps providers inside each service | Light; no new component to run | Abstraction scattered across services; no aggregate budget view; frameworks churn as fast as models | Rejected |
| D. Independent AI Gateway | One internal service in front of all providers | Central control of cost, quality, failover and policy; providers become plugins; genuine cross-cloud failover | Another component to run and secure, on the path of every guest chat; per-adapter parity work | Rejected on cost of ownership |

The recommendation is B, with a self-hosted open-weight tier *outside* the catalogue as the continuity path and a non-AI floor beneath that.

D is the better architecture on paper and it is worth being clear about why it loses. Its central control is real. But its two recurring costs, an adapter per provider and a highly available service on the path of every guest chat, are both paid by a platform team that also has to keep the gates open, and the catalogue's unified API now does the adapter half for free. The part of D that actually delivers "models are configuration" is the capability contract, and that survives in B unchanged: it simply lives in a config file and a client library instead of a service.

What B does not give you, and what you therefore have to build, is set out in section 5.2 rather than left implicit. Revisit D if a second cloud ever becomes a requirement.

### 5.2 The design

**Capability contracts.** A feature asks for a *capability*, never a model: `chat.guide`, `summarise.welfare`, `explain.ops`, `classify.sentiment`, `generate.offer`, `embed.text`. Each contract states:

- required features (structured output, vision, tool use, context size),
- p95 latency,
- maximum cost per 1,000 requests,
- minimum eval score.

**Model registry.** A versioned catalogue of models and providers, holding eval scores per capability, a price sheet (input, output, cached input, batch), rate limits, data-residency flags, status (candidate, active, deprecated, blocked) and deprecation dates. The price sheet is the single source of truth for cost, so a provider price change is entered once and every dashboard reprices immediately.

**Routing policy.** Per capability, an ordered list of candidates that have passed eval. Routing is by capability and provider health, not per-request cascading — caches are model-scoped, so a cascade forfeits them. Measure a capable model at lower effort before reaching for a cascade.

**Provider portfolio, ordered by failure domain.**

| Tier | Purpose | Failure domain | Example |
|---|---|---|---|
| 1a | Day-to-day quality | Catalogue, primary region | The best eval-passing frontier family the catalogue carries |
| 1b | Model deprecation, model-specific degradation, regional capacity | Catalogue, secondary region | A second family, or the same family in another region |
| 2 | Loss of the catalogue, the account or the commercial relationship | Outside the catalogue's control plane | Llama, Qwen or Mistral class on vLLM, in a *separate* account with a separate provider |
| 3 | The product keeps working with zero GenAI | Your own services | FAQ search, templated reports, rule-based offers |

Tiers 1a and 1b share an account and a control plane, so anything above the model level fails together. Do not present two regions as two vendors. The real independence is at Tier 2, which is why under option B it has to be funded and drilled rather than aspirational.

**Cost governance.**

- *Meter everything.* Every request logs input, output and cached tokens with the registry price at the time, tagged by capability, tenant and feature flag.
- *Budget per capability, daily and monthly.* Warn at 120% of plan, auto-shift to the next eval-passing candidate at 150%, degrade to Tier 3 at a hard cap.
- *Rate-limit guest-facing chat per user.* Abuse is the real budget killer, not list prices.
- *Pull the free levers before swapping models:* prompt caching of the stable prefix (system prompt, knowledge chunks), semantic caching of repeated guest questions, the batch API for nightly briefs and offer generation (half price), and lower effort settings for routine work.

**Availability governance.** A circuit breaker per candidate (opens on error rate or latency), automatic failover to the next candidate in the capability map, health probes, and a per-feature kill switch behind a feature flag.

**What the catalogue does not do, and you must.** This is the honest half of choosing B:

| Gap | Why the catalogue does not close it | What you build |
|---|---|---|
| A hard cost cap | It alerts on budget; it does not stop inference. Account-level budget actions are too blunt to aim at one capability | A metering consumer that flips the capability's feature flag to Tier 3. Off the request path, so it need not be highly available |
| Per-guest rate limits | Not a model-layer concern | Limits at the public API edge, before a call is ever made |
| Semantic caching | Prompt caching is native; semantic caching is not | Built once in the Engagement service, for the guest guide only |
| Capability indirection | Nothing resolves a capability to a model for you | The capability map plus a thin client library |
| A failure domain outside the cloud | Every model in the catalogue shares one control plane | Tier 2 self-hosted in a separate account with a separate provider |

**Quality governance.** Covered in section 7. The point here is that evals are what make swapping *safe*: you can only replace a model if you can measure the replacement.

**Data ownership and exit plan.**

- Prompts, completions, traces, eval sets, feedback labels, embeddings *and their source text* all live in your own storage.
- Re-embedding is just a batch job, so the embedding model is replaceable too.
- Prompt templates stay provider-agnostic, with small per-model dialect overlays.
- Avoid fine-tuning closed models. If you must, treat the fine-tune as disposable and keep the training set.
- Do not build on proprietary stateful features (hosted memory, assistant threads) without an adapter.

**Edge AI is immune to all of this.** Vision and anomaly models are owned artifacts (ONNX), versioned in your own registry and deployed to gateways over MQTT. Vendor churn cannot touch them.

### 5.3 Playbooks for the three scenarios in the brief

**A better model appears.** Register it → nightly eval against the golden sets → shadow 2-5% of live traffic, scored offline → canary 10% → promote to first candidate. Days of elapsed time, zero code changes, reversible by config.

**The provider changes prices.**

1. Update the price sheet; cost dashboards reprice historical and forecast spend instantly.
2. Apply the free levers first (caching, batch, effort settings) before anything that affects quality.
3. If the capability still breaches its budget, the budget consumer shifts traffic to the next eval-passing candidate — or the team negotiates committed-use pricing on the catalogue.

**The provider shuts down.** Under a single catalogue this splits into two cases, and saying so is more convincing than blurring them.

- *One model is withdrawn or degrades.* The circuit opens within seconds and traffic moves to the next candidate, a second family or region inside the catalogue. This is the common case and it is absorbed completely.
- *The catalogue, account or commercial relationship is lost.* Every candidate fails at once, because they share a control plane. Tier 2 self-hosted takes over at eval-verified acceptable quality in a separate account with a separate provider; capabilities it cannot serve go to Tier 3. Every generative feature degrades together, which is the accepted price of B. What bounds the damage is that the degradation is designed, eval-scored and drilled monthly rather than discovered on the day.
- *Announced deprecation.* The registry's deprecation calendar raises a ticket 90 days out, the successor is evaluated early, and version pins move on your schedule.

### 5.4 Worked cost model (why cost is a governance problem, not an existential one)

The guest guide (AI-4) dominates GenAI spend, so model that and the rest is noise.

Assumptions at target scale: 15,000 visitors a day, 25% use the guide, 6 turns each, roughly 2,500 input tokens per turn (mostly a cacheable prefix) and 250 output tokens — about 56M input and 5.6M output tokens a day.

Prices are Anthropic first-party list prices as of June 2026. Treat them as illustrative; the live sheet lives in the registry.

| Model routed to `chat.guide` | Input $/M | Output $/M | Approx. daily (uncached) | Approx. monthly |
|---|---|---|---|---|
| Claude Haiku 4.5 | 1.00 | 5.00 | $84 | $2.5k |
| Claude Sonnet 5 | 2.00 | 10.00 | $169 | $5.1k |
| Claude Opus 5 | 5.00 | 25.00 | $421 | $12.6k |

Those are the uncached figures. Prompt caching on the stable prefix cuts the input side substantially (cache reads bill at a fraction of base input price) and semantic caching removes repeated questions entirely. Nightly briefs, ops summaries and offer generation are a few hundred calls a day, so they are negligible.

**The argument for the judges.** Against gate revenue at 15,000 visitors a day, GenAI spend is a rounding error: a 2x price hike is uncomfortable, not fatal, and the policies above absorb it automatically. The risks that genuinely deserve architecture are **dependency** (availability, deprecation) and **quality drift** — which is why the capability contract, the provider portfolio and the eval loop are the core of the answer. The same numbers are also the case against option D: at a few thousand a month, a gateway service with its own on-call rotation would cost more to own than the traffic it governs.

---

## 6. Conformance: do the AI additions match the base architecture?

| Characteristic | How each AI addition conforms |
|---|---|
| Availability under partition | Edge AI runs locally; cloud GenAI features degrade to Tier 3 fallbacks; nothing safety-related depends on a model call |
| Evolvability | Models are registry entries behind capability contracts, repointed by config within a minute; owned models deploy over MQTT |
| Observability | Every model call traced with tokens, cost, latency, eval-tagged outputs; edge models report confidence and drift metrics |
| Data integrity | AI never writes to ticketing, payments or animal records directly; it produces recommendations that humans or deterministic services apply |
| Elastic scalability | GenAI calls are made in-process, so there is no shared component to scale; batch workloads run off-peak; edge inference is per-zone |
| Cost transparency | Token metering per capability with budgets and alerts |
| Security and privacy | Video stays on the edge; PII redaction before any external model call; consent gates personalisation |

---

## 7. Validation and verification of AI (does it work?)

**Before production.**
- Golden datasets per capability, human-labelled: guest questions with reference answers and required citations; anomaly windows with keeper verdicts; counting frames with ground-truth counts.
- Automated eval run in CI on every prompt, retrieval or model change. Scores: accuracy, groundedness, refusal correctness, schema validity, tone (for guest content), latency and cost. LLM-as-judge with a human-calibrated rubric plus a sampled human review.
- Promotion gate: no model or prompt goes live below the capability's minimum score.
- Determinism where possible: structured outputs, low temperature, deterministic pre- and post-processing, so only the language layer is non-deterministic.

**In production.**
- Tracing (OpenTelemetry to a tool such as Langfuse or Arize Phoenix) for every call, including retrieved chunks and the final answer.
- Online signals: schema failure rate, citation coverage, guest thumbs up/down, keeper accept/reject rate, escalation-to-human rate, forecast error versus actuals, count disagreement versus manual counts.
- Drift monitors with alert thresholds on those signals, plus latency, cost and refusal rate. Alerts route to the owning team; a kill switch disables the feature and the Tier 3 fallback takes over.
- Shadow and canary deployments for every model or prompt change; automatic rollback when the canary's online signals fall below the control's.
- Periodic human audits: sampled guest conversations, sampled welfare briefs against the underlying data, monthly manual animal counts.

---

## 8. ADRs to write

Keep each to one or two pages: Title, Status, Context (with alternatives), Decision (the why), Consequences (trade-offs).

| ADR | Title | Key trade-off |
|---|---|---|
| 001 | Edge-first, event-driven hybrid architecture | Resilience and privacy versus more hardware and two deployment surfaces |
| 002 | MQTT topology, QoS and store-and-forward | Simplicity of QoS 1 with idempotent consumers versus exactly-once cost |
| 003 | Offline-verifiable signed tickets | Offline entry versus revocation latency |
| 004 | Classic ML versus GenAI selection principle | Ownership and determinism versus development effort per model |
| 005 | Model access through a hyperscaler catalogue | Cost of ownership and no component on the guest path, versus correlated failure and catalogue lag |
| 006 | Multi-provider portfolio with self-hosted open-weight fallback | Continuity versus cost of idle capacity and lower fallback quality |
| 007 | Model registry, price sheet and cost policies | Automation versus the risk of automated quality downgrades (mitigated by eval gates) |
| 008 | Evaluation-gated model promotion (offline, shadow, canary) | Confidence versus slower adoption of new models |
| 009 | RAG and prompting over fine-tuning closed models | Portability versus peak accuracy on narrow tasks |
| 010 | Edge computer vision with no cloud video | Privacy and bandwidth versus edge GPU cost and harder model updates |
| 011 | Human-in-the-loop for welfare and ride alerts | Safety and trust versus slower response and keeper load |
| 012 | LLM observability, guardrails and kill switches | Cost of tracing storage versus blind spots |
| 013 | Privacy-preserving footfall and consent-based personalisation | Less signal versus legal and reputational risk |
| 014 | Caching and batch as first cost levers | Staleness of semantic cache versus spend |

---

## 9. Diagrams to produce

Use Mermaid in markdown (GitHub renders it) so diagrams live next to the ADRs and diff cleanly.

1. C4 level 1: system context (guests, keepers, ops, vets, payment provider, AI providers, weather).
2. C4 level 2: containers, split edge and cloud.
3. Zone deployment view: devices, gateway, backhaul.
4. Data flow: device to cloud to warehouse, with buffering.
5. Comprehensive AI view: the model access layer, registry, eval pipeline, provider tiers.
6. Targeted views, one per AI-1 to AI-7: a sequence or flow diagram plus the fallback path.
7. Model lifecycle: candidate → eval → shadow → canary → active → deprecated.
8. Failure scenarios: provider outage, price breach, link partition.
9. Validation loop: golden sets, CI gate, tracing, online signals, rollback.

---

## 10. Repository layout and 7-day plan

```
README.md                     navigation, characteristics, how to read
docs/overview.md              short narrative (including how the team used AI)
docs/architecture/            base architecture + C4 diagrams
docs/ai/                      one file per AI use with targeted diagram
docs/uncertainty.md           models, prices, vendors (section 5)
docs/validation.md            evals and production monitoring (section 7)
docs/adr/                     ADR-001 ... ADR-014
docs/implementation/          optional: model access config, MQTT topics, eval harness sketch
```

| Day | Date | Work |
|---|---|---|
| 1 | Wed 9 Sep | Agree base architecture and characteristics; write ADR-001 to 003; C4 L1 and L2 |
| 2 | Thu 10 Sep | AI use cases AI-1 to AI-4 with targeted diagrams |
| 3 | Fri 11 Sep | AI-5 to AI-7; model access design; ADR-004 to 008 |
| 4 | Sat 12 Sep | Uncertainty section, cost model, failure diagrams; ADR-009 to 011 |
| 5 | Sun 13 Sep | Validation section, model lifecycle diagram; ADR-012 to 014 |
| 6 | Mon 14 Sep | README, overview narrative, optional implementation details |
| 7 | Tue 15 Sep | Review against the judges' table in section 1; buffer; submit before Wed deadline |

---

## 11. Draft overview narrative (starter)

The Von Digitalis Estates need to sell more tickets, keep 200 exotic animals healthy, and learn where their 5,000 daily visitors actually go, on a site where WiFi is unreliable. We designed an edge-first, event-driven platform: MQTT devices and zone gateways keep gates, alarms and keeper tools working when the cloud is unreachable, and a cloud tier owns ticketing, analytics and AI. AI is used where it changes an outcome: on-device vision counts the piranhas, anomaly models flag sick or under-fed animals before a vet bill arrives, forecasting places staff where crowds will be, and a grounded guest guide turns a first visit into a return visit. Every generative model is reached by naming a capability that configuration resolves to an entry in a single model catalogue, backed by a priced model registry, a portfolio ordered by failure domain and evaluation-gated promotion, so a better model, a price rise or a vendor shutdown is a configuration change, not a rewrite. Every AI output is measured before release and monitored in production, with a human in the loop wherever an animal, a ride or a price is involved.
