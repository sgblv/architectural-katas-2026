# 043. Animal welfare monitoring and the limits of automated action

## Status

Proposed

## Decision

Deterministic thresholds set by a veterinarian may act on hardware. Model output may only advise a human.

## Context

Animal care is costly and gets costlier when an animal falls ill (P3). Feeding is unmeasured (P9) and the piranhas are uncounted (P6). ADR-042 defines the telemetry arriving from the enclosures. This record covers what the system may do with it, given that model behaviour is not deterministic (C12) and nobody on the estate could diagnose one misbehaving (C13).

## Thresholds come from a vet

Safe ranges, alert thresholds and per-species response procedures are supplied by a veterinarian and held as versioned configuration. The system never defines what unhealthy means, and it can always report which version was in force when an alert fired.

## Three classes of event

| Class | Example | Automatic action | Who is told, how fast |
|---|---|---|---|
| **1. Risk to people** | Containment door opens on a venomous enclosure | None, alarm only | Security and duty keeper, immediately, on the estate |
| **2. Fast risk to an animal** | Dissolved oxygen falls below the vet's floor | Standing remediation fires, such as a backup aerator | Duty keeper, immediately, then updates until it recovers |
| **3. Slow welfare signal** | An animal has not fed for three days | None | Queued for keeper review, same day |

Class 1 has no model in its path, because a door sensor reporting open is a fact. Class 3 is advisory because the situation moves over days, and that time is better spent on a keeper looking than a machine acting. Class 1 and 2 alerts require explicit acknowledgement and escalate on a timer, since an unacknowledged alert is one nobody has seen.

## The authority boundary

|  | May act on hardware | May raise, rank and explain |
|---|---|---|
| Threshold defined by the vet | Yes | Yes |
| Anything a model infers | **No** | Yes |

This holds even where a model is more accurate than the threshold beside it. A threshold can be audited and explained to a vet after an incident, whereas a model behaving strangely at three in the morning needs an expert the estate does not employ. Models exist to notice trouble earlier and describe patterns a person would take longer to see, always as a recommendation.

## The alerting unit differs by enclosure

Terrestrial alerts concern one animal that a keeper can go and inspect. Lagoon alerts concern the population, because separating one fish from a school is not realistic, so the alert says feeding response is abnormal and asks for observation at the next feed. Individual tracking still detects the pattern sooner, which earns its place as evidence rather than instruction. Piranha work is scoped to counting and tracking groups, and identifying individual fish across weeks is a later refinement that nothing here depends on.

## Knowing when the system is wrong

Every alert is closed with a disposition recorded by whoever handled it, either confirmed, not confirmed or unclear, stored with the readings that produced it and the threshold version in force. One unconfirmed alert is noise. Patterns are the signal, and two patterns mean different things.

| Pattern in unconfirmed alerts | Likely cause |
|---|---|
| The same animal, repeatedly | That individual's baseline is wrong, not the model |
| Many animals at once | Model drift, a fouled sensor, or an unrecorded change to the environment |

The unconfirmed rate is monitored as a measure of the system's own health. Dispositions accumulate as the only labelled ground truth available, and cost no extra staff time because closing an alert has to happen anyway.

## Sensitivity over precision

Missing a genuinely sick animal is worse than crying wolf, so every class is tuned to catch everything. The cost is that keepers who receive unfounded alerts stop reading them, and an ignored system misses animals just as surely. Queuing Class 3 keeps volume from becoming constant interruption, and if keepers start calling the alerts noise the fix is a better model or corrected baselines, never higher thresholds.

## Consequences

**Gains**

- A vet defines illness, keepers decide what to do, and software only moves information between them.
- One sentence settles every question about what may run unsupervised.
- Life safety survives both a failed uplink and a failed model.
- Evaluation data generates itself from work keepers already do.
- A wrong individual baseline and a drifting model are told apart rather than averaged together.

**Costs**

- Cases only a model would catch wait on a human seeing the alert, so response is slower.
- Threshold configuration across 55 enclosures and many species is ongoing work nobody has costed.
- Dispositions closed carelessly degrade the ground truth invisibly.
- Tuning for sensitivity sends keepers to healthy animals repeatedly, which erodes trust in the alerts.
- Individual intervention in the lagoon stays physically out of reach.

## Assumptions

- A veterinarian supplies thresholds and standing procedures per species and reviews them periodically.
- Where there is no in-house vet, a designated head keeper is first responder under those procedures.
- Security are on the estate during opening hours and reachable outside them.

## Related

- **ADR-040** established the tiered topology that lets Class 1 and 2 run without the uplink
- **ADR-042** defines the telemetry and delivery guarantees this consumes
- **ADR-044** covers behaviour when a gateway or the uplink is unavailable
- Diagram: `docs/diagrams/C-welfare-loop.drawio`
