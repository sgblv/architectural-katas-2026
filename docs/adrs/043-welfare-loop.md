# 043. Watching animal health, and what the system may do on its own

## Status

Proposed

## The decision

Fixed rules written by a vet can switch equipment on by themselves. Anything the AI works out on its own can only be a suggestion to a person.

## Why we need to decide this

Looking after the animals is expensive, and it gets much more expensive when one falls ill (P3). Today nobody records whether an animal has eaten (P9), and nobody knows how many piranhas there are (P6).

ADR-042 covers the readings coming out of the enclosures. This record covers what we are allowed to do with those readings.

AI does not behave the same way every time (C12). If it starts making bad calls at three in the morning, there is nobody at the estate who could work out why (C13). So we have to be clear about what it is allowed to touch.

## The vet decides what "unwell" means

A vet gives us the safe range for each species, the point at which we should worry, and what to do about it. We save those settings and keep the old versions, so we can always say which settings were in use when an alert went out.

The system never invents its own idea of a sick animal.

## Three kinds of problem

| Kind of problem | Example | Does anything happen automatically? | Who hears about it, and when |
|---|---|---|---|
| **People in danger** | A door opens on a venomous enclosure | No. It only raises the alarm | Security and the keeper on duty, straight away, using equipment on the estate |
| **Animal in danger now** | Oxygen in the lagoon drops below the vet's limit | Yes. The backup pump comes on | The keeper on duty, straight away, then updates until the reading recovers |
| **Animal may be unwell** | A fish has not fed for three days | No | Added to the keeper's list to check that day |

A door is either open or closed. There is nothing to interpret, so no AI is involved in that first row at all.

The third row develops over days. There is time for a person to walk over and look, so the system only suggests.

For the first two rows, someone has to confirm they have seen the alert. If nobody confirms within a set time, it goes to the next person on the list. An alert nobody has confirmed is an alert we assume nobody has seen.

## What the system may do without asking

|  | Can switch equipment on | Can raise an alert and explain it |
|---|---|---|
| A fixed limit set by the vet | Yes | Yes |
| Anything the AI works out | **No** | Yes |

We keep this rule even when the AI is better at spotting problems than the fixed limit is.

The reason is simple. If a pump comes on because oxygen dropped below a number the vet wrote down, we can explain that to anyone, afterwards, exactly. If it comes on because a model decided something, we would need someone who understands the model to explain it, and the estate does not employ that person.

So the AI's job is to spot trouble earlier than a fixed limit would, and to point out patterns a person would take longer to notice. It tells a human. The human acts.

## Fish are different from lizards

In a land enclosure, an alert is about one animal, and a keeper can go and look at that animal.

In the lagoon we cannot do that. Picking one fish out of a school and catching it is not realistic. So an alert about the lagoon says that feeding looks wrong across the group, and asks a keeper to watch the next feed.

We still track individual fish, because it spots the problem sooner than looking at averages would. But what a keeper receives is information, not an instruction to go and find fish number 47.

For the piranhas we are counting them and following groups over time. Recognising the same individual fish week after week may be possible later. Nothing here depends on it.

## Telling when we get it wrong

When a keeper deals with an alert, they close it with one of three answers: yes there was a problem, no there wasn't, or unclear. We save that answer next to the readings that caused the alert and the vet's settings at the time.

One wrong alert means nothing. A pattern of wrong alerts means something, and there are two patterns worth telling apart.

| What we see | What it usually means |
|---|---|
| The same animal keeps triggering false alerts | We have the wrong idea of normal for that particular animal |
| Lots of animals start triggering false alerts | Something has changed. The model has drifted, a sensor is dirty, or the enclosures changed and nobody told us |

We watch how often alerts turn out to be wrong. That number is how we know whether the system is still working.

Those keeper answers are also the only real evidence we have of what a sick animal looks like here, and collecting them costs nothing extra, because closing an alert has to happen anyway.

## We would rather be wrong than miss something

Missing a genuinely sick animal is worse than a false alarm, so we set everything to catch as much as possible.

This has a cost. Keepers who keep getting alerts about healthy animals stop reading them, and a system nobody reads misses sick animals too. Two things help. Slow alerts go on a list instead of interrupting anyone. And we watch how often we are wrong, so we notice trust slipping before keepers give up on it.

If keepers start calling the alerts noise, the answer is a better model or a corrected idea of normal for that animal. Making the system quieter by raising the limits would just hide the misses.

## What we chose, and what it cost us

| We chose | Instead of | What it costs us |
|---|---|---|
| A vet sets every limit | Letting the system learn on its own what normal looks like | Someone has to write and maintain settings for 55 enclosures, and there is no vet on staff by default |
| Only fixed rules can switch equipment on | Letting the AI act by itself when it is confident | The AI may spot a problem hours early and still wait for a person to read the alert |
| Alerts about the lagoon cover the whole group | Alerts about one named fish | We can say feeding looks wrong, but not which fish to catch |
| Alert whenever there is any doubt | Only alerting when the system is sure | Keepers will walk to healthy animals, and if it happens too often they stop trusting alerts |

## What this gives us

- The vet decides what illness is, keepers decide what to do, and the software only carries messages between them.
- One sentence answers every question about what runs on its own.
- If the internet goes down, or the AI fails, the safety alarms still work.
- We find out whether the AI is any good from work keepers already do.
- We can tell the difference between one odd animal and a system going wrong.

## Assumptions

- A vet supplies the limits and the procedures for each species, and reviews them from time to time.
- Where there is no vet on site, a senior keeper responds first, following those procedures.
- Security are on the estate during opening hours and reachable outside them.

## Related

- **ADR-040** set up the network that lets the safety alarms work without the internet
- **ADR-042** defines the readings this uses
- **ADR-044** covers what happens when the connection drops
- Diagram: 
