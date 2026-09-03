# Tokenisation, Encryption and PCI Scope

## What it is (3 sentences max)
Tokenisation replaces a sensitive value — PAN, Emirates ID, IBAN — with a
surrogate at ingest so the raw value never lands in S3, while encryption
(SSE-KMS, envelope encryption) protects whatever does land. Format-
preserving tokens keep the value's shape so column types and BI tools keep
working. Tokenising at ingest versus masking at read is what decides
whether the account sits inside the PCI DSS cardholder data environment.

## Why a data PM cares
- Scope drives audit cost: a lake inside the CDE puts every bucket, role and
  account touching it into the annual QSA assessment.
- KMS key policy, not IAM alone, is the real boundary — a role with S3 read
  and no decrypt grant fails in a way that does not read as access denied.
- The token service becomes a hard ingest dependency with its own
  availability SLA — a capacity and on-call commitment.

## Decisions a PM actually makes
- Deterministic vs random tokens: deterministic tokens let you join and
  aggregate per card without ever detokenising and leak equality, so
  frequency analysis on the token is possible; random tokens leak nothing
  and force a detokenisation call for any cross-table join.
- Format-preserving vs opaque tokens: FPE keeps column types and downstream
  tooling working and narrows the cryptographic choices, opaque tokens are
  simpler and break every length and format assumption downstream.
- Vault-based vs vaultless FPE: a vault gives revocability and a per-request
  audit trail and is a stateful HA dependency in ingest, vaultless is
  stateless and trivially scalable and makes rotation harder.
- Customer-managed keys per domain vs one account key: per-domain CMKs make
  key policy the enforcement point and multiply cross-account grant admin,
  a single key is simple and lets any decrypt grant read the whole lake.
- Where detokenisation is permitted: a dedicated in-scope account keeps the
  CDE small and adds a hop for investigators, an authorised analytics role
  is convenient and drags analytics into PCI scope.

## Failure modes
- Re-tokenisation across a service redeploy — the same card receives two
  tokens either side of the boundary and per-card aggregates split; the
  symptom is active card count rising with no acquisition activity.
- KMS request throttling — a wide Athena scan over an SSE-KMS bucket trips
  the KMS rate limit and queries fail only at high concurrency, which reads
  as an intermittent Athena fault.
- Raw values surviving in a secondary path — the main table is tokenised
  while the quarantine bucket was never wired into the tokenisation flow;
  found by a scan months later.

## Vocabulary
- **CDE** — cardholder data environment; the PCI DSS assessment boundary.
- **FPE** — format-preserving encryption.
- **Deterministic token** — same input always yields the same token.
- **Vaultless tokenisation** — token derived cryptographically, not stored.
- **Envelope encryption** — data key encrypted by a KMS master key.
- **CMK / key policy** — customer-managed key and its resource policy.
