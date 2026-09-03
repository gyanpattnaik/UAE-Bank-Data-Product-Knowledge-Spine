# Core Banking and Payments Data

## What it is (3 sentences max)
Core banking data is the customer, account, balance and general-ledger
posting record held in a vendor platform — Finacle, T24/Temenos, Flexcube
or Oracle Banking — with an ISO 20022 payments layer around it. In the UAE
this connects to CBUAE rails: UAEFTS/UAEIPP for domestic transfers, the
Aani instant payments scheme, direct debit (UAEDDS), and SWIFT for
cross-border. The lakehouse view of this domain is almost always
CDC-replicated tables plus a message archive, never direct core access.

## Why a data PM cares
- Balance is a point-in-time concept with at least three variants —
  ledger, available, and cleared — and picking the wrong one silently
  breaks liquidity and overdraft reporting.
- Payment messages arrive as pacs.008/pacs.002 pairs; without joining the
  status response you cannot distinguish attempted from settled volume.
- The core's business date is not the calendar date — the Friday-Saturday
  to Saturday-Sunday weekend change and end-of-day cutovers shift when a
  transaction lands.

## Decisions a PM actually makes
- Replicate the core schema as-is vs model a canonical layer: as-is is fast
  and couples every consumer to vendor column names that change on upgrade,
  canonical costs a quarter of modelling and survives the next core
  migration.
- Event-level vs daily-snapshot balances: event-level supports intraday
  liquidity and multiplies row counts, daily snapshots are cheap and cannot
  answer "what was the balance at 14:00".
- Business date vs system timestamp as the partition key: business date
  matches finance and the GL, system timestamp matches operational reality
  and produces cross-day partitions at cutover.
- ISO 20022 full message retention vs extracted fields: retaining raw XML
  is defensible for disputes and expensive to query, extracted fields are
  fast and lose remittance information the investigations team needs.
- Reconciliation frequency: daily GL-to-lake reconciliation catches drift
  within one cycle at real ops cost, monthly is cheaper and lets an error
  reach a regulatory return.

## Failure modes
- Reversals modelled as updates rather than new postings — the transaction
  count is right, the audit trail is gone, and net position cannot be
  rebuilt for a prior date.
- Multi-currency amounts stored without the rate used — AED-equivalent
  totals cannot be reproduced and finance restates the report.
- CDC deletes not propagated — closed accounts stay active in the mart and
  the active-customer count drifts up a few percent per quarter.

## Vocabulary
- **pacs.008 / pacs.002** — ISO 20022 credit transfer and status messages.
- **Aani** — UAE instant payments platform.
- **UAEFTS** — CBUAE domestic funds transfer system.
- **Business date** — the core's accounting date for a posting.
- **Available vs ledger balance** — usable funds vs posted funds.
- **GL posting** — the double-entry record behind a transaction.
- **Cutover** — the core's end-of-day batch window.
