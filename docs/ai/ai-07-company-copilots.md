# AI-7 Company copilots

## Problem

The estate's staff spend time on work that language models do well: sorting support requests, writing up incidents, and turning a keeper's spoken notes into structured records. None of it is guest-facing or safety-critical, all of it is language in and language out, and all of it slows the people who should be looking after animals and guests (brief F8 and F9; requirements F6.2, F6.3).

## Approach

Three internal copilots, all reached by naming a capability, all producing drafts that a person confirms.

| Copilot | Capability | What it does | Who confirms |
|---|---|---|---|
| Support triage | `triage.support`, `classify.sentiment` | Classifies incoming guest messages (ticketing, lost property, complaint, accessibility), drafts a reply from approved templates, and flags unhappy guests to AI-5 for service recovery | Support agent before send |
| Incident summaries | `summarise.incident` | Builds a timeline and summary from ops logs, alerts and staff notes for any incident (ride stoppage, animal escape drill, first aid) | Duty manager before filing |
| Keeper voice notes | `transcribe.notes` | Transcribes a keeper's spoken observation on the tablet and extracts it to the welfare record schema (animal, behaviour, appetite, concern level) | Keeper reviews fields before saving |

Prompts and the extraction schema are versioned in the registry. All three copilots run over redacted text and produce structured output that is validated before it is shown.

## Targeted view

```mermaid
flowchart LR
  IN["Guest message, ops logs, or keeper voice note"] --> RED["Redaction and schema selection"]
  RED --> GWY["Capability triage.support, summarise.incident, or transcribe.notes"]
  GWY -->|"provider healthy"| DR["Draft with structured fields"]
  GWY -->|"all tiers down or budget cap"| MAN["Blank form, manual process"]
  DR --> VAL["Schema validation"]
  VAL -->|"pass"| HUM["Person reviews and confirms"]
  VAL -->|"fail"| MAN
  MAN --> HUM
  HUM --> REC["Record saved or reply sent"]
  HUM -->|"edits as feedback"| EV["Eval set"]
```

## Data and models

| Input | Source | Cadence | Where stored |
|---|---|---|---|
| Guest messages | Email, app, web form | Continuous | Support system |
| Ops logs and alerts | Park operations service | Continuous | Event backbone, warehouse |
| Keeper voice notes | Keeper tablet, offline-first | Ad hoc | Encrypted restricted-data store; retention is policy-controlled, with audit and legal holds |
| Reply templates | Support lead, approved | As edited | CMS |
| Confirmed edits | Staff | Per use | Eval set in data lake |

| Model | Type | Ownership |
|---|---|---|
| Triage and reply drafting | GenAI via `triage.support` | Catalogue capability |
| Sentiment | GenAI via `classify.sentiment` | Catalogue capability |
| Incident summary | GenAI via `summarise.incident` | Catalogue capability |
| Transcription | Speech to text in an approved restricted-data endpoint inside the estate cloud boundary | Estate-controlled processing path |
| Field extraction | GenAI via `transcribe.notes`, from the redacted transcript only | Catalogue capability |

## Where it runs

In the cloud. The keeper tablet records audio offline and uploads it when a link is available to an encrypted restricted-data store. Raw audio is sent only to the approved transcription endpoint inside the estate cloud boundary; it never goes to the shared external model catalogue. PII redaction runs on the transcript before the catalogue extracts structured fields. The extracted record comes back on the next sync. Support and incident copilots are used from the office network.

The legal and licensing owner sets the encrypted audio retention period in policy. Deletion is never triggered merely by confirmation: a randomly selected quality-audit sample is placed on hold at confirmation, and a legal or incident hold also suspends deletion. When the policy window and every hold have ended, the source audio is deleted automatically and the access log is retained with the confirmed record.

## Degradation and fallback

| Condition | What staff see |
|---|---|
| Keeper tablet offline | Note is stored locally; extraction arrives after sync; keeper can type fields manually |
| One model or region fails | No visible change; the next candidate in the catalogue serves the call |
| The whole catalogue is unreachable | Tier 2 self-hosted drafts at lower quality; if it is gone too, staff get the blank form and the manual process |
| All tiers down or budget cap | Blank form and manual process, exactly as today |
| Schema validation fails | Draft discarded; blank form shown; failure counted |

## Validation

Pre-production:
- Golden sets: labelled support messages with correct categories and reference replies; past incidents with reference timelines; recorded keeper notes with hand-filled records.
- Metrics: routing accuracy, reply grounding in templates, timeline faithfulness, field-level extraction accuracy, schema validity.
- Gate: each capability's minimum score must be met; extraction must reach the agreed field accuracy before keepers see it.

Production:
- Online signals: edit distance between draft and confirmed version, schema failure rate, time saved per item, staff thumbs.
- Thresholds: alert when edit rate rises or schema failures cross the cap.
- Rollback: prompt and model versions in the registry; feature flag returns the blank form.
- Human audit: every draft is reviewed before use; the held monthly sample of confirmed records is checked against source audio and logs before its hold is released.

## Conformance

| Property | How AI-7 honours it |
|---|---|
| **Invariant — life safety** | Copilots have no authority over containment, duress, animal treatment, ride operation or emergency response |
| **Invariant — data integrity** | A person confirms before any record is saved or reply sent |
| **Invariant — security and privacy** | Raw audio stays in the restricted transcription path; only a redacted transcript reaches the external catalogue. Encryption, access logs, retention policy and audit/legal holds govern deletion |
| Availability under partition | Tablet works offline; copilots are non-critical and fail to the manual form |
| Evolvability | Capabilities, not models; schemas and prompts are versioned |
| Observability | Edit rate and schema failures per capability |
| Elastic scalability | Stateless calls, made in-process against the catalogue |
| Cost transparency *(constraint)* | Small budgets per capability; negligible volume |

## Value

- Records become structured enough to feed AI-2. That is structural: a transcript extracted to a schema is machine-readable in a way a paper note is not.
- **Hypothesis:** keepers spend minutes on records instead of half an hour, support replies go out faster, and incident write-ups become more consistent. Time saved is the claim most often asserted and least often measured for this class of feature, so it is measured here — edit distance before acceptance, and time-to-record sampled before and after — rather than assumed.
- **Break-even condition:** at a running cost measured in tens of pounds a month, the feature pays for itself on a saving of well under an hour per staff member per week. The reason to be careful is the opposite risk: a transcription that needs heavy correction costs more time than it saves, which is why nothing enters a record unconfirmed and why edit distance is the signal that would kill the feature.

## Related

- [ADR-005 Model access and capability contracts](../adr/ADR-005-model-access-and-capability-contracts.md)
- [ADR-011 Human in the loop](../adr/ADR-011-human-in-the-loop.md)
- [ADR-012 LLM observability and kill switches](../adr/ADR-012-llm-observability-and-kill-switches.md)
- [AI-2](ai-02-welfare-anomaly-and-brief.md), [AI-5](ai-05-retention-and-revenue.md), [AI overview](00-ai-overview.md)
