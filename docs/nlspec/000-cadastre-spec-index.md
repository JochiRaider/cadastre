---
doc_id: CADASTRE-NLSPEC-000
title: Cadastre Spec Index and Source-of-Truth Map
doc_type: authoritative-nlspec
status: migration_active
generated_on: 2026-05-17
source_prd: PRD-Cadastre.md
source_prd_sha256: b17ac5d44618c43a57efe8ebd9b6c6e0bd8debc949b513368383c515c07f9748
---

## Authority

This document is a generated Cadastre NLSpec candidate. It is `migration_active` until the migration ledger marks its source rows complete and `docs/nlspec/120-validation-fixtures-and-acceptance.md` records passing acceptance evidence.

This document owns the contracts listed in `Exports`. Other active Cadastre NLSpecs may import those contracts by exact name and must not restate them.

## Purpose

Define the authoritative documentation set, source-of-truth ownership, spec status vocabulary, dependency policy, migration ledger pointers, and archival rules.

## Explicit Non-Scope

- Domain runtime behavior.
- Data model fields except document-governance records.
- Package, graph, identity, source, API, or validation algorithms.

## Imports

- `nlspec-spec.md`

## Exports

- `SpecStatus`
- `SpecDependency`
- `SourceOfTruthRow`
- `MigrationLedgerRow`
- `DecisionLedgerRow`

## Document Status Vocabulary

| Status | Meaning | May drive implementation |
| --- | --- | --- |
| `draft` | Authoring text that has not completed migration traceability. | No. |
| `migration_active` | Generated or migrated text under active decomposition. | No, except as migration evidence. |
| `candidate_authoritative` | Domain owner has reviewed the contract and all owned PRD rows have disposition. | Only in controlled implementation spikes. |
| `authoritative` | Acceptance criteria pass and the index marks the document active. | Yes. |
| `superseded` | Replaced by a newer document or version. | No. |
| `archived` | Retained for historical traceability only. | No. |
| `inactive_deferred` | Preserved future-domain candidate with no active MVP effect. | No. |

## Authoritative Source Rule

Implementation authority must be determined in this order:

1. Active Cadastre NLSpecs listed in this index with status `authoritative`.
2. Generated validation and acceptance reports referenced by `docs/nlspec/120-validation-fixtures-and-acceptance.md`.
3. ADRs only for rationale, never for runtime behavior.
4. Research reports only as supporting evidence, never as runtime authority.
5. `docs/archive/PRD-Cadastre.revised-draft.md` only as superseded intent after cutover.

Until this index marks a document `authoritative`, the original PRD remains the upstream intent container and the generated NLSpec is migration evidence.

## Source-of-Truth Map

| ID | File | Status | Owns |
| --- | --- | --- | --- |
| `000` | `docs/nlspec/000-cadastre-spec-index.md` | migration_active | documentation governance and source-of-truth ownership |
| `010` | `docs/nlspec/010-system-boundary-and-authority.md` | migration_active | product boundary, authority classes, and forbidden inputs |
| `020` | `docs/nlspec/020-lakehouse-feeds-and-table-state.md` | migration_active | lakehouse raw feed, table-state, read, import, and maintenance contracts |
| `030` | `docs/nlspec/030-processing-dag-lifecycle-and-versioning.md` | migration_active | stage execution, lifecycle, run locks, and version manifests |
| `040` | `docs/nlspec/040-canonical-data-observation-and-fact-model.md` | migration_active | core records, IDs, scalars, omission, and canonical serialization |
| `050` | `docs/nlspec/050-normalization-external-schema-and-mapping.md` | migration_active | parser, silver mapping, OCSF/CIM/external schema contracts |
| `060` | `docs/nlspec/060-source-authority-completeness-coverage-and-absence.md` | migration_active | source authority, completeness, staleness, coverage, absence, and watermarks |
| `070` | `docs/nlspec/070-identity-resolution-and-target-selectors.md` | migration_active | identity evidence, resolver behavior, review, split, and selector safety |
| `080` | `docs/nlspec/080-temporal-corrections-replay-and-gold-derivation.md` | migration_active | time semantics, gold derivation, correction, late arrival, replay, and side effects |
| `090` | `docs/nlspec/090-graph-projection-serving-and-backends.md` | migration_active | graph projection, backend adapters, graph apply, graph query, rebuild, and drift |
| `100` | `docs/nlspec/100-packages-supply-chain-and-activation.md` | migration_active | package release, trust, activation, rollback, quarantine, and emergency behavior |
| `110` | `docs/nlspec/110-api-ux-health-errors-and-security.md` | migration_active | public APIs, UX states, health, shared errors, redaction, audit, and security |
| `120` | `docs/nlspec/120-validation-fixtures-and-acceptance.md` | migration_active | fixtures, validation matrices, golden corpus, aggregate acceptance reports |
| `130` | `docs/nlspec/130-analysis-enrichment-and-registry-governance.md` | migration_active | analysis rules, threat-intel enrichment, lineage, and registry governance |
| `200` | `docs/deferred/200-future-reachability-analysis-domain.md` | inactive_deferred | inactive future reachability contract candidate |

