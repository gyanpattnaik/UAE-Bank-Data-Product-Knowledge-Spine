# PRD: Settled Card Transactions Data Product

> Worked example generated from `/knowledge`. Fictional institution
> ("Reference Bank UAE"); no real institution's data or staff.
> Template: `knowledge/pm-artifacts/data-product-prd.md`.
> Contract: [`data-contract.yaml`](./data-contract.yaml) — `cards.card_transactions.settled` v1.0.0

**Status:** Draft for sign-off · **Owner:** Head of Cards Analytics ·
**Target:** v1.0.0 certified, 8 weeks from kickoff

---

## 1. Consumers and use cases

Named decision per consumer. A consumer listed as a department with no
stated decision does not get built for
(`knowledge/pm-artifacts/data-product-prd.md`).

| Consumer | Decision they make | Query pattern | Sign-off |
|---|---|---|---|
| Head of Cards Analytics | Which MCC segments to target for the next spend campaign; where portfolio spend is shifting | Weekly aggregate over 13 months | Accountable owner |
| Finance — Cards Controller | Signs the monthly interchange accrual; reconciles scheme settlement to GL | Monthly, full-period scan, value-level | Required |
| Head of Financial Crime Analytics | Whether card-present typology thresholds are catching structuring across a party's cards | Daily incremental feed to the monitoring engine | Required |
| Retail Analytics (self-serve) | Segment-level spend behaviour for retail propositions | Ad-hoc, masked, Athena | Informed |

**Trade-off taken.** Financial Crime is in v1, not deferred. Including the
regulatory consumer from day one raises the quality and lineage bar for
everyone and adds roughly two weeks to the build; deferring would have
shipped faster and meant rebuilding the feed against a higher standard
later (`knowledge/pm-artifacts/data-product-prd.md`).

## 2. Interface

Iceberg table in the Glue Catalog, accessed through Lake Formation grants.
Athena and Redshift Spectrum read it directly; the Financial Crime feed is
a daily Glue job writing to the monitoring engine's landing zone.

**Trade-off taken.** Tables, not a semantic layer, for v1. Tables let the
four consumers move immediately and accept that each may define
"spend" differently in their own layer; a semantic layer would enforce one
definition and add a quarter before anyone gets data
(`knowledge/pm-artifacts/data-product-prd.md`). Mitigation: `settled_amount`
and `aed_equivalent_amount` are the only sanctioned amount columns, and the
metric definitions are published in the catalog glossary.

**Trade-off taken.** Copy-on-write, partitioned on `business_date`. CoW
gives fast reads for the monthly finance scan at the cost of expensive
updates, which is correct for a daily batch that rarely restates. Business
date aligns to finance and the GL and mis-sorts against operational
reality at cutover; `transaction_timestamp_utc` is available as a sort
column for ops queries
(`knowledge/aws-data-stack/s3-lakehouse-table-formats.md`).

## 3. SLA

| Dimension | Commitment |
|---|---|
| Freshness | Settled legs for business date D published by **06:00 GST on D+3** |
| Measured from | Settlement file cycle date — **not** pipeline completion |
| On-time target | 99.5% monthly (≈ one permitted miss per quarter) |
| Query availability | 99.9% |
| Breach notification | 30 minutes to `#cards-data-incidents` |

**Trade-off taken.** 99.5%, not 99.9%. 99.5% is achievable on the shared
platform with a single ingestion path; 99.9% would require a redundant path
around the scheme SFTP drop and roughly doubles run cost for a product
whose consumers all work on D+3 or later anyway
(`knowledge/data-product-practice/data-quality-slas-observability.md`).

**Trade-off taken.** Freshness measured against the settlement file cycle
date rather than job completion. This is the harder commitment — it makes a
late upstream file our SLA miss, not the scheme's — and it is the only
version of the metric that reflects what the consumer experiences
(documented failure mode, same source).

## 4. Sources and lineage

| Source | Mechanism | Cadence |
|---|---|---|
| Visa BASE II clearing files | SFTP drop → S3 raw | Daily |
| Mastercard IPM clearing files | SFTP drop → S3 raw | Daily |
| Core banking posting record | DMS full load + CDC | Continuous |
| `cards.settlement_file_control` | Scheme file totals for reconciliation | Per file |
| Card-to-token mapping | Tokenisation service at ingest | Continuous |

Lineage emitted via OpenLineage from the Glue jobs into the catalog, at
table level. **Trade-off taken:** table-level, not column-level.
Table-level answers impact analysis, which is the daily need; column-level
would answer an auditor's provenance question directly and costs several
times more to build and maintain — revisit if an audit finding lands
(`knowledge/data-product-practice/lineage-and-catalog.md`).

