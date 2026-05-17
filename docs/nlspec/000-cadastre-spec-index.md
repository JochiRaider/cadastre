---
doc_id: CADASTRE-NLSPEC-000
title: Cadastre Spec Index and Source-of-Truth Map
doc_type: candidate-nlspec
status: migration_active
generated_on: 2026-05-17
source_prd: docs/archive/PRD-Cadastre.revised-draft.md
source_prd_sha256: 99437d5ec12d52752a0003577ac37f8a6c6f1221ac3ae3b7cce713b003aeae55
---

## Authority

This document is a generated Cadastre NLSpec candidate. It is `migration_active` until the migration ledger, decision ledger, gap ledger, source inventory, and `docs/nlspec/120-validation-fixtures-and-acceptance.md` acceptance reports prove cutover readiness.

This document owns documentation governance, status vocabulary, document classes, source-of-truth ownership, dependency policy, promotion gates, cutover state, source-inventory reconciliation, and archival rules. It must not define product runtime behavior.

## Purpose

Define the authoritative documentation set, source-of-truth ownership, spec status vocabulary, dependency policy, migration ledger pointers, acceptance report dependencies, cutover states, and archival rules.

## Explicit Non-Scope

- Domain runtime behavior.
- Data model fields except documentation-governance records.
- Package, graph, identity, source, API, or validation algorithms.
- Product decisions that are still open in `docs/migration/open-decision-ledger.md`.

## Imports

- `docs/reference/standards/nlspec-spec.md`

## Exports

- `SpecStatus`
- `DocumentClass`
- `SourceOfTruthRow`
- `SpecDependency`
- `SpecSetVersion`
- `PromotionGate`
- `CutoverState`
- `MigrationLedgerRow`
- `DecisionLedgerRow`
- `SourceInventoryReconciliationRule`

## Document status vocabulary

| Status | Meaning | May drive implementation |
| --- | --- | --- |
| `draft` | Authoring text that has not completed migration traceability. | No. |
| `migration_active` | Generated or migrated text under active decomposition. | No, except as migration evidence. |
| `candidate_authoritative` | Domain owner has reviewed the contract and all owned PRD rows have disposition. | Only in controlled implementation spikes named by an acceptance report. |
| `authoritative` | Acceptance criteria pass and this index marks the document active. | Yes. |
| `superseded` | Replaced by a newer document or version. | No. |
| `archived` | Retained for historical traceability only. | No. |
| `inactive_deferred` | Preserved future-domain candidate with no active MVP effect. | No. |
| `active_standard` | Reference standard for NLSpec quality only. | No product runtime authority. |

## Document class registry

| Document class | Scope | Runtime authority | Promotion rule |
| --- | --- | ---: | --- |
| `authoritative_spec` | Active NLSpecs with status `authoritative`. | Yes, for owned contracts only. | Requires passing `120` evidence and no unresolved owned blockers. |
| `candidate_spec` | NLSpecs under migration or candidate review. | No. | May become `authoritative_spec` only through `PromotionGate`. |
| `deferred_spec` | Inactive future-domain material. | No. | Requires future accepted promotion, owners, decisions, and fixtures. |
| `reference` | Glossary, inventory, research index, research reports, standards. | No. | Must not be promoted without conversion into an owning NLSpec. |
| `rationale` | ADRs. | No. | Remains rationale only. |
| `migration_support` | Migration, gap, and decision ledgers. | No. | Provides disposition and blocker evidence only. |
| `archive` | Superseded PRD and historical documents. | No after cutover. | Historical trace only after `MIG-AC-009` passes. |
| `navigation` | README and manifest navigation. | No. | Must point to this index for status. |

## Authoritative source rule

Implementation authority must be determined in this order:

1. Active Cadastre NLSpecs listed in this index with status `authoritative`.
2. Generated validation and acceptance reports referenced by `docs/nlspec/120-validation-fixtures-and-acceptance.md`.
3. ADRs only for rationale, never for runtime behavior.
4. Research reports only as supporting evidence, never as runtime authority.
5. `docs/archive/PRD-Cadastre.revised-draft.md` only as upstream intent before cutover and historical trace after cutover.

