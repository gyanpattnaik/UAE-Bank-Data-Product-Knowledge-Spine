# Streaming Ingestion (Kinesis/MSK) vs Batch (DMS)

## What it is (3 sentences max)
Streaming ingestion moves events continuously through Kinesis Data Streams
or Amazon MSK, typically fed by Debezium CDC off the core banking database
or by the payments switch emitting ISO 20022 messages. Batch ingestion uses
AWS DMS in full-load-plus-CDC mode, or a nightly extract, landing files on
S3 on a schedule. Most banks run both: DMS for the 400-table core, MSK for
the handful of feeds where seconds matter.

## Why a data PM cares
- Fraud and transaction monitoring lose detection value past roughly
  60 seconds; balances and month-end reporting do not, so latency is a
  per-feed decision, not a platform decision.
- MSK Provisioned at three brokers plus MSK Connect is a five-figure annual
  floor before a single consumer exists — the business case has to name the
  feeds that justify it.
- DMS CDC breaks on core-banking schema changes and the vendor gives you
  little notice; that is a change-management dependency you own.

## Decisions a PM actually makes
- MSK vs Kinesis Data Streams: MSK gives Kafka compatibility, existing
  Debezium connectors and 7-day-plus retention, Kinesis gives no broker
  management and per-shard billing but caps at 1 MB/s ingest per shard.
- DMS full load plus CDC vs vendor-supplied nightly extract: DMS gives
  near-real-time change capture but puts read load and log-retention
  demands on the core, the extract is contractually supported and slower.
- Exactly-once vs at-least-once with idempotent Iceberg merge: exactly-once
  costs throughput and complexity, at-least-once plus a merge on a natural
  key is what most payment feeds actually ship.
- Micro-batch every 5 minutes vs true streaming: five-minute Glue streaming
  writes keep file sizes sane and cost predictable, sub-second Flink on
  KDA meets fraud SLAs at several times the run rate.
- Replay window: 7-day Kafka retention lets you rebuild a bad consumer
  deploy without touching the source, 24-hour retention halves storage but
  makes every incident a source-system re-extract.

## Failure modes
- Hot partition key — hashing on branch code instead of account ID sends
  60% of traffic to one shard; `WriteProvisionedThroughputExceeded` spikes
  while average utilisation reads 20%.
- Silent DMS CDC stall — the task shows `Running` but
  `CDCLatencySource` climbs past 3,600 seconds and the balances table is a
  day stale with no alert fired.
- Out-of-order events with no watermark — reversal arrives before the
  original authorisation, and the daily net position is wrong by the value
  of the reversals.

## Vocabulary
- **CDC** — change data capture from the source database transaction log.
- **Debezium** — open-source CDC connector, usually run on MSK Connect.
- **Shard / partition** — unit of parallelism and ordering guarantee.
- **Watermark** — the event-time threshold a stream job treats as complete.
- **Dead-letter queue** — sink for events that fail processing.
- **CDCLatencySource** — DMS metric for lag between source and DMS.
- **Tombstone** — a null-payload record marking a deleted key.
