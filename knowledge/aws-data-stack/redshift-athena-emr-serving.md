# Redshift vs Athena vs EMR for Serving

## What it is (3 sentences max)
These are the three query surfaces over the same S3 lakehouse: Redshift for
governed, concurrent BI and finance reporting, Athena for ad-hoc and
low-frequency serverless SQL, EMR (Spark) for heavy transformation and ML
feature generation. Redshift Serverless and Athena both read Iceberg tables
in the Glue Catalog, so the choice is about concurrency, cost shape and
latency rather than data access. In practice a bank runs all three and the
PM decides which workload sits where.

## Why a data PM cares
- Cost shape differs, not just cost level: Athena bills per TB scanned,
  Redshift Serverless per RPU-hour, EMR per instance-hour — a dashboard
  refreshing every 5 minutes is cheap on one and ruinous on another.
- Concurrency limits decide whether 300 branch users can open a report at
  09:00; Athena defaults to 20-25 concurrent DML queries per account.
- BI tool behaviour matters more than benchmarks — Power BI DirectQuery on
  Athena generates repeated full scans that Redshift materialised views
  absorb.

## Decisions a PM actually makes
- Redshift Serverless vs provisioned RA3: serverless suits spiky
  month-end close with no idle cost, provisioned with reserved instances is
  30-50% cheaper for a steady 12-hour daily load.
- Redshift managed storage vs Spectrum/Iceberg external tables: managed
  storage gives predictable sub-second joins for the regulatory mart,
  external tables avoid a second copy and keep one lineage story.
- Athena for self-serve vs a curated Redshift semantic layer: Athena lets
  analysts move immediately but produces contradictory numbers, the
  semantic layer enforces one definition of `net_revenue` and adds a
  two-week queue.
- EMR on EC2 vs EMR Serverless vs Glue jobs: EMR on EC2 with Spot wins on
  long nightly jobs, EMR Serverless removes cluster ops for bursty work,
  Glue is simplest but harder to tune past ~1 TB shuffles.
- Workload isolation: separate Redshift workgroups per domain prevent
  finance close from starving fraud analytics, one workgroup is cheaper and
  easier to right-size.

## Failure modes
- Unpartitioned Athena scan — a single analyst query scans 4 TB and the
  monthly bill jumps by a four-figure amount with no performance signal.
- Redshift queue saturation — `WLMQueueLength` climbs at month-end and
  finance dashboards time out precisely during the close window.
- Spark small-file shuffle spill — an EMR job's stage runtime doubles
  week-over-week while input volume is flat; executors show disk spill in
  the Spark UI.

## Vocabulary
- **RPU** — Redshift Serverless compute unit, billed per hour.
- **Spectrum** — Redshift querying S3 external tables directly.
- **WLM** — Redshift workload management queues and concurrency scaling.
- **Materialised view** — precomputed result refreshed on schedule.
- **Data sharing** — Redshift cross-cluster read without copying.
- **Shuffle spill** — Spark writing intermediate data to disk.
- **Result reuse** — Athena serving a cached result for an identical query.