**Dependency risk.** DMS CDC on the core breaks on vendor schema changes
with little notice, and `CDCLatencySource` can climb while the task still
reports `Running`. A latency alarm at 3,600 seconds is a build requirement,
not an operational nicety
(`knowledge/aws-data-stack/streaming-vs-batch-ingestion.md`).

## 5. Governance

- **PCI DSS scope.** PAN is tokenised at ingest; raw PAN never lands in S3.
  This keeps the lakehouse outside the cardholder data environment for the
  annual QSA assessment, at the cost of losing joins that need the real PAN
  — those stay in the PCI-scoped environment. The alternative, masking in
  Lake Formation, would have kept flexibility and pulled the whole account
  into scope (`knowledge/banking-domains/cards-and-transaction-data.md`,
  `knowledge/aws-data-stack/tokenisation-and-encryption.md`).
- **Deterministic, format-preserving tokens.** Deterministic so that
  per-card aggregation and the `card_token` sort order work without a
  detokenisation call on every join; the accepted cost is that equality
  leaks and frequency analysis on the token is possible, which is why
  `card_token` is classified `PCI-Token` and not treated as anonymised.
  Random tokens would leak nothing and make every cross-table join a call
  to the token service (`knowledge/aws-data-stack/tokenisation-and-encryption.md`).
- **Detokenisation is not permitted in this account.** It happens only in
  the dedicated in-scope account. This keeps the CDE small and adds a hop
  for investigators (same source).
- **Free-text PAN leak.** `merchant_descriptor_raw` is scanned at ingest
  against a 13–19 digit pattern; the check blocks publication. This is the
  documented failure mode where PAN reaches S3 and Athena query logs via a
  narration field and is found months later by a PCI scan.
- **Classification.** Per-column labels drive LF-Tags. New columns default
  to `PII-Review` — fails closed, so an unlabelled column cannot ship
  readable (`knowledge/aws-data-stack/glue-lake-formation-governance.md`).
- **Residency.** `me-central-1`, entity of origin `reference-bank-onshore`.
  DIFC-booked portfolios are an explicit non-goal for v1; pooling them is a
  transfer question before it is an engineering one
  (`knowledge/uae-regulatory/difc-adgm-data-residency.md`).
- **Crawler-free.** Table defined in IaC, not inferred by a Glue crawler,
  so schema changes arrive as a reviewable diff rather than silent drift.

### Open regulatory items — carried forward, not resolved

Verified against `knowledge/uae-regulatory/SOURCES.md` (3 September 2026).
Two items resolved; three remain open and must be closed with Legal and the
DPO before sign-off.

**Resolved — now design constraints, not questions:**

- **Retention floor is five years**, from transaction completion or the end
  of the business relationship, whichever is later, under Federal
  Decree-Law No. 10 of 2025 and Cabinet Resolution No. 134 of 2025. The
  planned 7-year lifecycle therefore exceeds the AML floor by two years
  with no AML basis, which is a data minimisation exposure under PDPL, not
  a safe default. **Action: justify years 6-7 against a named purpose or
  cut the lifecycle to five.**
- **Rights response window is one month by default**, extendable by up to
  two further months where a request is onerous. That is the budget for
  locating one customer's rows across S3, Redshift and backups. The PDPL
  executive regulations remain unissued as of March 2026, so operational
  detail may still change. **[VERIFY]**

**Still open:**

- Whether crypto-shredding is an acceptable erasure method to our external
  auditor, versus Iceberg row deletes plus compaction. **[VERIFY]**
- Whether destroying a customer's token mapping is accepted as erasure of
  the card data, given the token itself remains in the table. **[VERIFY]**
- Applicability of the DIFC regime (Law No. 5 of 2020, as amended 8 July
  2025) to any card portfolio booked through the DIFC entity. **[VERIFY]**

## 6. Cost

Steady-state estimate at 1.2M settled legs/day. **All figures are AWS
list-price estimates and must be confirmed against the pricing calculator
for `me-central-1` before sign-off — Region pricing is not us-east-1
pricing** (`knowledge/aws-data-stack/data-platform-cost-model.md`). Volume
is an assumption pending the scheme file history review.

| Line | Basis | Est. monthly |
|---|---|---|
| S3 storage | ~700 GB steady state incl. snapshot overhead | ~$18 |
| Glue ETL | ~22 DPU-hours/day | ~$300 |
| Compaction + table maintenance | Nightly | ~$60 |
| Glue Data Quality | 9 rules, daily | ~$50 |
| Athena (self-serve + analytics) | ~6 TB scanned/month, partitioned | ~$30 |
| Redshift Serverless (finance mart) | 8 RPU × ~4 hrs/day | ~$430 |
| **Total** | | **~$890** |