Until this index marks a document `authoritative`, the original PRD remains the upstream intent container and the generated NLSpec set is migration evidence.

## Cutover state

| Cutover state | Meaning | Required transition evidence |
| --- | --- | --- |
| `migration_active` | Documents are under migration and must not drive implementation. | Default current state. |
| `candidate_cutover` | All owner rows are dispositioned and validation rows are defined, but acceptance has not passed. | `120` creates `not_run` or `blocked` reports for remaining criteria. |
| `cutover_accepted` | `120` reports pass for all cutover criteria. | Passing `MIG-AC-001` through `MIG-AC-010`. |
| `cutover_rejected` | One or more cutover criteria fail. | `120` acceptance report with failure codes. |
| `post_cutover_authoritative` | This index promotes eligible docs to `authoritative` and PRD becomes archive trace. | Accepted cutover plus updated document registry. |

Default cutover state is `migration_active`.

## Promotion gates

| From status | To status | Required gate | Blocking conditions |
| --- | --- | --- | --- |
| `migration_active` | `candidate_authoritative` | Owner review, migration-ledger disposition, decision-ledger status, gap-ledger status, validation-row definition. | Any unresolved owner decision, blocking gap, missing validation row, or source-inventory mismatch. |
| `candidate_authoritative` | `authoritative` | Passing `AcceptanceReport` from `120` for the owner contract and aggregate cutover. | Failed, blocked, missing, or stale acceptance evidence. |
| `authoritative` | `superseded` | New accepted spec-set version. | Missing superseding version or checksum. |
| `inactive_deferred` | `migration_active` | Future accepted promotion adds owners, decisions, validation fixtures, and scope. | Any unresolved future-domain activation decision. |

## SpecSetVersion record

| Field | Type | Required | Default | Rule |
| --- | --- | ---: | --- | --- |
| `spec_set_version` | string | Yes | none | Monotonic version assigned by this index. |
| `spec_set_checksum` | sha256 string | Yes for authoritative handoff | none | Computed over included authoritative docs using canonical path order and exact bytes. |
| `included_docs` | array of path/checksum rows | Yes | empty | Must include every authoritative doc. |
| `deferred_docs` | array of path/checksum rows | Yes | empty | Must include `docs/deferred/200-future-reachability-analysis-domain.md` when still deferred. |
| `acceptance_report_id` | string | Yes for authoritative handoff | none | Must reference a passing `AcceptanceReport`. |
| `supersedes` | string or null | No | null | Must reference prior spec-set version when replacing it. |
| `effective_from` | timestamp or null | Yes for authoritative handoff | null | RFC3339 UTC timestamp assigned only after cutover acceptance. |
| `open_exclusions` | array | Yes | empty | Explicit deferred or unresolved issues implementation must not infer. |
| `validation_matrix_refs` | array | Yes | empty | Required owner validation rows from `120`. |
| `implementation_scope` | array | Yes | empty | Contracts, interfaces, algorithms, errors, defaults, and mappings covered. |
| `feedback_rule` | string | Yes | `spec_change_required` | Implementation discoveries that affect behavior must create a spec change before or alongside code. |

## Document registry

