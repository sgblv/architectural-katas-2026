# 040. Tiered MQTT topology with prioritised local buffering

## Status

Proposed

## Context

The Von Digitalis estate is large, sprawling, and has patchy wifi coverage ([C1](https://github.com/Humanberto/architectural-katas-2026/blob/adr/040-connectivity/docs/requirements.md#:~:text=C1,park%20is%20patchy)). There is budget for MQTT-capable hardware devices installed throughout the park ([C3](https://github.com/Humanberto/architectural-katas-2026/blob/adr/040-connectivity/docs/requirements.md#:~:text=C3,park%20is%20funded)), cloud services are permitted, but the path from the estate to the cloud has to be designed rather than assumed ([C2](https://github.com/Humanberto/architectural-katas-2026/blob/adr/040-connectivity/docs/requirements.md#:~:text=C2,designed%2C%20not%20assumed)).

The estate must monitor 200+ animals across 55 enclosures ([C5](https://github.com/Humanberto/architectural-katas-2026/blob/adr/040-connectivity/docs/requirements.md#:~:text=C5,displays%20and%20enclosures)), a mix of aquatic and land-based species, several of them venomous, where physical access is restricted and hazardous ([C6](https://github.com/Humanberto/architectural-katas-2026/blob/adr/040-connectivity/docs/requirements.md#:~:text=C6,animals%3B%20some%20poisonous), [C10](https://github.com/Humanberto/architectural-katas-2026/blob/adr/040-connectivity/docs/requirements.md#:~:text=C10,restricted%20and%20hazardous)). It must also measure how visitors move through 40 rides and the grounds ([C4](https://github.com/Humanberto/architectural-katas-2026/blob/adr/040-connectivity/docs/requirements.md#:~:text=C4,40%20rides)) to answer the Countess's question about which parts of the estate are popular ([P2](https://github.com/Humanberto/architectural-katas-2026/blob/adr/040-connectivity/docs/requirements.md#:~:text=P2,deployment%20are%20guesswork)).

Day-to-day technology is run by a single internal IT staff member, with an outsourced team billed hourly for emergencies. Any topology we choose has to be operable by one person on a normal day.

Three options were considered.

**A. Sensors publish directly to a cloud IoT service.** Simplest topology, no on-site infrastructure to own. Rejected: it makes every reading dependent on wifi that is known to be unreliable, and it means nothing on the estate works when the uplink is down. A tank heater failure at 2am would go unnoticed until connectivity returned.

**B. A single central broker on the estate, all sensors publishing to it directly.** Better, but it assumes every sensor across a sprawling estate can reach one radio. Rejected on physics: range and obstruction make full coverage from a single point unrealistic, and it concentrates all risk in one device.

**C. A tiered topology — chosen.** Sensors publish to nearby satellite brokers; satellites bridge to a central broker; the central broker feeds an on-premise server; the server syncs opportunistically to the cloud.

## Decision

We will deploy a three-tier MQTT topology with local processing and prioritised, opportunistic cloud sync.

**Tier 1 — sensors.** Devices in enclosures, on rides, and at path pinch-points publish to the nearest satellite. They hold no responsibility beyond publishing and buffering their own recent readings.

**Tier 2 — satellite brokers.** Distributed across the park so every sensor is within reliable radio range with minimal physical obstruction. Each satellite bridges to the central broker and buffers locally when that link is unavailable.

**Tier 3 — central broker and on-premise server.** Located centrally and hard-wired to the estate server. Performs pre-processing, then hands data to the server for processing, analysis, and local decision-making.

**Cloud sync is opportunistic and prioritised.** The server packages data and ships it to the cloud when connectivity allows, ordering the outbound queue by priority rather than by arrival time.

### What runs where

The split is by consequence and time horizon, not by data type.

| Runs on the estate | Runs in the cloud |
|---|---|
| Enclosure environment control and alarms | Visitor movement and behaviour analysis |
| Security and access control | Long-horizon animal welfare trends |
| Feeding events and immediate welfare alerts | On-premise purchase and revenue analysis |
| Anything with a safety consequence in minutes | Anything measured in hours or days |

The rule: **if a failure to act within minutes has a safety or revenue consequence, it runs locally.** A tank heater failing at 2am cannot wait on an uplink. Footfall analysis can wait until morning.

Out of scope for this ADR: whether and how AI-derived instructions may travel back down the tiers to actuate enclosure controls. That is an authority-boundary decision and is recorded separately (see ADR-043 and the AI platform ADRs).

## Consequences

### Positive

- **The estate keeps working when the uplink does not.** Safety-critical monitoring, alarms, and access control have no cloud dependency.
- **No readings are lost to a wifi outage.** Buffering at Tier 1 and Tier 2 turns lost data into delayed data.
- **Sensors are cheap and dumb.** Range requirements are satisfied by satellite placement rather than by expensive long-range devices, which keeps us inside the funded hardware budget ([C3](https://github.com/Humanberto/architectural-katas-2026/blob/adr/040-connectivity/docs/requirements.md#:~:text=C3,park%20is%20funded)).
- **Prioritisation makes a weak uplink usable.** When bandwidth is scarce it is spent on what matters, instead of first-in-first-out.
- **Adding an enclosure is a local change.** A new sensor joins its nearest satellite; nothing upstream is reconfigured.

### Negative

- **We now own physical hardware.** Satellites and the central broker need power, weatherproofing, patching, and eventual replacement, on an estate with one internal IT person. Every satellite is another device that can fail quietly in a hedge.
- **The central broker is a single point of failure.** Everything upstream of Tier 2 depends on it. Mitigating this properly means redundancy we have not yet costed, and until then a central broker outage degrades the whole estate. This is the largest open risk in this decision.
- **Satellite placement is a physical survey problem.** Coverage depends on range and line of sight, so siting cannot be decided from a map alone and will need adjusting after installation.
- **Data in the cloud is stale by an unpredictable amount.** Any cloud-side analysis must tolerate gaps and late arrivals, and must not be used for anything time-critical.
- **Two places where logic lives.** Local and cloud processing must be kept consistent, which is ongoing effort rather than a one-time cost.
- **Prioritisation needs a policy someone maintains.** A queue ordered by priority is only as good as the classification behind it, and that classification will drift as the estate adds features.

### Assumptions

- One internal IT staff member handles day-to-day operations, with outsourced support billed hourly for emergencies.
- The funded MQTT hardware budget covers satellites and the central broker; anything beyond that requires a separate case.
