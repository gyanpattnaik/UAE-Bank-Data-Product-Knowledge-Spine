# Reference Repo: Data Product PM Knowledge Spine

Machine-readable context for a coding agent generating PM artifacts for
data-product roles at UAE banks and fintechs.

`/knowledge` holds 22 files across five folders:

- `aws-data-stack` — S3 lakehouse, Glue and Lake Formation, ingestion,
  serving engines, tokenisation and PCI scope, cost model.
- `data-product-practice` — contracts, quality SLAs, lineage, ownership.
- `banking-domains` — core banking and payments, cards, AML/KYC, credit
  risk and IFRS 9.
- `uae-regulatory` — CBUAE Open Finance, PDPL, AML/CFT, DIFC/ADGM.
- `pm-artifacts` — templates to render against.

Every file uses one fixed shape: what it is, why a data PM cares, decisions
with both sides of each trade-off named, failure modes with observable
symptoms, and vocabulary. Unverified regulatory claims carry `[VERIFY]`;
provenance is in `uae-regulatory/SOURCES.md`.

`CLAUDE.md` routes an agent to the right folders and sets the generation
rules. `/examples` holds artifacts produced from the spine — start with
`examples/cards-transactions/`.
