# Metric Tree for a Data Product

## What it is (3 sentences max)
A metric tree decomposes one top metric into the inputs that mathematically
produce it, each owned and instrumented, so a movement in the top number
resolves to a specific branch. For a data product the top metric is usually
decision impact — provision accuracy, alert precision, time-to-insight —
with adoption and reliability as supporting branches. It is a diagnosis
tool, not a dashboard wishlist.

## Why a data PM cares
- It stops the "we track 40 KPIs" conversation by forcing every metric to
  justify its position as a driver of something above it.
- When a number moves, the tree tells you which branch to open first, which
  is the difference between a 20-minute and a 2-day investigation.
- It exposes metrics you cannot currently measure, which is a build backlog
  rather than an analysis failure.

## Decisions a PM actually makes
- Top metric as business outcome vs platform outcome: a business outcome
  (Stage 2 provision variance, false positive rate) earns executive
  attention and is confounded by factors you do not control, a platform
  outcome (SLA attainment) is fully attributable and reads as
  self-serving.
- Additive decomposition vs multiplicative: additive branches are easy to
  read and hide rate effects, multiplicative (volume × rate × value) makes
  the driver obvious and is harder to explain in a steering committee.
- Instrument adoption via query logs vs a survey: CloudTrail and Redshift
  query logs give exact usage per consumer with no goodwill cost, surveys
  capture perceived value and arrive quarterly at best.
- Include cost per query as a branch vs keep cost separate: including it
  forces efficiency into the product conversation and invites cost-cutting
  arguments against quality, separating it keeps focus and lets run cost
  drift.
- Refresh cadence of the tree itself: monthly review keeps it live and
  consumes a meeting, quarterly is cheap and lets a stale branch survive an
  entire roadmap cycle.

## Failure modes
- Branches that do not sum or multiply to the parent — a movement in the
  top metric has no explanation in the tree and the tree is abandoned.
- Every leaf owned by the data team — nothing in the tree changes as a
  result of anyone else's behaviour.
- Adoption measured as distinct users rather than repeat users; the number
  rises with each onboarding email and falls silently as people stop
  returning.

## Vocabulary
- **Top metric** — the single number the product exists to move.
- **Driver** — an input that mathematically changes the parent.
- **Leaf** — a directly instrumented, owned metric.
- **Decomposition** — the arithmetic linking a parent to its children.
- **Guardrail metric** — a value that must not degrade while others improve.
- **Attribution window** — the lag before an effect is measurable.
- **Time-to-insight** — elapsed time from consumer question to answer.
