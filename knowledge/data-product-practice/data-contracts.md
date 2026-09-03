# Data Contracts

## What it is (3 sentences max)
A data contract is a versioned, machine-readable agreement between a
producing system and its consumers covering schema, semantics, freshness,
volume bounds and PII classification, checked in CI rather than agreed in a
meeting. In an AWS stack it usually lives as YAML or JSON Schema in the
producer's repo, registered in Glue Schema Registry or an open standard
like ODCS, and enforced at the ingestion boundary. The contract's real
function is to make a breaking change fail a producer's pipeline instead of
a consumer's regulatory report.

## Why a data PM cares
- It converts "who broke the balances feed" from a two-day forensic hunt
  into a failed build with a named owner.
- Contracts are the only practical control over a vendor core banking
  system you cannot modify — you assert what you expect and alarm on drift.
- Onboarding a new consumer becomes reading one file instead of three
  interviews, which is the difference between 2 days and 3 weeks.

## Decisions a PM actually makes
- Enforce at write vs validate at read: blocking bad records at ingest
  protects every downstream consumer but stops the pipeline during a
  month-end incident, quarantine-and-continue keeps the feed alive and
  leaves consumers reading incomplete data.
- Schema-only vs schema plus semantics: schema-only contracts are cheap and
  catch type breaks, adding value constraints (`currency IN (AED,USD,...)`,
  `amount >= 0`) catches the errors that actually reach the regulator.
- Producer-owned vs platform-owned contracts: producer ownership creates
  real accountability but means 12 teams learning the format, platform
  ownership ships faster and quietly recreates a central data team.
- Backward-compatible-only evolution vs versioned breaking releases:
  additive-only avoids consumer migrations forever but accumulates dead
  columns, versioned `v1`/`v2` tables are clean but double the storage and
  the operational surface for a quarter.
- Contract violation severity: failing the build on any breach gets
  attention but trains teams to disable checks, warn-then-fail after 3
  consecutive breaches survives contact with a real on-call rota.

## Failure modes
- Contract exists but is not enforced in CI — schema drifts for six weeks
  and is discovered when a consumer's IFRS 9 staging counts move.
- Nullability declared loosely to make the first release pass — the
  `customer_segment` column is 40% null in production and every
  segmentation metric is quietly biased.
- No volume bound — an upstream reload emits 12 million rows instead of
  400,000, all pass schema validation, and dashboards double overnight.

## Vocabulary
- **ODCS** — Open Data Contract Standard, a common YAML contract spec.
- **Schema registry** — service storing versioned schemas and compat rules.
- **Backward compatible** — new schema readable by old consumers.
- **Quarantine table** — sink for records failing contract validation.
- **Semantic constraint** — a rule about values, not types.
- **Producer** — the team that owns the emitting system.
- **Breaking change** — any change requiring consumer code edits.
