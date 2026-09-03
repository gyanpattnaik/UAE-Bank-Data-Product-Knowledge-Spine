# AML/KYC and Transaction Monitoring

## What it is (3 sentences max)
This domain covers customer due diligence records (identity, beneficial
ownership, PEP and sanctions screening, risk rating) and the transaction
monitoring system that generates alerts against typology rules and models.
Vendor platforms are typically Actimize, Oracle FCCM, or SAS AML, fed by a
harmonised transaction and party feed from the lake. In the UAE, confirmed
suspicion is reported to the FIU through the goAML platform.

## Why a data PM cares
- Monitoring rules are only as good as the party data behind them; a
  customer whose accounts are not linked to one party ID is invisible to
  aggregation-based typologies.
- False positive rates of 90-98% are normal, so the measurable product goal
  is alert quality and investigator throughput, not alert volume.
- Every rule threshold change is a model-governance event requiring
  documented tuning evidence and above-the-line/below-the-line testing.

## Decisions a PM actually makes
- Feed the monitoring engine from the lake vs directly from the core:
  lake-sourced gives one harmonised, quality-checked feed and adds hours of
  latency, direct-from-core is fresher and duplicates transformation logic
  in a vendor tool nobody can audit.
- Real-time screening vs batch: real-time sanctions screening at payment
  initiation blocks bad payments and needs sub-second p99, end-of-day batch
  is far cheaper and detects only after settlement.
- Rules-only vs rules plus ML risk scoring: rules are explainable to the
  regulator line by line, an ML score cuts false positives materially and
  requires model risk documentation and ongoing validation.
- Threshold tuning cadence: quarterly tuning tracks changing behaviour and
  consumes analyst capacity plus a governance cycle, annual tuning is
  cheap and leaves stale thresholds through a period of growth.
- Retention of alert and investigation data: keeping full case history in
  the lake enables tuning analytics and expands the sensitive-data
  footprint, keeping only outcomes is cleaner and makes back-testing
  impossible.

## Failure modes
- Party resolution failure — one customer holds accounts under two party
  IDs after a name-format difference; structuring across the two accounts
  never triggers an aggregation rule.
- Feed gap during a migration — a product's transactions stop reaching the
  monitoring engine for weeks; discovered when alert volume for that
  product drops to zero on a trend chart nobody was watching.
- Screening list staleness — the sanctions list is refreshed weekly rather
  than on publication and a newly listed entity transacts in the gap.

## Vocabulary
- **CDD / EDD** — customer due diligence, enhanced for high-risk parties.
- **PEP** — politically exposed person.
- **goAML** — UAE FIU reporting platform for STRs and SARs.
- **STR / SAR** — suspicious transaction / activity report.
- **Typology** — a documented pattern of illicit behaviour a rule targets.
- **Above-the-line testing** — validating alerts just above a threshold.
- **Structuring** — splitting amounts to stay under a reporting threshold.