## Dependency Rule

A spec may depend on another spec only through a named record, named interface, named algorithm, named error code, named lifecycle state, or named acceptance report.

A spec must not restate an imported contract except for a one-sentence reference and the exact imported name.

## Dependency Matrix

| Spec | May import from | Must export | Must not define |
| --- | --- | --- | --- |
| `000` | `nlspec-spec.md` | Source-of-truth map, status vocabulary, migration ledger refs | Product runtime behavior |
| `010` | `000` | Authority classes and boundary rules | Lakehouse table fields, graph algorithms, identity decisions |
| `020` | `010` | Feed and table-state records, read/import interfaces | Source absence interpretation or gold truth |
| `030` | `010`, `020` | DAG, lifecycle, state, manifest, lock contracts | Domain-specific output semantics |
| `040` | `010`, `030` | Core data records, IDs, canonical serialization | Source authority, resolver behavior, graph algorithms |
| `050` | `020`, `030`, `040`, `100`, `120` | Parser, mapping, schema-profile, CIM projection contracts | Gold authority or identity merge behavior |
| `060` | `010`, `020`, `030`, `040` | Authority, completeness, staleness, coverage, absence contracts | Resolver merge rules or graph apply behavior |
| `070` | `030`, `040`, `050`, `060` | Resolver, target selector, review, split handoff contracts | Direct graph mutation |
| `080` | `020`, `030`, `040`, `060`, `070` | Temporal, gold derivation, correction, replay contracts | Backend graph apply or external schema mappings |
| `090` | `010`, `030`, `040`, `060`, `070`, `080`, `100` | Graph projection, apply, backend, query, rebuild, lag contracts | Fact existence, identity, source completeness |
| `100` | `010`, `030`, `120` | Package release, trust, activation, rollback contracts | Package business behavior outside activation and execution permissions |
| `110` | `040`, `050`, `060`, `070`, `080`, `090`, `100` | API, health, error, security, redaction contracts | Domain behavior owned elsewhere |
| `120` | All active domain specs | Validation matrices, fixtures, acceptance reports | New behavioral requirements not owned by a domain spec |
| `130` | `040`, `050`, `060`, `090`, `100`, `110`, `120` | Analysis, enrichment, lineage, registry governance contracts | Fact, identity, graph, completeness, or watermark authority |
| `200` | `010`, `020`, `040`, `060`, `080`, `090`, `120` | Deferred reachability draft only | Active behavior without explicit promotion |

## Migration Control

The migration ledger at `docs/migration/PRD-to-NLSpec-ledger.md` must contain one disposition row for every PRD section that owns behavior. A PRD section is complete only when exactly one active or deferred target owns it and any residual gap is listed in `docs/migration/gap-and-ambiguity-ledger.md`.

## Archival Policy

`PRD-Cadastre.md` must be archived as `docs/archive/PRD-Cadastre.revised-draft.md` only after all migration acceptance criteria pass. The archived PRD must not be cited as implementation authority after this index marks the NLSpec set authoritative.

## Definition of Done

| ID | Criterion |
| --- | --- |
| `000-AC-001` | Every document listed in the source-of-truth map has exactly one owner, status, dependency list, import list, export list, and migration state. |
| `000-AC-002` | Every PRD top-level section, Section 10 model, Section 11 interface, Section 12 default, Section 17 error family, Section 18 edge case, Section 23 acceptance criterion, and Section 24 product decision has exactly one disposition row. |
| `000-AC-003` | No active spec defines a runtime behavior owned by another active spec. |
| `000-AC-004` | Every open decision has an owning spec and blocks authoritative status for that spec until resolved. |

## Source Traceability

| Source | Section or artifact | Location |
| --- | --- | --- |
| PRD-Cadastre.md | `Document Status` | lines 69-277 |
| PRD-Cadastre.md | `Ambiguity Audit` | lines 12854-12868 |
| PRD-Cadastre.md | `Required Product Decisions Before Final NLSpec` | lines 13635-13681 |
| Decomposition plan | Current user prompt | Domain decomposition, disposition matrix, dependency model, gap ledger, and migration acceptance criteria. |

## Open Questions

Open questions marked `TODO:` block authoritative status for the affected contract. A downstream implementation must not resolve a `TODO:` by inference.
