# Migration Runbook

## What it is (3 sentences max)
A time-sequenced document for moving a data workload — a Teradata or
on-premise Oracle warehouse to Redshift and S3, or a mart from one platform
to another — with pre-checks, cutover steps, validation queries, rollback
triggers and named owners per step. Each step carries an expected duration,
a verification command and a go/no-go decision. The runbook is written to
be executed at 02:00 by someone who did not build the system.

## Why a data PM cares
- The parallel-run period is where migration budgets die; deciding its
  length is a product decision with a monthly cost attached.
- Rollback triggers must be defined before the window, because at 03:00 the
  team will always argue for pressing on.
- Regulatory reporting cannot miss a cycle, so the cutover date is
  constrained by close calendars and CBUAE submission dates, not by
  engineering readiness.

## Decisions a PM actually makes
- Big-bang cutover vs domain-by-domain: big-bang ends dual-running cost in
  one weekend and puts every consumer at risk at once, phased migration
  runs two platforms for two quarters and keeps blast radius small.
- Parallel-run length: 2 cycles proves the mechanics and misses quarter-end
  behaviour, 3 months covers a full close cycle and doubles the licence
  overlap cost.
- Row-count-and-checksum validation vs full value-level reconciliation:
  checksums are fast and miss compensating errors, full reconciliation on
  key financial columns is slow and is what finance will actually accept.
- Freeze upstream changes during migration vs allow them: a freeze
  simplifies validation and blocks the business for weeks, allowing changes
  means every delta must be applied twice.
- Decommission immediately after cutover vs keep the legacy platform
  read-only for 90 days: immediate decommission captures the savings the
  business case promised, a read-only period is the only real rollback once
  writes have moved.

## Failure modes
- Validation passes on aggregates while a join fan-out doubles a low-volume
  segment — found weeks later when a product-level report is queried.
- Rollback untested — the documented step assumes a snapshot that was never
  taken, discovered during the window.
- Downstream consumers not inventoried — an unmonitored Excel-over-ODBC
  extract used by treasury breaks on cutover and nobody knew it existed.

## Vocabulary
- **Cutover window** — the agreed period when writes move.
- **Parallel run** — both platforms producing output for comparison.
- **Rollback trigger** — the pre-agreed condition that aborts the cutover.
- **Reconciliation query** — the SQL comparing old and new outputs.
- **Freeze** — a moratorium on upstream schema or logic changes.
- **Go/no-go** — a named decision point with a named decider.
- **Decommission** — switching off and archiving the legacy platform.
