# 043. Animal welfare monitoring and the limits of automated action

## Status

Proposed

## Context

The estate holds more than 200 animals across 55 enclosures (C5), aquatic and terrestrial, several of them venomous, in spaces where physical access is restricted and hazardous (C6, C10). Care is expensive and becomes far more expensive when an animal falls ill (P3). Feeding is currently unmeasured, so nobody knows whether an animal ate or how well (P9), and the piranha population is not counted at all (P6).

ADR-042 defines the telemetry arriving from these enclosures. This record covers what happens to it. What counts as a problem, who is told, how quickly, and above all what the system may do without a person involved.

That last question is the important one. Animal welfare decisions carry real consequences, the estate runs on a small operations staff with no machine learning expertise on hand (C13), and model behaviour is not deterministic (C12). A design that lets a probabilistic system act unsupervised on living animals would be difficult to defend, and impossible to reason about when it goes wrong.

## Decision

### Thresholds and procedures come from a veterinarian

Safe ranges, alert thresholds and the response procedure for each species are supplied by a veterinarian, either in house or on call. They are configuration, not code, and they are versioned so it is always possible to say which thresholds were in force when an alert fired.

The system does not decide what unhealthy means. It applies a definition given to it by someone qualified to write one, and it carries out a procedure someone qualified has approved.

Where the estate has no in-house vet, a designated head keeper is the first responder and follows the standing procedure, escalating to the on-call vet when the procedure says to or when there is no improvement.

### Three classes of event

Events are handled by consequence and by how fast the situation moves, which mirrors the local and cloud split established in ADR-040.

**Class 1, immediate risk to people.** Containment state change on a venomous enclosure. Security and the duty keeper are notified simultaneously and immediately, on the estate, with no dependency on the uplink. The procedure is human from the first second: isolate the area, confirm whether an animal is missing, review camera footage, and widen the closed area if needed. No model sits in this path. A door sensor reporting open is a fact, and it is treated as one.

**Class 2, fast-moving risk to an animal's life.** Dissolved oxygen falling, a temperature excursion, filtration failure. A vet-defined threshold being crossed triggers the standing remediation automatically, such as bringing a backup aerator online. The duty keeper is notified at the same time and receives periodic updates. If the reading has not recovered within the interval the procedure specifies, the alert escalates and asks for physical intervention.

**Class 3, slow welfare signals.** An animal not feeding, a change in activity, a gradual shift in condition. These are advisory. They are queued for keeper review rather than pushed as interruptions, and nothing happens automatically. There is time here, measured in hours or days, and that time should be spent on a person looking rather than a machine acting.

### What the system may do on its own

The boundary is between a rule and an inference.

Automatic action is permitted only when the trigger is a deterministic threshold defined by the veterinarian and crossed by a directly measured value. Dissolved oxygen below the configured floor starts the aerator. That is a rule doing what a rule does, and its behaviour can be stated exactly in advance.

No output from a model actuates anything. Models notice problems earlier and describe patterns a person would take longer to see, and everything they produce arrives as a recommendation to a human. A model may raise the priority of an alert, add context to it, or surface a trend nobody asked about. It may not open a door, start a pump, or close a case.

This holds even where a model is clearly more accurate than the threshold it sits beside. The reason is not distrust of the model. It is that a threshold can be audited, explained to a vet, and reasoned about after an incident, and the estate has nobody on staff who could diagnose a model behaving strangely at three in the morning (C13).

### Escalation and acknowledgement

Every Class 1 and Class 2 alert requires an explicit human acknowledgement. Unacknowledged alerts escalate on a timer to the next contact in the chain. An alert that nobody has acknowledged is treated as an alert nobody has seen, regardless of what was delivered.

Class 1 alerts are raised and escalated on the estate itself, so a failed uplink cannot delay them.

### The unit of alerting differs by enclosure

For terrestrial enclosures, an alert concerns an individual animal, and a keeper can go and look at that animal.

For the lagoon, the actionable alert concerns the population. Identifying one fish in a school and separating it for examination is not realistic, so an alert reads that feeding response across the lagoon is abnormal and asks a keeper to observe at the next feed. Individual-level tracking is still worth doing because it detects the pattern sooner and more precisely than a population average would, but its output is evidence for a human rather than an instruction to act on one animal.

