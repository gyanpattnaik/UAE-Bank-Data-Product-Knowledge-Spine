# Lineage and Catalog

## What it is (3 sentences max)
Lineage is the traced path from a source column in the core banking system
to the field printed in a regulatory return, at table and ideally column
granularity. A catalog is the searchable inventory of data products with
owners, classifications, definitions and lineage attached — DataZone,
Collibra, Alation, or OpenMetadata over the Glue Catalog. Lineage is
evidence; the catalog is the interface people actually use.

## Why a data PM cares
- Impact analysis before a core banking upgrade is the difference between a
  1-day change window and an unplanned outage of the AML feed.
- An internal audit finding on "data provenance for regulatory reporting"
  is closed with column lineage screenshots, not architecture diagrams.
- Duplicate metric definitions are found by search, not by memory — two
  `active_customer` definitions across retail and cards is normal until
  someone can see both.

## Decisions a PM actually makes
- Automated parse-based lineage vs manually curated: parsing Spark and
  Redshift SQL covers 80% of pipelines with no team effort but misses
  vendor black boxes, manual curation is complete for the 12 tables you
  care about and stale within two quarters.
- Column-level vs table-level lineage: column-level answers auditor
  questions directly and costs several times more to build and maintain,
  table-level is enough for impact analysis.
- Buy (Collibra/Alation) vs assemble (DataZone plus OpenLineage):
  buying gives a governance workflow the CDO's team already knows and a
  six-figure annual licence, assembling costs 2-3 engineers for two
  quarters and fits the AWS accounts exactly.
- Catalog coverage target: cataloguing everything makes search noisy and
  never finishes, cataloguing only certified data products leaves the long
  tail invisible and pushes analysts back to Slack.
- Steward model: a named steward per domain keeps definitions current but
  is a real 20% role, crowd-sourced editing scales and drifts.

## Failure modes
- Lineage stops at a stored procedure or a vendor ETL tool — the graph
  shows a gap exactly where the regulatory transformation happens.
- Catalog entries with owner fields pointing at people who left — a
  freshness incident sits unassigned for 6 hours.
- Definitions in the catalog diverge from the SQL in production; the
  catalog says `active = transacted in 90 days`, the mart uses 30, and
  nobody notices until two decks disagree.

## Vocabulary
- **OpenLineage** — open spec for emitting lineage events from jobs.
- **Column-level lineage** — field-to-field derivation mapping.
- **Steward** — accountable owner of a domain's definitions.
- **Business glossary** — canonical term definitions linked to tables.
- **Certified data product** — an entry meeting the platform's own bar.
- **Impact analysis** — listing downstream consumers of a proposed change.
- **DataZone** — AWS catalog and data-sharing service over Glue.
