# Agent instructions

This repo is a knowledge spine plus worked examples. You generate PM
artifacts for data-product roles at UAE banks and fintechs, using
`/knowledge` as your context source. `/examples` holds artifacts already
generated from the spine; read one before generating the same artifact type.

## Routing — read only what the artifact needs

| Artifact requested | Read these folders |
|---|---|
| Data contract | `pm-artifacts/data-contract-template.md`, `data-product-practice/data-contracts.md`, the relevant `banking-domains` file |
| Data product PRD | `pm-artifacts/data-product-prd.md`, `pm-artifacts/metric-tree.md` (success metrics), `aws-data-stack/data-platform-cost-model.md` (cost section), `data-product-practice/*`, the relevant `banking-domains` file |
| Migration runbook | `pm-artifacts/migration-runbook.md`, `aws-data-stack/*` |
| Metric tree | `pm-artifacts/metric-tree.md`, the relevant `banking-domains` file |
| Anything touching personal data, residency, consent, or reporting obligations | add `uae-regulatory/*` |
| Anything touching PAN, Emirates ID, IBAN, PCI scope, or encryption keys | add `aws-data-stack/tokenisation-and-encryption.md` |
| Any cost, pricing, or run-rate question | `aws-data-stack/data-platform-cost-model.md` |
| Architecture or platform choice | `aws-data-stack/*` |

Do not read all 22 knowledge files. Reading the whole spine for a
single-artifact request is a failure, not thoroughness.

## Rules

1. **Cite the source file for every trade-off you assert.** Inline, as
   `(knowledge/banking-domains/cards-and-transaction-data.md)`. A trade-off
   with no citation is one you invented and must be marked as such.
2. **Never state a `[VERIFY]`-marked claim as fact.** Every `[VERIFY]` line
   in `uae-regulatory/` is unconfirmed. Carry it into the artifact as an
   open item with the marker intact, or leave the claim out. Do not
   silently resolve one.
3. **Never invent a legal citation.** No article numbers, decree numbers,
   circular references, or effective dates beyond what `uae-regulatory/`
   already contains. `uae-regulatory/SOURCES.md` records where each claim
   came from and when it was checked; cite it when a reader may need to
   re-verify, and never add a regulatory claim without adding its source
   there.
4. **Every number is traceable or labelled.** A threshold, price, latency
   or volume figure either comes from a knowledge file, from the user, or
   is written as an explicit assumption the reader can challenge. AWS
   prices are list-price estimates and must say so.
5. **Resolve trade-offs; do not present them.** The knowledge files name
   both sides. An artifact picks one side and states why. Handing the
   reader an unresolved options list is not a deliverable.
6. **Both sides get named.** When you state a decision, name what you gave
   up. "Settlement-based, accepting T+3 latency" — not "settlement-based."
7. **Design against the documented failure modes.** Each knowledge file
   lists three with observable symptoms. Show in the artifact how the
   design prevents them.

## Adding to `/knowledge`

Match the existing shape exactly: five sections in fixed order (`What it
is` at 3 sentences, `Why a data PM cares` at 3 bullets, `Decisions a PM
actually makes` at 4-6 bullets with both sides of each trade-off named,
`Failure modes` at 3 bullets each with an observable symptom, `Vocabulary`
at 5-8 one-line terms). `SOURCES.md` is exempt from the shape and the
cap. Hard cap 300-500 words per knowledge file. No intros, no
motivational framing, no advice that would be true for any company in any
country. Do not define terms a PM already knows.
