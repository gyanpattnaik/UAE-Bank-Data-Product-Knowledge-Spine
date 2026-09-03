# Credit Risk and IFRS 9 Reporting

## What it is (3 sentences max)
Credit risk data supports origination scoring, behavioural scoring,
collections, and the IFRS 9 expected credit loss (ECL) calculation that
drives the provision in the financial statements. IFRS 9 classifies
exposures into three stages — performing, significant increase in credit
risk (SICR), and credit-impaired — and requires PD, LGD and EAD inputs with
forward-looking macroeconomic scenarios. In the UAE, retail exposures also
carry Al Etihad Credit Bureau (AECB) data as a scoring input.

## Why a data PM cares
- ECL is a P&L number the CFO signs; the data lineage behind it gets
  external audit attention every year, so reproducibility is the
  requirement, not a nice-to-have.
- Stage 2 is triggered by SICR rules that depend on origination-date risk
  ratings — if you never stored the rating at origination, the whole
  calculation is unbuildable.
- Days-past-due calculation depends on the cure and re-ageing policy, and
  small definitional differences move the provision by material amounts.

## Decisions a PM actually makes
- Store risk ratings as slowly changing dimension vs current state only:
  SCD Type 2 gives auditable SICR comparison and multiplies the dimension
  size, current-state is simple and makes stage transition unprovable.
- Monthly vs daily ECL data refresh: monthly matches the reporting cycle
  and is cheaper, daily supports risk appetite monitoring and forces the
  full model pipeline into a nightly window.
- Run ECL models in the lake (EMR/SageMaker) vs a vendor engine: in-lake
  gives one lineage story and requires model validation of your own code,
  vendor engines carry accepted methodology and a data export boundary the
  auditor will question.
- Facility-level vs account-level granularity: facility-level matches the
  credit system and complicates retail reporting, account-level suits
  retail portfolios and needs aggregation rules for corporate exposures.
- Macro scenario storage: retaining all scenarios and weights supports
  sensitivity disclosure and adds several dimensions to every output table,
  storing only the weighted result is compact and fails the disclosure ask.

## Failure modes
- Restatement without version pinning — rerunning last quarter's ECL with
  today's model produces a different number and the audit trail is broken.
- DPD reset on partial payment inconsistent between collections and risk —
  Stage 2 population differs between the two systems by a visible
  percentage each month-end.
- Missing AECB refresh for a segment — behavioural scores go stale, and the
  Stage 1 to Stage 2 migration rate flatlines unrealistically.

## Vocabulary
- **ECL** — expected credit loss provision under IFRS 9.
- **PD / LGD / EAD** — probability of default, loss given default,
  exposure at default.
- **SICR** — significant increase in credit risk; the Stage 2 trigger.
- **DPD** — days past due.
- **AECB** — Al Etihad Credit Bureau, the UAE credit bureau.
- **SCD Type 2** — dimension modelling that keeps historical versions.
- **Cure** — a delinquent account returning to performing status.
