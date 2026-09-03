# S3 Lakehouse and Table Formats

## What it is (3 sentences max)
An S3 lakehouse stores banking data as Parquet with a table format —
Iceberg, Delta Lake, or Hudi — layering ACID transactions, schema evolution
and time travel over the object store. Iceberg is the AWS default because
Glue Catalog, Athena v3, Redshift and EMR read it natively. The table
format, not the file format, is what makes a lakehouse restatable.

## Why a data PM cares
- Time travel is your regulatory answer to "what did the balance look like
  on 31 Dec at cut-off?" — without it you rebuild from backups and lose two
  weeks.
- Table format choice locks your engine options for roughly three years;
  Hudi on Redshift Spectrum still means copy pipelines.
- Small-file counts drive Athena scan cost more than data volume does; a
  2 TB table in 40 million files costs more to query than 20 TB in 8,000.

## Decisions a PM actually makes
- Iceberg vs Delta: Iceberg gives broad AWS engine support and hidden
  partitioning, Delta gives better Databricks performance and a maturer
  merge path if the bank already runs Spark on Databricks.
- Copy-on-write vs merge-on-read: CoW gives fast reads for the daily
  regulatory extract, MoR gives cheap frequent upserts for a CDC feed
  landing every 5 minutes — you cannot have both on one table.
- Partition by transaction date vs booking date: transaction date suits
  fraud and ops queries, booking date suits finance reconciliation and
  IFRS 9 — pick one, expose the other as a sort column.
- Retention: 7-year hot Iceberg history satisfies auditors instantly but
  costs ~3x tiering to Glacier Instant Retrieval with a documented 4-hour
  restore SLA.
- Compaction cadence: hourly compaction keeps query latency flat but burns
  EMR/Glue compute continuously; nightly compaction is cheaper but leaves
  intraday queries scanning 10x more files.

## Failure modes
- Snapshot expiry misconfigured — an auditor asks for a Q2 point-in-time
  read and Athena returns `Table does not have a snapshot at timestamp`.
- Unmanaged small files — Athena p95 query time climbs from 12s to 90s over
  a quarter while row counts are flat.
- Schema evolution by column rename rather than add — downstream Redshift
  external tables return NULL columns silently, and the finance report
  totals drop without an error.

## Vocabulary
- **Manifest list** — the Iceberg file enumerating a snapshot's data files.
- **Snapshot expiry** — deleting old table versions; ends time travel.
- **Compaction** — rewriting many small files into fewer large ones.
- **Hidden partitioning** — Iceberg partitioning without a partition column
  in the query predicate.
- **Copy-on-write (CoW)** — rewrite whole files on update.
- **Merge-on-read (MoR)** — write delete/update deltas, merge at read time.
- **Glacier Instant Retrieval** — archive tier with millisecond reads.