**Trade-off taken.** Budget alert at $1,000 rather than a hard cap. A hard
cap would force design trade-offs early and could block a legitimate
month-end close spike — precisely when finance cannot tolerate a failure;
the alert is flexible and relies on someone acting on it, so it routes to
the accountable owner, not a shared channel
(`knowledge/pm-artifacts/data-product-prd.md`).

**Trade-off taken.** Redshift Serverless on-demand for v1, not a
reservation. A 1-year commitment would take 30-50% off the largest line but
commits before the finance workload's real shape is known; revisit at the
first renewal with three months of RPU-hour data
(`knowledge/aws-data-stack/data-platform-cost-model.md`).

**Trade-off taken.** Finance reads through Redshift rather than Athena
despite Athena being cheaper per scan. Finance runs a full-period,
value-level reconciliation repeatedly through close week, which is the
pattern where per-TB-scanned billing turns expensive and sustained RPU-hours
do not (same source). Cost is tracked as unit cost — spend per 1,000
consumer queries — so the figure survives volume growth.

**Trade-off taken.** Nightly compaction, not hourly. Nightly is materially
cheaper and leaves intraday queries scanning more files — acceptable
because no consumer of this product queries intraday
(`knowledge/aws-data-stack/s3-lakehouse-table-formats.md`).

## 7. Non-goals

Explicitly out of scope for v1.0.0. Each is a "while you're building it,
can you add…" request with an answer already recorded.

- **Authorisation-level data.** Lives in
  `cards.card_authorisations.stream`. Never union the two — authorisation
  and settlement spend differ by roughly 2–5% on any day through reversals,
  partial clearings and tips, and unioning them double-counts every
  purchase (`knowledge/banking-domains/cards-and-transaction-data.md`).
- **Real-time or intraday spend.** Settlement is inherently T+2/T+3.
  Anything needing sub-day latency is an authorisation-stream product.
- **DIFC-booked card portfolios.** Residency and regime scoping required
  first.
- **Guaranteed merchant name canonicalisation.**
  `merchant_name_canonical` is nullable and best-effort; a coverage
  commitment needs an ongoing rules-and-review function that is not funded.
- **Fraud feature store.** This table is a source for one, not the thing
  itself.
- **Chargeback and dispute linkage.** Joining disputes to the originating
  transaction is v1.1 scope; without it, dispute rates cannot be expressed
  as a proportion of spend.

## 8. Success metrics

**Top metric.** Cards interchange reconciliation time at month-end close:
**4 working days → 1**. A business outcome, not an SLA attainment figure —
it earns executive attention and is partly confounded by finance process
changes we do not control; the platform-outcome alternative would have been
fully attributable and read as self-serving
(`knowledge/pm-artifacts/metric-tree.md`).

Supporting branches:

- **Adoption** — consumers running ≥1 query in a week, measured from
  Redshift and Athena query logs rather than a survey. Repeat consumers,
  not distinct users; distinct-user counts rise with every onboarding email
  and fall silently as people stop coming back (documented failure mode).
- **Reliability** — SLA attainment against the D+3 06:00 commitment.
- **Guardrail** — run cost must not exceed $1,000/month, tracked as unit
  cost per 1,000 queries, while the above improve. All platform resources
  carry a per-data-product cost allocation tag at creation; untagged spend
  is the documented route to a bill no domain will accept
  (`knowledge/aws-data-stack/data-platform-cost-model.md`).

## 9. Acceptance criteria

v1.0.0 is certified when all of the following hold:

1. `data-contract.yaml` merged, validated in CI, registered to Glue Schema
   Registry.
2. All 9 quality rules executing daily; `uniqueness_transaction_id` and
   `pan_leak_scan` blocking publication on failure.
3. Scheme total reconciliation passing for 10 consecutive business days,
   value-level, not row-count-only.
4. Freshness instrumented against settlement file cycle date, with the
   alarm firing correctly in a deliberate late-file test.
5. `CDCLatencySource` alarm proven to fire at the 3,600-second threshold.
6. Lake Formation grants verified for all four consumers, including a
   negative test proving the self-serve role cannot read `card_token`.
7. `IAMAllowedPrincipals` confirmed removed from the database — LF grants
   are not evidence on their own
   (`knowledge/aws-data-stack/glue-lake-formation-governance.md`).
8. Quarantine bucket confirmed inside the tokenisation flow — a negative
   test proving no raw PAN reaches the rejected-records path.
9. Every resource carries a per-data-product cost allocation tag; a Cost
   Explorer view returns the product's spend with zero unallocated.
10. Lineage visible end-to-end in the catalog with no gap at the
    transformation step.
11. All **[VERIFY]** items in §5 closed by Legal and the DPO in writing.
12. Named sign-off from Finance — Cards and Financial Crime.
