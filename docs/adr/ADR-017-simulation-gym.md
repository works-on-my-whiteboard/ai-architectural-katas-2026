# ADR-017: Simulation gym for estate and delivery decisions

## Status

Accepted, 2026-09-16. Supersedes / Superseded by: none.

## Context

The estate must triple visitor volume in three years while keeping a venomous collection well and a collection of eighteenth-century rides running. That path is a sequence of expensive and largely irreversible decisions: which zones to invest in, what qualified staffing an operating plan needs, whether a plan survives at 15,000 visitors a day, and in what order to deliver the programme.

[AI-3](../ai/ai-03-crowd-flow-and-staffing.md) already forecasts demand and proposes a roster. It does so against the estate as it currently is, one day ahead. It cannot evaluate a counterfactual, because the factors interact: queues drive dwell time, dwell time drives occupancy, a closure displaces crowds onto neighbours, and a staffing pattern that works at 5,000 visitors can fail at 15,000 for reasons a forecast does not expose.

The default alternative is to test changes on live guests and live animals.

Forces:

- Guest safety and animal welfare are not tradable against revenue, and a decision process that compares them numerically will eventually trade them.
- The estate is small and cannot absorb a large misinvestment; it also cannot afford a large modelling programme.
- Data to calibrate against exists only once the occupancy, work-order and cost capture of earlier phases is running.
- The estate the model is calibrated against is continuously disappearing as volume grows, so calibration decays by design.
- A "digital twin" is easy to admire and easy to mistake for reality.

### Alternatives considered

| Option | Summary | Why not (or why partially) |
|---|---|---|
| Dashboards and forecasts only | Keep AI-3 and the reporting surface; decide from description and judgement | Describes the past well and cannot test interacting counterfactuals. This remains the fallback whenever the gym is unavailable or uncalibrated |
| High-fidelity 3D digital twin first | Visual model of the estate with detailed physical fidelity | Expensive, slow, and rarely required to answer a staffing or capacity question. Visualisation is not the part that decides |
| Ask a generative model to produce scenarios and outcomes | Prompt an LLM for the likely effect of a plan | Fluent prose with no valid model of constraints or uncertainty. It would produce confident numbers with no provenance, which is the failure mode this estate can least afford |
| Optimise against live operations | Change the plan, measure the result, iterate | Real feedback, but the experiment runs on guests, animals and staff. Rejected on safety grounds |
| Calibrated simulation gym, read-only, constraints before revenue | Chosen | Needs input data, modelling expertise and ongoing calibration, and it describes plausible outcomes rather than guarantees |

## Decision

Build a **simulation gym**: two separate owned models, read-only, with hard constraints evaluated before any comparison of revenue.

1. An **estate operations model** — discrete-event simulation of arrival profiles, timed-entry capacity, throughput and cycle times, queue formation, availability, staff roles and qualifications, weather and events, maintenance windows, cost, revenue and guest itinerary choices.

2. A **delivery portfolio model** — project scope, dependencies, required capabilities, team availability, ramp-up, budget and delivery risk. Separate on purpose: how the park operates and how the transformation is staffed are different questions, and one model answering both answers neither well.

Four rules make it trustworthy rather than merely impressive:

- **Inputs are distributions calibrated from the estate's own observed data**, not single values. Single-value inputs produce a confident answer about a park that does not exist.
- **Hard constraints are a gate, not a weighting.** A scenario breaching guest safety, animal welfare, licence obligations or mandatory maintenance is rejected with its reason and never enters the ranking. Only surviving scenarios are compared on service, cost, resilience and revenue. Constraint definitions are authored and signed by the safety and welfare owners; an unowned constraint blocks the run.
- **Outputs are ranges with their reasoning** — queue percentiles, staff utilisation, welfare and maintenance coverage, cost and revenue ranges, milestones at risk, the binding bottleneck, and the assumptions the result is most sensitive to. The existing `explain.ops` capability renders a comparison in plain language; no new capability is added.
- **It is read-only.** No write path into any operational service. It cannot publish a roster or a price, schedule maintenance, alter a ride control or operate an enclosure.

Scope is bounded to three questions for the first version: which zones break first at 15,000 visitors a day and why; what qualified staffing mix a proposed operating plan needs; and what team capability mix and sequencing the roadmap needs. A 3D representation is added only if it demonstrably changes a decision.

No scenario informs a material decision until a blind historical back-test passes and interval coverage holds. Every scenario is recorded with its question, owner, input and constraint versions, assumptions and random seed, and the actual outcome is recorded against it once the decision has run. That comparison, not a demonstration, is how the gym earns continued use.

This is a P4 capability. It cannot compete with gates, alarms and welfare records for delivery attention, and because nothing operational depends on it, its absence degrades no estate function.

## Consequences

### Positive

- Investment, capacity and staffing options are compared before capital, guests, animals or staff are exposed to a live experiment.
- Safety and welfare are structurally protected: an unsafe-but-profitable plan cannot reach a decision-maker, because it is rejected before ranking.
- The binding constraint on the growth path becomes explicit, which converts the 15,000-a-day target into a sequenced plan.
- Retained scenario records make the estate's own decision quality measurable over time.
- One avoided misinvestment pays for the capability.

### Negative

- Requires operations-research skill the estate does not currently have, and disciplined ongoing calibration.
- Garbage in, confident garbage out: the gym is only as good as the occupancy, work-order and cost capture beneath it, which is why it sits after those phases.
- A model that produces ranges invites a decision-maker to read the most convenient end of the range.
- Calibration decays as the estate grows, so validation is a standing cost rather than a one-off gate.
- It can be mistaken for a prediction. The wording of every output, and the refusal to build a 3D twin early, are deliberate defences against that.

### Trade-off analysis

| Quality attribute | Effect | Mitigation |
|---|---|---|
| Availability under partition | Neutral; not operationally required | Batch, cloud-only, P4; dashboards and judgement remain the fallback |
| Evolvability | Improved; models are versioned owned artifacts and constraints are configuration | Reuses `explain.ops` rather than adding a capability |
| Observability | Strongly improved; predicted against actual becomes a tracked metric | Append-only scenario records with seeds and assumptions |
| Data integrity | Protected by construction | Read-only, no write path to any operational store |
| Cost transparency | Compute attributable per scenario to the decision that asked for it | Scenario records carry their own cost |
| Safety | Improved, provided constraints stay a gate | Adversarial tests that a profitable unsafe scenario is rejected; constraints signed by their owners |
| Security and privacy | Neutral | Aggregates and asset-level facts only; no visitor identifiers |

## Related

- [AI-8 Simulation gym](../ai/ai-08-simulation-gym.md), the use-case view
- [ADR-004](ADR-004-classic-ml-vs-genai-selection.md), why the numeric work is owned and GenAI only explains
- [ADR-011](ADR-011-human-in-the-loop.md), why the gym ranks and a person decides
- [ADR-015](ADR-015-occupancy-grain-and-operating-hour-normalisation.md), the occupancy grain it calibrates against
- [ADR-016](ADR-016-local-incident-response.md) and [06-safety-case](../architecture/06-safety-case.md), the constraints it must never trade away
- [07-attraction-economics](../architecture/07-attraction-economics.md), recorded-fact popularity against cost
