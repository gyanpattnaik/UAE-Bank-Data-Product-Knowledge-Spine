# Cards and Transaction Data

## What it is (3 sentences max)
Cards data spans the authorisation stream from the switch (ISO 8583 or its
ISO 20022 successor), the clearing and settlement files from Visa (VSS/BASE
II) and Mastercard (IPM/Clearing Optimizer), and the posted transaction in
the core. The same purchase appears three times with different amounts and
timestamps: authorised, cleared, and posted. Any cards data product must
declare which of the three it counts.

## Why a data PM cares
- Authorisation-based spend and settlement-based spend differ by roughly
  2-5% on any given day because of reversals, partial clearings and tips —
  business teams will quote whichever is higher.
- MCC drives interchange, rewards eligibility and merchant categorisation
  for spend analytics; MCC quality from acquirers is inconsistent.
- PAN is the single most regulated field on the platform, and PCI DSS scope
  follows it into every table it touches.

## Decisions a PM actually makes
- Tokenise PAN at ingest vs mask in Lake Formation: ingest tokenisation
  keeps the lake out of PCI DSS cardholder-data scope and breaks joins that
  need the real PAN, LF masking keeps flexibility and keeps the whole
  account in scope for the annual assessment.
- Authorisation-based vs settlement-based spend as the headline metric:
  authorisation is near real-time and overstates, settlement is accurate
  and lands 2-3 days late.
- Store the full authorisation stream vs declines only in aggregate: full
  stream supports fraud modelling and roughly triples the table's row
  count, aggregates are cheap and useless for feature engineering.
- Merchant identity resolution: cleaning raw merchant descriptors into a
  canonical merchant improves customer-facing statements and PFM features
  but needs an ongoing rules and review process, raw descriptors are
  faithful and unreadable.
- Chargeback linkage: joining disputes back to the original transaction at
  ingest is complex and enables true loss reporting, keeping them separate
  is simple and leaves dispute rates unlinkable to spend.

## Failure modes
- Double counting the same purchase across auth and clearing — daily spend
  in the dashboard exceeds the settlement file total by a consistent margin
  and nobody can reconcile to finance.
- PAN leaking into a free-text field (merchant descriptor, narration) — the
  PCI scan flags it after the data has been in S3 and Athena logs for
  months.
- Timezone drift between the switch (often UTC) and the core (Gulf
  Standard Time, UTC+4) — end-of-day spend is misattributed by 4 hours and
  Ramadan late-night volume lands on the wrong day.

## Vocabulary
- **MCC** — merchant category code driving interchange and analytics.
- **ISO 8583** — legacy card authorisation message format.
- **BASE II / IPM** — Visa and Mastercard clearing file formats.
- **Interchange** — the fee paid by acquirer to issuer per transaction.
- **Partial clearing** — settlement for less than the authorised amount.
- **Chargeback** — disputed transaction reversed through the scheme.
- **PAN** — primary account number; the card number itself.
