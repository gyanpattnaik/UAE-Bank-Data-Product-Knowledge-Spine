# Domain Ownership and Mesh Trade-offs

## What it is (3 sentences max)
Data mesh assigns ownership of data products to the business domains that
produce them — payments, cards, lending, deposits — with a central platform
team providing self-serve infrastructure and federated governance. In a
bank this collides with the fact that the core banking system is one vendor
platform owned by one IT group, so the mesh boundary usually falls at the
consumption layer rather than the source. Most UAE bank implementations end
up hybrid: centralised ingestion, domain-owned marts.

## Why a data PM cares
- Ownership determines who answers the pager at 02:00; if that is unfunded
  in the domain's headcount, the mesh is a diagram, not an operating model.
- Federated governance still needs one set of enforced standards, and you
  are usually the person writing them.
- Domain-owned products let cards ship a new feature without queueing
  behind the finance close — that queue is often the real business case.

## Decisions a PM actually makes
- Centralised platform team vs domain-embedded engineers: central delivers
  consistency and becomes the bottleneck at roughly 8-10 consuming teams,
  embedded delivers speed and produces five ways to write a merge.
- Mesh boundary at source vs at mart: source-level ownership is the true
  mesh and impossible while a vendor core is a single black box, mart-level
  is achievable now and leaves ingestion centrally owned.
- Mandated standards vs paved road: mandating Iceberg, contracts and
  tagging gets compliance and resentment, making the paved road markedly
  easier gets adoption and tolerates two teams going their own way.
- Chargeback vs showback of platform cost: chargeback makes domains
  optimise queries within a quarter and triggers arguments about shared
  cost allocation, showback is politically easy and changes no behaviour.
- Domain granularity: 6 broad domains keep governance manageable, 20 fine
  domains match team boundaries and multiply the interface count.

## Failure modes
- Responsibility transferred without capability — the cards domain owns its
  data product and its two analysts have never written a Glue job; release
  cadence drops to one change a quarter.
- Federated governance with no enforcement — three domains define
  `customer` differently and the group-level customer count is unstateable.
- Platform team measured on tickets closed rather than self-serve adoption;
  ticket volume rises every quarter and is reported as success.

## Vocabulary
- **Data product** — a governed, discoverable, owned dataset with an SLA.
- **Federated governance** — shared standards, local implementation.
- **Paved road** — the supported default path teams are nudged onto.
- **Showback / chargeback** — reporting vs billing platform cost to teams.
- **Interface** — the published contract surface between two domains.
- **Self-serve platform** — infrastructure usable without a platform ticket.
- **Domain** — a business capability boundary that owns its data.
