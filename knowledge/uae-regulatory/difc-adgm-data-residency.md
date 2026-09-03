# DIFC/ADGM Data Protection and Data Residency

## What it is (3 sentences max)
DIFC and ADGM are financial free zones with their own GDPR-shaped data
protection regimes and their own commissioners, separate from the onshore
PDPL: DIFC Data Protection Law No. 5 of 2020, in force 1 July 2020 and
enforced from 1 October 2020, amended in 2022 and on 8 July 2025; and the
ADGM Data Protection Regulations 2021, in force 14 August 2021 for new
entities and 14 February 2022 for existing ones. Both are actively enforced
while the onshore regime awaits its executive regulations. A bank with an
onshore licence and a DIFC or ADGM entity is subject to more than one
regime at once, on different data.

## Why a data PM cares
- Entity of origin, not the AWS Region, determines which regime governs a
  record — one Redshift cluster can hold data under three regimes.
- Group-level analytics pooling onshore and free-zone customer data is a
  transfer question before it is an engineering question.
- ADGM requires controllers and processors to register with its Office of
  Data Protection, so "the group has a DPO" is not an answer; obligations
  attach per entity.

## Decisions a PM actually makes
- Separate accounts and buckets per entity vs one platform with row-level
  filters: separation makes residency provable to a commissioner and
  multiplies cost, filters are efficient and depend on Lake Formation
  configuration staying correct.
- Aggregate-only cross-entity reporting vs record-level pooling: aggregates
  sidestep the transfer question and limit analysis, record-level pooling
  needs a documented transfer mechanism under each regime.
- me-central-1 (UAE) vs me-south-1 (Bahrain) as primary Region: in-country
  is the simplest residency story with a narrower service catalogue,
  Bahrain has more services and puts data outside the UAE.
- In-country DR vs out-of-country: an in-UAE pairing keeps residency intact
  with limited options, out-of-country DR is operationally stronger and is
  a transfer that must be papered under each applicable regime.
- Tagging entity of origin at ingest vs deriving it later: tagging costs
  producer effort and makes residency enforceable, deriving it from
  account attributes is cheaper and wrong at the edges.

## Failure modes
- Entity attribution missing on a shared reference table — nobody can say
  which regime governs the customer dimension.
- A SaaS BI or observability tool egressing to a US Region — provable in
  flow logs and CloudTrail after the fact, not before.
- Free-zone data copied into an onshore sandbox for a PoC and never
  deleted; found in a later access review.

## Vocabulary
- **DIFC DP Law** — Data Protection Law No. 5 of 2020, as amended.
- **ADGM DPR 2021** — ADGM Data Protection Regulations 2021.
- **Office of Data Protection** — ADGM's authority; registration required.
- **Commissioner** — each free zone's data protection authority.
- **Entity of origin** — the licensed entity that collected the record.
- **me-central-1** — the AWS UAE Region.
- **Transfer mechanism** — the legal basis for exporting personal data.
