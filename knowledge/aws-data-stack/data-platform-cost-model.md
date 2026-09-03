# Data Platform Cost Model

## What it is (3 sentences max)
The run cost of a data product decomposed by billing unit — S3 GB-months,
Glue DPU-hours at $0.44, Athena per-TB scanned at $5, Redshift RPU-hours,
MSK broker-hours — plus the cost allocation tags that attribute it to a
domain. Cost shape differs by engine, not just level, so one workload can
be trivial on Athena and ruinous on Redshift. Cost Explorer with per-product
tags is how it is measured; list prices vary by Region and me-central-1 is
not us-east-1.

## Why a data PM cares
- A stated cost ceiling is the only basis for declining a 5-minute refresh
  on a dashboard opened twice a week.
- A PRD shipped without a run-cost number gets approved and then killed in
  the next quarterly cost review, after the build.
- Unit cost — cost per 1,000 queries, or per TB served — is the number that
  survives volume growth; absolute monthly spend is not.

## Decisions a PM actually makes
- Athena vs Redshift for a recurring workload: per-TB-scanned is cheap for
  infrequent large scans and punishing for a high-frequency refresh,
  RPU-hours are cheap under sustained concurrency and wasted while idle —
  the crossover sits around a query running several times an hour.
- Reserved capacity vs on-demand: a 1-year Redshift reservation or Compute
  Savings Plan takes 30-50% off and commits before the workload is known,
  on-demand keeps optionality at full list price.
- Tag granularity: per-data-product cost allocation tags give real
  accountability and require tagging enforced at resource creation, per-
  account tagging is automatic and hides which product drove the spend.
- Lifecycle tiering: moving regulatory history to Glacier Instant Retrieval
  after 90 days cuts storage materially and adds a retrieval charge on
  every audit request, S3 Standard throughout is simple and pays full rate
  on seven years of cold data.
- Fund a tuning workstream vs optimise opportunistically: a funded quarter
  of partitioning and compaction work usually cuts scan cost several-fold
  and consumes capacity that would otherwise ship a data product.

## Failure modes
- Untagged resources — a large share of the platform bill lands in
  "unallocated" and no domain accepts it at the quarterly review.
- Non-production left running — Redshift Serverless RPU-hours accrue
  overnight in the dev account; visible only as a monthly bill that rises
  while workload is flat.
- Estimate built at us-east-1 list price with no Region adjustment — month
  one exceeds the approved business case and the variance is structural,
  not usage-driven.

## Vocabulary
- **DPU-hour** — Glue billing unit for ETL compute.
- **RPU-hour** — Redshift Serverless billing unit.
- **Bytes scanned** — Athena's billable metric, per query.
- **Cost allocation tag** — tag surfaced in Cost Explorer for attribution.
- **Savings Plan** — commitment discount on sustained compute.
- **Unallocated spend** — cost no tag maps to a product or domain.
- **Unit cost** — spend normalised by queries served or TB delivered.
