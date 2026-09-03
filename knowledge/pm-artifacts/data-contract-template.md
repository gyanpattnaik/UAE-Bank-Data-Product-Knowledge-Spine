# Data Contract Template

## What it is (3 sentences max)
A YAML file checked into the producing domain's repo declaring identity,
owner, schema with types and nullability, semantic constraints, freshness
and volume SLAs, PII classification per column, and the version policy.
Required blocks: `id`, `owner`, `version`, `schema`, `quality`, `sla`,
`classification`, `distribution`, `version_policy`. It is validated in CI
and registered to Glue Schema Registry on merge.

## Why a data PM cares
- The template is the enforcement point for platform standards — if
  `classification` is a required field, no table reaches production
  unlabelled.
- It fixes the escalation path: `owner` plus `on_call` means an incident
  has an assignee in the first minute.
- Consumers listed in `distribution` give you the impact list for any
  proposed change without running lineage.

## Decisions a PM actually makes
- Required vs optional `classification` per column: required forces
  producers to think about Emirates ID and PAN at design time and slows the
  first release, optional ships faster and leaves gaps the DPO finds later.
- Single contract per table vs per data product: per-table is granular and
  proliferates files, per-product groups related tables and hides
  table-level SLA differences.
- Encode SLA numerically (`freshness_minutes: 60`) vs prose: numeric is
  machine-checkable and forces an uncomfortable commitment, prose is easy
  to agree and unenforceable.
- Include sample values vs schema only: samples make the contract readable
  and risk committing real customer data to git, schema-only is safe and
  less useful to a new consumer.
- Version bump policy: semantic versioning with a mandatory 30-day dual-run
  on major bumps protects consumers and slows the producer, immediate
  cutover is fast and breaks someone.

## Failure modes
- Contract merged with `owner` set to a team alias that routes nowhere —
  the first incident page bounces.
- `quality` block copied between contracts unchanged — every table asserts
  the same three generic rules and none of them are meaningful.
- Freshness declared against pipeline finish time rather than source event
  time, so the contract is met while the data is stale.

## Vocabulary
- **id** — stable identifier, `domain.product.table`.
- **distribution** — declared consumers and access mechanism.
- **classification** — per-column sensitivity label driving LF-Tags.
- **sla** — freshness, availability and volume commitments.
- **quality** — declarative rules, usually DQDL or Great Expectations.
- **dual-run** — old and new versions published in parallel.
- **deprecation date** — when the previous version stops publishing.