| ID | Path | Document class | Status | Owner | May drive implementation | Source-of-truth role | Cutover disposition |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `README` | docs/README.md | navigation | `migration_active` | 000 | No | Navigation only | Retained |
| `domain` | docs/nlspec/domain.md | candidate_spec | `migration_active` | domain | No | Root vocabulary and owner routing | Candidate root NLSpec |
| `000` | docs/nlspec/000-cadastre-spec-index.md | candidate_spec | `migration_active` | 000 | No | Governance owner | Candidate active NLSpec |
| `010` | docs/nlspec/010-system-boundary-and-authority.md | candidate_spec | `migration_active` | 010 | No | Product boundary owner | Candidate active NLSpec |
| `020` | docs/nlspec/020-lakehouse-feeds-and-table-state.md | candidate_spec | `migration_active` | 020 | No | Lakehouse feed owner | Candidate active NLSpec |
| `030` | docs/nlspec/030-processing-dag-lifecycle-and-versioning.md | candidate_spec | `migration_active` | 030 | No | DAG and lifecycle owner | Candidate active NLSpec |
| `040` | docs/nlspec/040-canonical-data-observation-and-fact-model.md | candidate_spec | `migration_active` | 040 | No | Core data owner | Candidate active NLSpec |
| `050` | docs/nlspec/050-normalization-external-schema-and-mapping.md | candidate_spec | `migration_active` | 050 | No | Normalization owner | Candidate active NLSpec |
| `060` | docs/nlspec/060-source-authority-completeness-coverage-and-absence.md | candidate_spec | `migration_active` | 060 | No | Authority and absence owner | Candidate active NLSpec |
| `070` | docs/nlspec/070-identity-resolution-and-target-selectors.md | candidate_spec | `migration_active` | 070 | No | Identity owner | Candidate active NLSpec |
| `080` | docs/nlspec/080-temporal-corrections-replay-and-gold-derivation.md | candidate_spec | `migration_active` | 080 | No | Temporal and gold owner | Candidate active NLSpec |
| `090` | docs/nlspec/090-graph-projection-serving-and-backends.md | candidate_spec | `migration_active` | 090 | No | Graph owner | Candidate active NLSpec |
| `100` | docs/nlspec/100-packages-supply-chain-and-activation.md | candidate_spec | `migration_active` | 100 | No | Package owner | Candidate active NLSpec |
| `110` | docs/nlspec/110-api-ux-health-errors-and-security.md | candidate_spec | `migration_active` | 110 | No | API and security owner | Candidate active NLSpec |
| `120` | docs/nlspec/120-validation-fixtures-and-acceptance.md | candidate_spec | `migration_active` | 120 | No | Validation owner | Candidate active NLSpec |
| `130` | docs/nlspec/130-analysis-enrichment-and-registry-governance.md | candidate_spec | `migration_active` | 130 | No | Analysis and registry owner | Candidate active NLSpec |
| `200` | docs/deferred/200-future-reachability-analysis-domain.md | deferred_spec | `inactive_deferred` | 200 | No | Inactive future reachability | Deferred |
| `PRD-ledger` | docs/migration/PRD-to-NLSpec-ledger.md | migration_support | `migration_active` | 000 | No | Migration evidence | Retained until archive |
| `gap-ledger` | docs/migration/gap-and-ambiguity-ledger.md | migration_support | `migration_active` | 000 | No | Blocker evidence | Retained until archive |
| `decision-ledger` | docs/migration/open-decision-ledger.md | migration_support | `migration_active` | 000 | No | Decision evidence | Retained until archive |
| `glossary` | docs/reference/glossary.md | reference | `migration_active` | domain | No | Navigation glossary | Retained |
| `research-index` | docs/reference/research-index.md | reference | `migration_active` | 000 | No | Research evidence index | Retained |
| `source-inventory` | docs/reference/source-inventory.md | reference | `migration_active` | 000 | No | Artifact inventory | Blocks promotion until reconciled |
| `nlspec-standard` | docs/reference/standards/nlspec-spec.md | reference | `active_standard` | 000 | No | Quality standard only | Retained |
| `ADR-0001` | docs/adr/ADR-0001-lakehouse-fed-boundary.md | rationale | `accepted-draft` | 010,020 | No | Rationale only | Retained |
| `ADR-0002` | docs/adr/ADR-0002-graph-as-derived-projection.md | rationale | `accepted-draft` | 090 | No | Rationale only | Retained |
| `ADR-0003` | docs/adr/ADR-0003-ocsf-as-silver-profile.md | rationale | `accepted-draft` | 050 | No | Rationale only | Retained |
| `ADR-0004` | docs/adr/ADR-0004-package-set-activation.md | rationale | `accepted-draft` | 100 | No | Rationale only | Retained |
| `research-reports` | docs/reference/research/*.md | reference | `research_report` | 000 | No | Evidence only | Retained |
| `PRD-archive` | docs/archive/PRD-Cadastre.revised-draft.md | archive | `revised-draft` | 000 | No | Upstream intent before cutover; historical trace after cutover | Archive after `MIG-AC-009` passes |

## Owner contract registry

| Contract | Owner spec | Imported by | Runtime authority class | Validation owner | Open blocker status |
| --- | --- | --- | --- | --- | --- |
| Documentation governance | `000` | README, MANIFEST, source-inventory, migration ledgers | governance | 120 | open |
| Root domain vocabulary | `domain` | glossary, all owner specs | vocabulary | 120 | open until domain TODOs close |
| Boundary and authority classes | `010` | 020,060,090,110,130 | runtime_boundary | 120 | open |
| Lakehouse feed and table-state contracts | `020` | 030,040,060,080,120 | runtime_data_input | 120 | open |
| DAG lifecycle and version manifest | `030` | 020,080,090,100,120 | runtime_orchestration | 120 | open |
| Canonical serialization and IDs | `040` | 020,030,050,060,070,080,090,110,120 | runtime_data_shape | 120 | open |
| External schema and mapping | `050` | 040,100,120,130 | runtime_normalization | 120 | open |
| Source authority and absence | `060` | 010,020,080,090,110,130 | runtime_authority | 120 | open |
| Identity resolution | `070` | 040,060,080,090,110 | runtime_identity | 120 | open |
| Temporal, gold, replay | `080` | 030,040,060,090,120 | runtime_gold | 120 | open |
| Graph projection and serving | `090` | 070,080,110,120,130 | derived_projection | 120 | open |
| Package-set activation | `100` | 030,050,090,110,120 | runtime_activation | 120 | open |
| API, errors, security | `110` | all owner specs | runtime_api | 120 | open |
| Validation and acceptance | `120` | 000 and all owner specs | validation | 120 | open |
| Analysis, enrichment, registry | `130` | 050,060,090,110 | non_authoritative_analysis | 120 | open |
| Deferred reachability | `200` | 090,110,120 | inactive_future_domain | 120 | deferred |

## Dependency rule

A spec may depend on another spec only through a named record, named interface, named algorithm, named error code, named lifecycle state, named acceptance report, or named validation matrix row.

A spec must not restate an imported contract except for a one-sentence reference and the exact imported name.

## Source inventory reconciliation rule

Any checksum, line count, uploaded path, canonical path, or alias mismatch in `docs/reference/source-inventory.md` blocks promotion to `candidate_authoritative` until the mismatch is corrected or recorded as an explicit alias with status `resolved_alias` or `unresolved_blocker`.

The uploaded PRD filename `PRD-Cadastre..md` is a source alias only. The canonical target path is `docs/archive/PRD-Cadastre.revised-draft.md`.

## Migration control

The migration ledgers may prove disposition and cutover state. They must not become behavior authority.

`docs/nlspec/120-validation-fixtures-and-acceptance.md` must define executable validation rows before this index may promote any active NLSpec to `authoritative`.

No document with unresolved `TODO:` rows, unresolved blocking ledger rows, unresolved product decisions, missing source-inventory reconciliation, or missing validation evidence may be marked `authoritative`.

## Archival policy

`docs/archive/PRD-Cadastre.revised-draft.md` remains upstream intent before cutover. It becomes historical trace only after `120` records passing `MIG-AC-009` evidence and this index records `cutover_accepted` or `post_cutover_authoritative`.

## Definition of Done

| ID | Criterion |
| --- | --- |
| `000-AC-001` | Every implementation-relevant behavior has exactly one owning spec. |
| `000-AC-002` | Every active spec declares imports, exports, dependencies, non-scope, and Definition of Done. |
| `000-AC-003` | The PRD is not cited as implementation authority after cutover. |
| `000-AC-004` | Research reports, ADRs, glossary, README, source inventory, and migration ledgers are marked non-authoritative for runtime behavior. |
| `000-PATCH-AC-001` | Every document in `MANIFEST.md` has one registry row or is explicitly outside the source-of-truth registry. |
| `000-PATCH-AC-002` | `docs/nlspec/domain.md` has exactly one status and target role. |
| `000-PATCH-AC-003` | No active spec has `authoritative` status while an owning decision or gap row remains open. |
| `000-PATCH-AC-004` | The PRD archive path and source-inventory path names are consistent. |

## Open questions

Open questions marked `TODO:` block authoritative status for the affected contract. A downstream implementation must not resolve a `TODO:` by inference.
