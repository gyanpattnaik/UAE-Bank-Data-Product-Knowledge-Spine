# Glue Catalog and Lake Formation Governance

## What it is (3 sentences max)
AWS Glue Data Catalog is the metastore holding database, table, column and
partition definitions; Lake Formation sits on top and enforces
table-, column-, row- and cell-level permissions plus LF-Tags. Together they
replace hand-written S3 bucket policies with grants an audit team can read.
In a multi-account bank landing zone, Lake Formation cross-account sharing
is what lets the risk account query the payments domain's tables without
copying them.

## Why a data PM cares
- Column-level masking of Emirates ID, IBAN and PAN is the control the
  CBUAE and internal DPO will ask to see demonstrated, not described.
- Every new consuming team is a grant request; without LF-Tags that becomes
  a 40-line ticket per table and a two-week onboarding lag.
- Glue crawlers silently redefine schemas, which is how a partition column
  becomes `string` after being `date` for a year.

## Decisions a PM actually makes
- LF-Tag-based access vs named-resource grants: tags scale to hundreds of
  tables and let you grant `Confidentiality=PII-Restricted` once, named
  grants are explicit and easier for an auditor to trace but do not scale.
- Crawler-managed schemas vs schemas declared in code: crawlers cover
  unknown incoming files, IaC-declared tables (CDK or Terraform) prevent
  drift and give you a reviewable diff — most banks end up crawler-free.
- Masking in Lake Formation vs tokenising at ingest: LF data filters keep
  one copy and are reversible for authorised roles, upstream tokenisation
  removes the raw value from S3 entirely and simplifies the PDPL story.
- Row-level filters vs separate tables per entity: filters keep one
  physical table for DIFC and onshore branches, separate tables make
  residency boundaries physically provable.
- Glue Catalog per account vs a central catalog with resource links:
  central is one lineage graph, per-account is a cleaner blast radius when
  one team's Terraform run drops a database.

## Failure modes
- IAMAllowedPrincipals left on a database after migration — Lake Formation
  grants appear correct in the console while S3 policies still permit
  everything; a read the DPO expected to be blocked returns full PAN.
- Crawler schema drift — a downstream Athena view fails with
  `HIVE_PARTITION_SCHEMA_MISMATCH` the morning after an upstream file adds
  a column.
- Partition explosion — a table partitioned by hour and account crosses
  ~1 million partitions and `GetPartitions` throttles, so scheduled queries
  time out at 30 minutes.

## Vocabulary
- **LF-Tag** — key-value label on catalog resources used to grant access.
- **Data filter** — Lake Formation row/column restriction on a table.
- **Resource link** — a catalog pointer to a table in another account.
- **IAMAllowedPrincipals** — legacy passthrough that bypasses LF grants.
- **Crawler** — Glue job that infers schema from S3 files.
- **Governed table** — deprecated LF table type; do not select it.
- **Partition index** — Glue index that keeps partition lookups fast.
