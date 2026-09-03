# Data Quality SLAs and Observability

## What it is (3 sentences max)
A data quality SLA states measurable guarantees for a data product —
freshness, completeness, accuracy, uniqueness — with a stated measurement
window and a named consequence when missed. Observability is the tooling
that measures it: AWS Glue Data Quality (DQDL rules), Deequ on EMR, or
Monte Carlo / Soda for anomaly detection, emitting to CloudWatch and an
on-call rota. The SLA is a product commitment; observability is only the
instrument.

## Why a data PM cares
- Without a stated freshness SLA every stakeholder assumes real time and
  every delay becomes an escalation.
- Regulatory feeds have hard external deadlines — CBUAE and internal
  finance close do not accept "the pipeline was slow", so the SLA has to be
  set below the deadline with buffer.
- Alert volume is a product decision: a team that gets 60 data alerts a
  week stops reading them within a month.

## Decisions a PM actually makes
- SLA target level: 99.5% on-time daily delivery allows roughly one miss
  per quarter and is achievable on a shared platform, 99.9% forces
  redundant ingestion paths and roughly doubles run cost.
- Rule-based checks vs ML anomaly detection: DQDL rules are explainable to
  an auditor and cheap, anomaly detection catches the drift nobody thought
  to write a rule for and generates false positives during Ramadan and
  Eid volume shifts.
- Block vs publish-with-warning on a failed check: blocking protects the
  regulatory report and delays every other consumer, publishing with a
  quality flag lets ops proceed and risks a bad number in a board pack.
- Row-level vs aggregate checks: row-level catches individual bad records
  at high compute cost, aggregate checks (row count within ±15% of trailing
  7-day mean) catch systemic breaks cheaply and miss single-record errors.
- Who owns the pager: producing domain teams fix root causes but resist
  24/7 rotas, a central platform rota responds faster and never fixes the
  underlying source.

## Failure modes
- Freshness measured at pipeline completion rather than source event time —
  the dashboard shows green while the data is 9 hours behind the core.
- Checks written only for the happy path — a feed goes to zero rows and
  passes every rule because nothing asserts a minimum count.
- No SLA for the derived layer — the raw ingestion meets its 06:00 target,
  the transformation runs at 11:00, and the consumer's real experience is
  never measured.

## Vocabulary
- **DQDL** — Glue Data Quality Definition Language rule syntax.
- **Freshness** — lag between source event time and data availability.
- **Completeness** — proportion of expected records actually present.
- **Deequ** — AWS open-source Spark data quality library.
- **Error budget** — allowable SLA misses in a period before escalation.
- **Circuit breaker** — automatic halt of publication on a failed check.
- **Trailing baseline** — historical window used to judge today's volume.
