# CBUAE Open Finance Regulation

## What it is (3 sentences max)
The CBUAE Open Finance Regulation was issued as Circular 7 of 2023 and
updated by Circular 3 of 2025, effective 10 July 2025; it requires licensed
financial institutions to expose customer data and payment initiation
through a central API hub rather than bilateral integrations, with customer
consent as the access control. Nebras Open Finance LLC, a CBUAE subsidiary,
operates that API hub and publishes the Open Finance Standards; Al Etihad
Payments is a separate CBUAE subsidiary running national rails such as Aani
and Jaywan, and is not the Open Finance operator. Onboarding is phased,
with the first phase covering banks including branches of foreign banks,
and insurance companies.

## Why a data PM cares
- Data-sharing obligations turn internal account and transaction models
  into external API contracts, so schema stability stops being internal.
- Consent state becomes a first-class data entity with its own lifecycle,
  audit trail and revocation semantics — it must be queryable historically.
- Aggregators surface inconsistencies (balance definitions, pending
  transactions) that internal reporting tolerated.

## Decisions a PM actually makes
- Serve reads from the core vs from a lake-backed read model: core-direct
  is authoritative and adds load and availability risk to the core, a read
  model gives isolation and introduces a lag you must publish.
- Consent held only in the hub's consent layer vs also modelled in the
  lake: hub-only is simpler, lake-modelled lets you evidence consent state
  at any past timestamp when a customer disputes access.
- Pending vs posted transactions in the API response: exposing pending
  matches the mobile app's view and produces amounts that later change,
  posted-only is stable and looks stale to the customer.
- Build for the mandated scope only vs a superset API: mandated-only is
  cheapest to certify, a superset positions the bank for BaaS and doubles
  the surface to secure.
- Consent expiry and re-consent cadence: shorter cycles are the better PDPL
  posture and increase friction and aggregator churn; any binding maximum
  is set by the regulation and the Standards, not by product. [VERIFY]

## Failure modes
- Consent revocation not propagated downstream — the aggregator is cut off
  and derived features built on that data stay in a training set.
- Read model drift — the API reports a balance differing from the mobile
  app for the same account, and complaints arrive via the aggregator.
- Rate limiting tuned for internal traffic — aggregator polling saturates
  the endpoint at month-start and error rates breach the availability
  requirement. [VERIFY]

## Vocabulary
- **Nebras Open Finance LLC** — CBUAE subsidiary operating the API hub.
- **Al Tareq** — consent, authentication and payment-initiation layer.
- **Al Etihad Payments** — separate CBUAE subsidiary; Aani and Jaywan.
- **TPP** — third-party provider consuming Open Finance APIs.
- **Consent artefact** — the record of scope, duration and revocation.
- **Data holder** — the institution obliged to expose customer data.
- **Open Finance Standards** — the hub's published API specifications.
