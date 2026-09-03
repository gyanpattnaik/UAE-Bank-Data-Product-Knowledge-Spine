# UAE PDPL (Federal Decree-Law 45 of 2021)

## What it is (3 sentences max)
Federal Decree-Law No. 45 of 2021 is the UAE's onshore personal data
protection law, setting lawful bases, data subject rights, controller and
processor obligations and breach notification; it does not displace the
DIFC and ADGM regimes. Its executive regulations were due within six months
of issuance and, as of March 2026, still have not been issued, so several
obligations cannot be operationalised from the text alone. The UAE Data
Office exists under Federal Decree-Law No. 44 of 2021 but is not fully
operational, with the TDRA acting as contact point.

## Why a data PM cares
- Enforcement is limited and the standard is your own risk assessment, so
  decisions made now get judged against regulations not yet written.
- Rights requests run to a one-month default response, extendable by two
  further months where onerous, so you must locate one customer's data
  across S3, Redshift and backups inside that.
- Cross-border transfer rules constrain where analytics and SaaS
  observability tools run, and the adequacy list that would clarify this
  awaits the Data Office. [VERIFY]

## Decisions a PM actually makes
- Consent vs another lawful basis for analytics: consent is clean to
  explain and revocable, so models must be retrainable without that
  customer; other bases are more durable and need a documented assessment
  whose form the regulations have not fixed.
- Pseudonymise at ingest vs retain identifiers in the raw zone: early
  pseudonymisation shrinks the compliance surface and blocks late-arriving
  joins, retaining them keeps flexibility and puts the raw zone in scope
  for every rights request.
- Hard delete vs crypto-shredding: Iceberg row deletes plus compaction
  genuinely remove data at real compute cost, per-customer key destruction
  is fast and requires your auditor to accept it.
- Retention by regulatory maximum vs purpose minimum: keeping everything
  for the five-year AML floor is simple and sits against data minimisation.
- Build to GDPR parity now vs to the bare UAE text: parity is defensible
  when the regulations land and over-engineers today, the bare text is
  cheaper and risks rework.

## Failure modes
- Erasure applied to the serving layer only — the customer is gone from
  Redshift, remains in the raw zone and in months of Athena results, and a
  rights audit finds them.
- No purpose registry — a team reuses a dataset collected for servicing
  and nobody can state the basis when challenged.
- PII in logs — masking implemented in tables but not in application logs
  retained for 90 days.

## Vocabulary
- **Controller / processor** — who sets purposes vs who acts on it.
- **UAE Data Office** — federal authority under Decree-Law 44 of 2021.
- **TDRA** — regulator acting as contact point.
- **Lawful basis** — the recorded justification for a processing purpose.
- **Crypto-shredding** — erasure by destroying the decryption key.
- **DPIA** — assessment required before high-risk processing.
- **Cross-border transfer** — moving personal data outside the UAE.
