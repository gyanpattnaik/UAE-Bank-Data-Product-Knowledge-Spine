# Data Product PRD

## What it is (3 sentences max)
A PRD for a data product describes the consumers and their decisions, the
interface (tables, APIs, or a semantic layer), the SLA, the source
dependencies, the governance classification, and the run cost, rather than
screens and user flows. Its acceptance criteria are contract terms and
quality thresholds, not UI behaviour. Sections that must appear: consumers
and use cases, interface, SLA, sources and lineage, governance, cost, and
explicit non-goals.

## Why a data PM cares
- Naming the decision each consumer makes is what prevents building a
  120-column table that serves nobody's actual query.
- Run cost stated up front is the only way a platform team can say no to a
  5-minute refresh on a dashboard viewed twice a week.
- Non-goals are the defence against a scope creep pattern specific to data:
  "while you're building it, can you add..."

## Decisions a PM actually makes
- Define success as adoption (weekly active consumers, queries per week) vs
  business outcome (provision accuracy, alert precision): adoption is
  measurable in week 2 and gameable, business outcome is the real bar and
  attributable only after a quarter.
- Ship a thin certified slice vs a complete domain model: a 12-column
  certified table lands in six weeks and forces consumers to ask for more,
  the full model takes two quarters and arrives against changed needs.
- Expose tables vs a semantic layer vs an API: tables are flexible and let
  every consumer redefine metrics, a semantic layer enforces definitions,
  an API suits operational consumers and is the most expensive to run.
- Include the regulatory reporting consumer in v1 vs defer: including them
  raises the quality and lineage bar for everyone from day one, deferring
  ships faster and usually means rebuilding.
- Cost ceiling stated as a hard cap vs a budget alert: a hard cap forces
  design trade-offs early and can block a legitimate spike, an alert is
  flexible and gets ignored.

## Failure modes
- Consumers listed as departments rather than named people with named
  decisions — no one signs off on acceptance and the product ships to
  silence.
- SLA agreed without checking the upstream source's own schedule — the
  06:00 target depends on a core extract that lands at 06:30.
- No non-goals section — three unplanned columns are added during build and
  the launch slips a sprint.

## Vocabulary
- **Consumer** — a named team with a stated decision and query pattern.
- **Interface** — the published access surface for the product.
- **Certified** — meets the platform's contract, quality and lineage bar.
- **Non-goal** — an explicitly excluded scope item.
- **Run cost** — monthly storage plus compute at expected query volume.
- **Thin slice** — the smallest columns set that answers the top decision.
- **Acceptance criteria** — contract terms plus quality thresholds.