Population counting for the piranhas is scoped to counting and to tracking groups over time. Reliable identification of individual fish across weeks is treated as a possible later refinement and is not something this design depends on.

### How we find out the system is wrong

Every alert is closed with a disposition recorded by the person who handled it: confirmed, not confirmed, or unclear. That disposition is stored with the alert, the readings that produced it, and the threshold version in force at the time.

A single unconfirmed alert is noise and is expected. Patterns in unconfirmed alerts are the signal, and two patterns mean different things.

Repeated unconfirmed alerts on the same animal usually mean the baseline for that individual is wrong rather than the model. An old animal that moves less than its species norm is not sick, and its normal needs adjusting.

Unconfirmed alerts rising across many animals at once usually mean something systemic. The model has drifted, a sensor has fouled, or the environment has changed in a way nobody recorded. That is the condition worth alarming on, and the rate of unconfirmed alerts is monitored for exactly this reason.

Keeper dispositions accumulate as labelled examples, which is the only source of ground truth this system has.

### False alarms against missed animals

The estate's position is that missing a genuinely sick animal is worse than crying wolf. All three classes are tuned toward catching everything, and unconfirmed alerts are accepted as the price.

For Class 1 and Class 2 this is uncontroversial. A containment alert that turns out to be a keeper propping a door open costs somebody a walk across the estate, and missing a real one is unacceptable in a way that needs no explanation.

For Class 3 the same choice carries a known failure mode. Keepers who receive a stream of unfounded welfare alerts stop reading them, and an ignored system misses sick animals just as surely as one tuned too conservatively. Two things reduce that risk without removing it. Class 3 alerts are queued for review rather than pushed as interruptions, so volume does not become constant interruption. And the rate of unconfirmed alerts is monitored as a measure of the system's own health.

If that rate climbs to the point where keepers describe the alerts as noise, the correct response is to improve the model or the individual baselines. Raising thresholds would quietly trade the noise for missed animals, which is the outcome this choice exists to avoid.

## Consequences

### Positive

Responsibility sits where the expertise is. A vet defines what unhealthy means, keepers decide what to do about it, and the software moves information between them quickly and reliably.

The authority boundary is a single sentence anyone can check a design against. Rules act, models advise. There is no case where somebody has to work out whether a given piece of automation was allowed.

Life safety does not depend on the uplink or on a model. A containment breach reaches security through local infrastructure, triggered by a physical sensor.

The system produces its own evaluation data. Keeper dispositions are labelled examples generated by work that has to happen anyway, so measuring whether the model still works costs nothing extra in staff time.

Two different failure modes are separated. An individual with a wrong baseline and a model that has drifted look identical in a raw false-alarm count, and telling them apart is what makes the alerting worth maintaining rather than worth ignoring.

### Negative

Cases that only a model would catch are delayed by a human step. Where a model spots deterioration earlier than any threshold does, the response still waits on a person seeing the alert. We accept slower response in exchange for behaviour that can be explained afterwards, and that trade is not free.

Threshold configuration is ongoing work for someone. Fifty-five enclosures, many species, and per-individual baselines add up, and there is no vet on staff by default. Nobody has costed this.

Keeper discipline carries the feedback loop. If dispositions are recorded carelessly, or alerts are closed in bulk at the end of a shift, the ground truth degrades quietly and every measurement built on it becomes wrong without anyone noticing.

Tuning toward sensitivity means keepers will walk to enclosures where nothing is wrong, and some of those walks will be wasted repeatedly on the same animal until its baseline is corrected. Over time that erodes trust in the alerts, which is the failure this choice is most exposed to. Queuing Class 3 alerts and watching the unconfirmed rate makes the erosion visible early, but neither prevents it.

Individual-level intervention in the lagoon remains out of reach. Better detection does not solve the physical problem of separating one fish from a school, and this design does not pretend it does.

### Assumptions

A veterinarian, in house or on call, will supply thresholds and standing procedures per species and review them periodically.

Where no in-house vet exists, a designated head keeper acts as first responder under those standing procedures.

Security personnel are available on the estate during opening hours and reachable outside them.
