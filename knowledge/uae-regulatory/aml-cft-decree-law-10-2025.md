# AML/CFT Federal Decree-Law 10 of 2025

## What it is (3 sentences max)
Federal Decree-Law No. 10 of 2025, in force 14 October 2025, is the UAE's
AML, counter-terrorist-financing and proliferation-financing law; it
repealed and replaced Federal Decree-Law No. 20 of 2018. Its implementing
regulation is Cabinet Resolution No. 134 of 2025, in force 14 December
2025, which replaced Cabinet Decision No. 10 of 2019 and carries the
operational detail. Reporting to the FIU is made through goAML.

## Why a data PM cares
- The five-year record-keeping floor — from transaction completion or the
  end of the business relationship, whichever is later — overrides data
  minimisation instincts and can extend where an investigation intervenes.
- Scope now expressly reaches virtual asset service providers, so a
  crypto-adjacent product line sits inside the monitoring perimeter from
  launch rather than being retrofitted.
- Beneficial ownership data must be updated within 15 working days of an
  identified change, which is a pipeline latency requirement, not a
  policy statement.

## Decisions a PM actually makes
- Full case data in the lake vs a walled AML account: lakehouse storage
  enables tuning analytics and widens tipping-off exposure through broad
  access, a separate account with Lake Formation isolation is safer and
  duplicates platform effort.
- Retention aligned to the five-year AML floor vs product-level purpose
  limits: one long retention is simple to defend in an inspection,
  per-purpose retention is the better PDPL posture and needs per-table
  lifecycle policy.
- Screening on every list update vs at defined intervals: event-driven
  rescreening closes exposure gaps and produces alert spikes, scheduled
  rescreening is predictable and leaves a window.
- Coverage evidence generated automatically vs assembled per examination:
  automated daily reconciliation costs build effort and answers the
  examiner in minutes, manual assembly costs weeks each time.
- Model documentation depth for ML scoring: full model risk documentation
  is expensive and is what a supervisor expects for automated decisioning,
  a lighter approach is faster and invites a finding. [VERIFY]

## Failure modes
- Controls still mapped to the repealed 2018 law — an examiner asks for the
  mapping to Decree-Law 10 of 2025 and the policy set still cites Cabinet
  Decision No. 10 of 2019.
- Product launched without being added to the monitoring feed — the
  coverage report shows a product with zero alerts since go-live.
- STR-linked identifiers appearing in a broadly shared table —
  investigators become identifiable and tipping-off risk is created
  internally.

## Vocabulary
- **DNFBP** — designated non-financial business or profession.
- **VASP** — virtual asset service provider, now expressly in scope.
- **FIU** — the UAE Financial Intelligence Unit receiving reports.
- **goAML** — the FIU's reporting platform.
- **Tipping-off** — disclosing that a report has been made; prohibited.
- **Proliferation financing** — funding of WMD proliferation; in scope.
- **Coverage reconciliation** — proving every in-scope account was
  monitored.
