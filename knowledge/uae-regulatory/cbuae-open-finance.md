# CBUAE Open Finance Regulation

## What it is (3 sentences max)
The CBUAE Open Finance Regulation in force is Circular C 03/2025, issued
10 July 2025, which repealed and replaced Circular C 7/2023; it requires
licensed financial institutions to expose customer data and payment
initiation through a central API hub rather than bilateral integrations,
with express user consent as the access control. Nebras Open Finance LLC, a CBUAE subsidiary,
operates the hub and publishes the Open Finance Standards; Al Etihad
Payments is a separate CBUAE subsidiary running rails such as Aani and
Jaywan. Onboarding is phased, the first phase covering banks including
foreign branches, and insurance companies.

## Why a data PM cares
- Data-sharing obligations turn internal account and transaction models into
  external API contracts, so schema stability stops being internal.
- Consent becomes a first-class data entity with its own lifecycle, audit
  trail and revocation semantics — it must be queryable historically.
- Aggregators surface inconsistencies (balance definitions, pending
  transactions) that internal reporting tolerated.

## Decisions a PM actually makes
- Serve reads from the core vs a lake-backed read model: core-direct is
  authoritative and adds load and availability risk, a read model gives
  isolation and introduces a lag you must publish.
- Consent held only in the hub's consent layer vs also modelled in the
  lake: hub-only is simpler, lake-modelled lets you evidence consent state
  at any past timestamp when a customer disputes access.
- Pending vs posted transactions in the response: exposing pending matches
  the mobile app and produces amounts that later change, posted-only is
  stable and looks stale to the customer.
- Mandated scope only vs a superset API: mandated-only is cheapest to
  certify, a superset positions the bank for BaaS and doubles the attack
  surface.
- Consent duration for recurring transactions: the Regulation caps validity
  at twelve months and requires withdrawal to be as easy as granting, so
  the only product choice is how far below twelve months to sit — shorter
  is the better PDPL posture and raises churn.

## Failure modes
- Revocation not propagated downstream — the aggregator is cut off and
  derived features built on that data stay in a training set.
- Read model drift — the API reports a balance differing from the mobile
  app for the same account, and complaints arrive via the aggregator.
- Rate limiting tuned for internal traffic — aggregator polling saturates
  the endpoint at month-start. The Regulation sets no numeric uptime or
  rate-limit figure, only an IT risk framework covering availability, so
  the binding number comes from the Standards and hub KPIs.

## Vocabulary
- **Nebras Open Finance LLC** — CBUAE subsidiary operating the API hub.
- **Al Tareq** — consent, authentication and payment-initiation layer.
- **Al Etihad Payments** — separate CBUAE subsidiary; Aani, Jaywan.
- **TPP** — third-party provider consuming the APIs.
- **Consent artefact** — record of scope, duration and revocation.
- **Data holder** — institution obliged to expose customer data.
- **Open Finance Standards** — the hub's API specifications.
