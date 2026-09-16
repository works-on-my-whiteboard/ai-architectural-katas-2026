F1. Ticketing and access
ID	Requirement	Priority	Acceptance criteria
F1.1	Guests can buy day tickets online and at the gate	Must	Purchase completes in under a minute; ticket delivered to the app, wallet or print
F1.2	Family passes cover a defined group	Must	One purchase issues a group token; each member is admitted individually; group size is enforced
F1.3	Tickets are validated at entrance and ride gates	Must	Validation in under two seconds; works offline; a ticket cannot be used twice for the same admission
F1.4	Refunds, cancellations and revocation	Must	A revoked ticket is refused at the gate within a defined sync window even when the link is intermittent
F1.5	Memberships and annual passes	Should	Renewals and member benefits are supported by the same validation path
F1.6	Timed entry and capacity limits	Could	Sales can be capped per time slot when the park approaches capacity
F2. Popularity and operations
ID	Requirement	Priority	Acceptance criteria
F2.1	Measure occupancy per attraction and area	Must	Counts available in 15-minute buckets, derived from gate scans and anonymous counters; popularity is expressed per operating hour so a ride closed for repair is not mistaken for an unpopular one
F2.2	Estimate queue length and dwell time	Should	Queue estimates published to ops and to guests with a stated confidence. Dwell time is reported as queue time and viewing time separately: for a ride, queue time is a cost to the guest; for an enclosure, viewing time is value
 to the guest. Long queue plus short viewing marks a bottleneck; long viewing plus low occupancy marks a hidden gem
F2.3	Forecast demand	Should	Day-ahead and week-ahead forecasts by area, with forecast error reported against actuals
F2.4	Recommend staffing	Should	A roster suggestion per day with an explanation; managers can accept, edit or reject
F2.5	Heatmaps and investment reporting	Must	Management can see the most and least popular areas over any period
F2.6	Value per attraction	Should	For each ride and enclosure, popularity (riders or visitors per operating hour) is shown against operating cost: maintenance hours, parts, downtime and staff. A ranked list of best and worst return over any period, so a popular
 ride that costs more than it earns is visible and a cheap, well-liked ride is not overlooked
F3. Animal welfare
ID	Requirement	Priority	Acceptance criteria
F3.1	Monitor enclosure environment	Must	Temperature, humidity and water quality (pH, dissolved oxygen, ammonia, salinity, turbidity where relevant) recorded per enclosure
F3.2	Monitor feeding quantity and quality	Must	Food dispensed and leftover recorded per feeding; deviations flagged
F3.3	Record keeper observations and vet records	Must	Keepers can log observations on a tablet offline; records sync when connected
F3.4	Detect welfare anomalies early	Should	Anomalies flagged with the supporting data; keepers can accept or reject each flag; acceptance rate tracked
F3.5	Daily keeper brief	Should	A plain-language morning brief per enclosure, citing the data points behind every statement
F3.6	Safety alarms work offline	Must	Environmental and safety alarms fire on the zone network with the cloud unreachable
F4. Piranha population
ID	Requirement	Priority	Acceptance criteria
F4.1	Automated population estimate	Must	An estimate with a confidence interval per feeding; a trend over time
F4.2	Manual reconciliation	Must	Monthly manual count recorded and compared with the estimate; disagreement tracked
F4.3	Video stays on the estate	Must	No raw video leaves the zone; only counts, confidence and a small number of sampled frames
F5. Guest growth and retention
ID	Requirement	Priority	Acceptance criteria
F5.1	Guest app with tickets, map and live information	Must	Works with poor signal: tickets, map and core content cached on the device
F5.2	Guest guide	Should	Answers grounded questions about animals, rides, history and accessibility with citations; refuses medical and safety advice safely
F5.3	Personalised itinerary	Should	A plan for the family's ages and time budget; updates when queues change
F5.4	Re-engagement	Should	Post-visit recaps and offers sent only to guests who consented; performance measured
F5.5	Off-peak pricing suggestions	Could	Suggestions with floors and caps; a human approves any change; no per-person price discrimination
F5.6	Feedback capture	Must	Guests can rate guide answers and the visit; ratings feed quality monitoring
F6. Profitability and internal efficiency
 
ID	Requirement	Priority	Acceptance criteria
F6.1	Ride condition advisory	Should	Vibration anomalies per ride baseline surfaced to engineers as a priority list; inspection regime unchanged
F6.5	Maintenance log per ride and enclosure	Should	Work orders, downtime, parts and labour cost recorded against the asset; feeds the cost side of F2.6 and the per-attraction view of G6
F6.2	Support triage and incident summaries	Could	Support requests categorised and routed; incident summaries drafted for human review
F6.3	Keeper voice notes to records	Could	Spoken observations transcribed into structured records that the keeper confirms
F6.4	AI cost visibility	Must	Spend per AI feature visible daily; budgets and alerts in place
 
 
Constraints
 
Constraint	Source	Effect on the design
Patchy WiFi	Brief	No critical function depends on WiFi; wired, LoRaWAN and private backhaul for devices
Estate-to-cloud path required	Brief	Zone gateways bridge MQTT to the cloud with store-and-forward
MQTT device budget	Brief	MQTT is the device protocol for scanners, counters, sensors, feeders and ride sensors
Historic rides	Brief	Sensors must be non-invasive; safety remains with inspectors
Poisonous animals	Brief	Keeper safety alarms are local and deterministic
AI market uncertainty	Judges' criteria	Models are configuration, not code; prices and providers can change without a rewrite
 
 
