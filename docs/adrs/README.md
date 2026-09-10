# Architecture Decision Records

Status legend: **Written** has a Decision. **Draft on branch** has the problem drafted
and the Decision deliberately blank, sitting on a branch for its seat to take or discard.
**Planned** has no file yet.

| # | Title | Seat | Status |
|---|-------|------|--------|
| 000 | [Template](000-template.md) | — | — |
| 010 | [Edge-first data plane with a conventional transactional core](010-architecture-style.md) | A | Written |
| 011 | [AI determinism tiers, and using the lowest tier that works](011-ai-determinism-tiers.md) | A | Written |
| 012 | [Admissions and ticketing, with gates that work offline](012-admissions-and-ticketing.md) | A | Written |
| 020 | Model gateway and provider abstraction | B | Draft on branch |
| 021 | Evaluating AI before release | B | Draft on branch |
| 022 | Detecting AI misbehaviour in production | B | Draft on branch |
| 023 | AI cost control and graceful degradation | B | Draft on branch |
| 040 | [Tiered MQTT topology with prioritised local buffering](040-connectivity-topology.md) | C | Written |
| 042 | [Telemetry model, topic taxonomy, and delivery guarantees](042-telemetry-model.md) | C | Written |
| 043 | [Watching animal health, and what the system may do on its own](043-welfare-loop.md) | C | Written |
| 044 | Operating while disconnected, and reconciling afterwards | C | Planned |
| 045 | Presence and flow sensing, counting without identifying | C | Planned |
| 046 | Piranha population estimation against a stock ledger | C | Planned |
| 060 | Ride condition monitoring | D | Draft on branch |
| 061 | Demand forecasting and pricing | D | Draft on branch |
| 062 | Visitor concierge | D | Draft on branch |
| 063 | Estate operations copilot | D | Draft on branch |
| 064 | Retention and next-best-offer | D | Draft on branch |

041 is unused. 044 to 046 are C's to claim, name and write.

## Draft on branch — what that means

These are not on `main` on purpose. They live on two branches:

- `adr/020-029-context` — seat B
- `adr/060-069-context` — seat D

Each file has the problem, the constraints, the requirement identifiers and a list of
alternatives worth rejecting. Nothing has been decided; the Decision and Consequences
sections are empty.

If the framing is useful, merge the branch and finish the records. If it is not, delete
the branch and write your own — no explanation needed, and nothing is lost. They exist
so that starting is cheap, not to settle anything on your behalf.

Whoever merges, tell A so the index rows get links.

## Numbering

Ranges are reserved per seat so nobody collides. See [roles](../roles.md).

| Range | Seat |
|-------|------|
| 000–009 | Template and meta |
| 010–019 | A — architecture style, admissions |
| 020–039 | B — AI platform, evaluation, cost |
| 040–059 | C — connectivity, telemetry, welfare, presence |
| 060–079 | D — rides, pricing, concierge, copilot, retention |

Add your file, and let A add the index row. This file conflicts otherwise.

## Reading order

Start with [010](010-architecture-style.md) for the shape of the system, then
[011](011-ai-determinism-tiers.md) for how AI is bounded within it. Everything else
elaborates one half or the other.

## What makes an ADR done

- At least two rejected alternatives, with why each lost. This is scored explicitly.
- Numbers where numbers exist — thresholds, volumes, costs, limits.
- Requirement identifiers from [requirements](../requirements.md) cited inline, like
  `patchy wifi (C1)`.
- Status flipped from Proposed to Accepted at the cold read on Monday 14 September.
