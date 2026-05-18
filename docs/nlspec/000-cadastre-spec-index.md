---
doc_id: CADASTRE-NLSPEC-000
title: Cadastre Spec Index and Source-of-Truth Map
doc_type: nlspec
status: candidate
---

## Authority

This document owns Cadastre documentation governance, document status vocabulary, document classes, source-of-truth ownership, dependency policy, promotion gates, document registry consistency, spec-set version records, and archival rules.

This document must not define product runtime behavior. Runtime behavior must be owned by the specific Cadastre NLSpec that exports the relevant contract.

## Purpose

Define the self-contained Cadastre documentation set, source-of-truth ownership, spec status vocabulary, dependency policy, acceptance dependencies, promotion rules, and archival rules.

## Explicit Non-Scope

- Domain runtime behavior.
- Data model fields except documentation-governance records.
- Package, graph, identity, source, API, or validation algorithms.
- Product decisions that are still open in owner-local `TODO:` rows.

## Imports

- `docs/reference/standards/nlspec-spec.md`

## Exports

- `SpecStatus`
- `DocumentClass`
- `SourceOfTruthRow`
- `SpecDependency`
- `SpecSetVersion`
- `PromotionGate`
- `DocumentRegistryConsistencyRule`
- `VolatilityClass`
- `VolatilityClassificationRow`
- `VolatilityRegistryConsistencyRule`

## Document Status Vocabulary

`SpecStatus` is a closed vocabulary for document authority. A status value not listed in this table must not be used by the document registry.

| Status | Meaning | May drive implementation |
| --- | --- | --- |
| `draft` | Authoring text that is not review-complete. | No. |
| `candidate` | Candidate NLSpec text that may be reviewed and validated. | No, except controlled validation or implementation spikes explicitly named by an acceptance report. |
| `authoritative` | Accepted NLSpec text whose validation criteria pass and whose registry row marks it active. | Yes, for owned contracts only. |
| `superseded` | Replaced by a newer registered version. | No. |
| `archived` | Retained for historical reference only. | No. |
| `accepted_rationale` | Accepted decision rationale with no runtime authority. | No. |
| `inactive_deferred` | Preserved future-domain candidate with no active MVP effect. | No. |
| `active_standard` | Reference standard for NLSpec quality only. | No product runtime authority. |

## Document Class Registry

| Document class | Scope | Runtime authority | Promotion rule |
| --- | --- | ---: | --- |
| `authoritative_spec` | Active NLSpecs with status `authoritative`. | Yes, for owned contracts only. | Requires passing `120` evidence, no unresolved owner `TODO:` rows, and an active registry row. |
| `candidate_spec` | NLSpecs under candidate review. | No. | May become `authoritative_spec` only through `PromotionGate`. |
| `deferred_spec` | Inactive future-domain material. | No. | Requires future accepted promotion, owners, decisions, and fixtures. |
| `reference` | Research reports, standards, cleanup references, and other evidence documents. | No. | Must not be promoted without conversion into an owning NLSpec. |
| `rationale` | ADRs and decision rationale. | No. | Remains rationale only. |
| `archive` | Superseded product drafts and historical documents. | No. | Historical reference only. |
| `navigation` | Manifest and navigation documents. | No. | Must point to this index for status when status is needed. |

Reference and rationale documents never provide runtime authority unless their content is converted into an owning NLSpec that exports the relevant contract and passes acceptance.

## Authoritative Source Rule

Implementation authority must be determined in this order:

1. Active Cadastre NLSpecs listed in this index with status `authoritative`.
2. Validation and acceptance reports referenced by `docs/nlspec/120-validation-fixtures-and-acceptance.md` for acceptance state only.
3. ADRs only for rationale, never for runtime behavior.
4. Research reports only as supporting evidence, never as runtime authority.
5. Archived product documents only for historical reference, never for implementation authority.

Until this index marks a document `authoritative`, the NLSpec set remains candidate text and must not be treated as production implementation authority except for controlled validation or implementation spikes explicitly named by an acceptance report.

## Promotion Gates

| From status | To status | Required gate | Blocking conditions |
| --- | --- | --- | --- |
| `candidate` | `authoritative` | Passing owner `AcceptanceReport`; no unresolved owner `TODO:` rows; registry row marks the file active. | Failed, blocked, missing, or stale validation evidence; unresolved owner `TODO:` rows; duplicate owner for a runtime contract. |
| `authoritative` | `superseded` | New accepted `SpecSetVersion` names the replacement. | Missing replacement checksum or missing registry row. |

## SpecSetVersion Record

| Field | Type | Required | Default | Rule |
| --- | --- | ---: | --- | --- |
| `spec_set_version` | string | Yes | none | Monotonic version assigned by this index. |
| `spec_set_checksum` | sha256 string | Yes for authoritative handoff | none | Computed over included authoritative docs using canonical path order and exact bytes. |
| `included_docs` | array of path/checksum rows | Yes | empty | Must include every authoritative doc. |
| `deferred_docs` | array of path/checksum rows | Yes | empty | Must include `docs/deferred/200-future-reachability-analysis-domain.md` while it remains deferred. |
| `acceptance_report_id` | string | Yes for authoritative handoff | none | Must reference a passing `AcceptanceReport`. |
| `supersedes` | string or null | No | null | Must reference the prior spec-set version when replacing it. |
| `effective_from` | timestamp or null | Yes for authoritative handoff | null | RFC3339 UTC timestamp assigned when the spec-set version becomes authoritative. |
| `open_exclusions` | array | Yes | empty | Explicit deferred or unresolved issues implementation must not infer. |
| `volatility_registry_checksum` | sha256 string | Yes for authoritative handoff | none | SHA-256 over canonical volatility classification rows in path and artifact order. |
| `activation_artifact_registry_refs` | array | Yes | empty | Refs to activation-controlled artifact registries included in the spec-set version. |
| `open_volatility_exclusions` | array | Yes | empty | Explicit volatility classification exclusions that implementation must not infer. |
| `validation_matrix_refs` | array | Yes | empty | Required owner validation rows from `120`. |
| `implementation_scope` | array | Yes | empty | Contracts, interfaces, algorithms, errors, defaults, and mappings covered. |
| `feedback_rule` | string | Yes | `spec_change_required` | Implementation discoveries that affect behavior must create a spec change before or alongside code. |

## Volatility Classification Governance

`VolatilityClass` is a closed governance vocabulary. Every implementation-relevant exported contract, profile, row set, package, fixture, validation artifact, registry artifact, and runtime state class must have exactly one volatility classification.

| Volatility class | May define runtime behavior | May be separately activated | Must appear in `VersionManifest` | Must have lifecycle status | May appear in `ProductionPackageSetManifest` | Required owner |
| --- | --- | --- | --- | --- | --- | --- |
| `stable_core_contract` | Yes, only through the authoritative owner NLSpec. | No. | Yes when the contract version affects output or replay. | Yes when promotion or runtime eligibility depends on lifecycle. | No. | Owner NLSpec registered in `000`. |
| `activation_controlled_artifact` | No. It may instantiate exported stable behavior only. | Yes. | Yes when it can affect output, validation, visibility, replay, or activation. | Yes. | Yes when package-supplied. | Stable owner NLSpec plus artifact owner row. |
| `runtime_state_record` | No. It records execution, replay, commit, snapshot, lifecycle, health, or validation state. | No. | Yes when output-affecting or replay-affecting. | Only when the state record itself has production eligibility semantics. | No, except as referenced evidence. | Domain owner and `030.VersionManifest`. |
| `reference_evidence` | No. | No. | No, except as cited non-runtime evidence in acceptance material. | No. | No. | `000`. |
| `rationale` | No. | No. | No. | No. | No. | `000`. |
| `inactive_future_domain` | No while inactive. | No while inactive. | Deferred docs must be recorded in `SpecSetVersion.deferred_docs`. | Future activation requires owner assignment. | No while inactive. | `000` and deferred owner. |

Rules:

- `stable_core_contract` may define runtime behavior only when its owner NLSpec is authoritative.
- `activation_controlled_artifact` must not define new runtime behavior. It may instantiate behavior exported by a stable core contract.
- `runtime_state_record` records execution, replay, commit, snapshot, lifecycle, or health state and must not create authority by itself.
- `reference_evidence`, `rationale`, and `inactive_future_domain` must not drive runtime behavior.
- Every `Owner Contract Registry` row must include `volatility_class`.
- Every exported contract, profile, row set, package, fixture, validation artifact, registry artifact, and runtime state class must have exactly one volatility classification.
- If an activation-controlled artifact requires behavior not exported by its owner stable core contract, promotion or activation must fail.

`VolatilityClassificationRow` fields are `artifact_or_contract`, `volatility_class`, `owner_spec`, `stable_core_owner`, `may_affect_output`, `version_manifest_requirement`, `package_set_manifest_requirement`, `lifecycle_requirement`, and `validation_row_refs`.

`VolatilityRegistryConsistencyRule` must fail before promotion using the most specific failure code.

| Failure code | Emitted when |
| --- | --- |
| `VOLATILITY_CLASS_MISSING` | An implementation-relevant artifact or contract lacks a volatility class. |
| `VOLATILITY_CLASS_CONFLICT` | More than one volatility class is assigned to the same artifact or contract. |
| `VOLATILE_ROW_IN_STABLE_CORE` | A stable core spec contains production-active volatile row instances rather than non-normative examples or blocking `TODO:` rows. |
| `ACTIVATION_ARTIFACT_OWNER_MISMATCH` | An activation-controlled artifact does not reference the stable core owner contract that exports its behavior. |
| `ACTIVATION_ARTIFACT_RUNTIME_RESTATEMENT` | An activation-controlled artifact restates or defines runtime behavior instead of instantiating owner-exported behavior. |

## Document Registry

| ID | Path | Document class | Status | Owner | May drive implementation | Source-of-truth role |
| --- | --- | --- | --- | --- | --- | --- |
| `MANIFEST` | MANIFEST.md | navigation | `draft` | 000 | No | Path inventory and navigation only. |
| `domain` | docs/nlspec/domain.md | candidate_spec | `candidate` | domain | No | Root vocabulary and owner routing. |
| `000` | docs/nlspec/000-cadastre-spec-index.md | candidate_spec | `candidate` | 000 | No | Governance owner. |
| `010` | docs/nlspec/010-system-boundary-and-authority.md | candidate_spec | `candidate` | 010 | No | Product boundary owner. |
| `020` | docs/nlspec/020-lakehouse-feeds-and-table-state.md | candidate_spec | `candidate` | 020 | No | Lakehouse feed owner. |
| `030` | docs/nlspec/030-processing-dag-lifecycle-and-versioning.md | candidate_spec | `candidate` | 030 | No | DAG and lifecycle owner. |
| `040` | docs/nlspec/040-canonical-data-observation-and-fact-model.md | candidate_spec | `candidate` | 040 | No | Core data owner. |
| `050` | docs/nlspec/050-normalization-external-schema-and-mapping.md | candidate_spec | `candidate` | 050 | No | Normalization owner. |
| `060` | docs/nlspec/060-source-authority-completeness-coverage-and-absence.md | candidate_spec | `candidate` | 060 | No | Authority and absence owner. |
| `070` | docs/nlspec/070-identity-resolution-and-target-selectors.md | candidate_spec | `candidate` | 070 | No | Identity owner. |
| `080` | docs/nlspec/080-temporal-corrections-replay-and-gold-derivation.md | candidate_spec | `candidate` | 080 | No | Temporal and gold owner. |
| `090` | docs/nlspec/090-graph-projection-serving-and-backends.md | candidate_spec | `candidate` | 090 | No | Graph owner. |
| `100` | docs/nlspec/100-packages-supply-chain-and-activation.md | candidate_spec | `candidate` | 100 | No | Package owner. |
| `110` | docs/nlspec/110-api-ux-health-errors-and-security.md | candidate_spec | `candidate` | 110 | No | API and security owner. |
| `120` | docs/nlspec/120-validation-fixtures-and-acceptance.md | candidate_spec | `candidate` | 120 | No | Validation owner. |
| `130` | docs/nlspec/130-analysis-enrichment-and-registry-governance.md | candidate_spec | `candidate` | 130 | No | Analysis and registry owner. |
| `200` | docs/deferred/200-future-reachability-analysis-domain.md | deferred_spec | `inactive_deferred` | 200 | No | Inactive future reachability. |
| `nlspec-standard` | docs/reference/standards/nlspec-spec.md | reference | `active_standard` | 000 | No | Quality standard only. |
| `ADR-0001` | docs/adr/ADR-0001-lakehouse-fed-boundary.md | rationale | `accepted_rationale` | 010,020 | No | Accepted rationale only; runtime owned by `010` and `020`. |
| `ADR-0002` | docs/adr/ADR-0002-graph-as-derived-projection.md | rationale | `accepted_rationale` | 090 | No | Accepted rationale only; runtime owned by `090`. |
| `ADR-0003` | docs/adr/ADR-0003-ocsf-as-silver-profile.md | rationale | `accepted_rationale` | 050 | No | Accepted rationale only; runtime owned by `050`. |
| `ADR-0004` | docs/adr/ADR-0004-package-set-activation.md | rationale | `accepted_rationale` | 100 | No | Accepted rationale only; runtime owned by `100`. |
| `research-reports` | docs/reference/research/*.md | reference | `draft` | 000 | No | Evidence only. |
| `doc-cleanup` | docs/doc_clean-up/*.md | reference | `draft` | 000 | No | Formatting reference only. |
| `product-archive` | docs/archive/PRD-Cadastre.revised-draft.md | archive | `archived` | 000 | No | Historical reference only. |

## Owner Contract Registry

| Contract | Owner spec | Imported by | Runtime authority class | Volatility class | Validation owner | Owner-local closure state | Open blocker status |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Documentation governance | `000` | `MANIFEST.md` and registered documentation artifacts | governance | `stable_core_contract` | 120 | blocked_validation | open until registry consistency rows pass |
| Root domain vocabulary | `domain` | all owner specs | vocabulary | `stable_core_contract` | 120 | blocked_owner_todo | open until domain TODOs close |
| Boundary and authority classes | `010` | 020,060,090,110,130 | runtime_boundary | `stable_core_contract` | 120 | closed_local | validation rows required |
| Lakehouse feed and table-state contracts | `020` | 030,040,060,080,120 | runtime_data_input | `stable_core_contract` | 120 | blocked_validation | open until manifest fixtures and checksum rows close |
| RawFeedManifest | `020` | 030,040,060,120 | runtime_data_input | `runtime_state_record` | 120 | closed_local | validation fixture checksums required |
| Raw record identity import | `020` | 040,120,domain | runtime_data_input | `stable_core_contract` | 120 | closed_local | imports `040.ComputeRawRecordId`; no local ID order blocker |
| DAG lifecycle and version manifest | `030` | 020,080,090,100,120 | runtime_orchestration | `stable_core_contract` | 120 | blocked_owner_todo | run-lock lease timing and lifecycle transition rows remain TODO |
| ActivationControlledArtifactRef | `030` | 020,050,060,070,080,090,100,110,120,130 | runtime_orchestration | `stable_core_contract` | 120 | blocked_validation | activation artifact ref validation rows required |
| Core record schema registry, scalar registry, canonical serialization, IDs, checksums, omission states, and evidence refs | `040` | 020,030,050,060,070,080,090,110,120 | runtime_data_shape | `stable_core_contract` | 120 | blocked_validation | owner enum and one-of validation rows required |
| ComputeRawRecordId | `040` | 020,120,domain | runtime_data_shape | `stable_core_contract` | 120 | closed_local | validation fixture checksums required |
| External schema and mapping | `050` | 040,100,120,130 | runtime_normalization | `stable_core_contract` | 120 | blocked_owner_todo | exact OCSF rows and artifact checksums remain TODO |
| Source authority and absence | `060` | 010,020,080,090,110,130 | runtime_authority | `stable_core_contract` | 120 | blocked_owner_todo | source-specific coverage rows remain TODO |
| Identity resolution | `070` | 040,060,080,090,110 | runtime_identity | `stable_core_contract` | 120 | blocked_owner_todo | unsupported entity and candidate cap owner decisions remain TODO |
| Temporal, gold, replay | `080` | 030,040,060,090,120 | runtime_gold | `stable_core_contract` | 120 | blocked_owner_todo | temporal, correction, late-arrival, and replay rows remain TODO |
| Graph projection and serving | `090` | 070,080,110,120,130 | derived_projection | `stable_core_contract` | 120 | blocked_owner_todo | graph apply numeric defaults and validation checksums remain TODO |
| Package-set activation | `100` | 030,050,090,110,120 | runtime_activation | `stable_core_contract` | 120 | blocked_owner_todo | package deprecation windows remain TODO |
| API, errors, security | `110` | all owner specs | runtime_api | `stable_core_contract` | 120 | blocked_owner_todo | page-token expiration bounds remain TODO |
| Validation and acceptance | `120` | 000 and all owner specs | validation | `stable_core_contract` | 120 | blocked_validation | fixture and expected-output checksums remain TODO |
| Analysis, enrichment, registry | `130` | 050,060,090,110 | non_authoritative_analysis | `stable_core_contract` | 120 | blocked_validation | fixture checksums required |
| Deferred reachability | `200` | 090,110,120 | inactive_future_domain | `inactive_future_domain` | 120 | inactive_deferred | deferred |

Owner-local closure states are closed values: `closed_local`, `blocked_owner_todo`, `blocked_validation`, `inactive_deferred`, and `candidate_not_promoted`. A `closed_local` owner contract may still be prevented from promotion by candidate document status or missing validation evidence.

## Dependency Rule

A spec may depend on another spec only through a named record, named interface, named algorithm, named error code, named lifecycle state, named acceptance report, or named validation matrix row.

A spec must not restate an imported contract except for a one-sentence reference and the exact imported name.

### Core schema dependency map

| Importing spec | Required 040 imports |
| --- | --- |
| `020` | `RawRecordSchema`, `ComputeRawRecordId`, `ValidateCoreRecord`. |
| `030` | `CoreRecordSchema`, `CoreRecordValidationAlgorithm`, `CoreRecordChecksumPolicy`. |
| `050` | `CadastreSilverObservationSchema`, `ValidateCoreRecord`. |
| `060` | `GoldFactSchema`, `FactAbsenceOutcome`, `EvidenceRef`. |
| `070` | `CanonicalEntitySchema`, `SourceAssetSchema`, `IdentifierSchema`. |
| `080` | `GoldFactSchema`, `ComputeGoldFactKeyId`, `ComputeGoldFactId`. |
| `090` | `GraphNodeDeltaShape`, `GraphEdgeDeltaShape`, `EvidenceRef`. |
| `110` | `CoreRecordErrorCodeSet`, `EvidenceRef`, `CommonRecordHeader`. |
| `120` | All exported record schemas and validation algorithms. |
| `docs/nlspec/domain.md` | Vocabulary and owner routing only. |

## Document Registry Consistency Rule

Every registry row must reference a normalized repository-relative path or explicit glob. A registry path must not contain an absolute prefix, backslash, empty segment, `.` segment, or `..` segment.

Every target file in the NLSpec set must have exactly one registry row. Duplicate rows for the same target path are invalid. A manifest path outside the NLSpec set may be represented by an explicit reference, rationale, archive, or navigation row.

No non-040 file may define a field schema for a core record exported by `040`. A downstream spec may import an exact schema name and define behavior for its own stage, but it must not restate field paths, defaults, nullability, ID inputs, checksum inputs, or extension policy for the 040-owned record.

### OwnerLocalStatusConsistencyRule

`ValidateSpecSet` must evaluate owner-local closure state independently from document authority status. The following failure codes are closed for this rule.

| Failure code | Emitted when |
| --- | --- |
| `DOMAIN_OWNER_STATUS_CONTRADICTION` | `domain.md` marks a named behavior unresolved while every named owner contract is `closed_local`, or marks it resolved while any named owner-local blocker remains. |
| `DOMAIN_RUNTIME_RESTATEMENT` | `domain.md` copies a downstream schema, ID input order, default, nullability rule, checksum input set, or collision behavior owned by another spec instead of routing by exact owner contract name. |
| `OWNER_SPEC_CONTRADICTION` | Two owner specs define incompatible runtime behavior for the same named contract. |
| `ADR_STATUS_UNREGISTERED` | An ADR file or registry row uses a status outside `SpecStatus`. |
| `REGISTRY_MANIFEST_PATH_MISMATCH` | A manifest path lacks exactly one registry row or explicit non-registry reason. |
| `VOLATILITY_CLASS_MISSING` | An implementation-relevant artifact or contract lacks exactly one volatility class. |
| `VOLATILITY_CLASS_CONFLICT` | An artifact or contract has conflicting volatility classes. |
| `VOLATILE_ROW_IN_STABLE_CORE` | A stable core spec contains production-active volatile rows rather than non-normative examples or blocking `TODO:` rows. |
| `ACTIVATION_ARTIFACT_OWNER_MISMATCH` | An activation-controlled artifact lacks a matching owner stable core contract. |
| `ACTIVATION_ARTIFACT_RUNTIME_RESTATEMENT` | An activation-controlled artifact defines runtime behavior instead of instantiating exported owner behavior. |

Required consistency checks:

| Check | Required behavior |
| --- | --- |
| Domain unresolved and owners closed | Fail with `DOMAIN_OWNER_STATUS_CONTRADICTION`. |
| Domain resolved and owner blocked | Fail with `DOMAIN_OWNER_STATUS_CONTRADICTION`. |
| Domain runtime restatement | Fail with `DOMAIN_RUNTIME_RESTATEMENT`; `domain.md` must use owner-routing language only. |
| Owner runtime contradiction | Fail with `OWNER_SPEC_CONTRADICTION`; `domain.md` must not pick a side. |
| ADR status mismatch | Fail with `ADR_STATUS_UNREGISTERED` unless both file and registry row use `accepted_rationale`. |
| Manifest path mismatch | Fail with `REGISTRY_MANIFEST_PATH_MISMATCH`; every manifest path must have one registry row or explicit non-registry reason. |

## Registry and Acceptance Control

No document may be marked `authoritative` when it has unresolved owner `TODO:` rows, duplicate runtime ownership, missing validation evidence, or a missing registry entry.

Promotion to `authoritative` must include the `120.CoreRecordSchemaValidationMatrix` rows for every `040` exported record in `SpecSetVersion.validation_matrix_refs`. Unresolved owner `TODO:` rows in core schemas, owner error codes, fixture checksums, expected output checksums, or downstream cross-references remain implementation exclusions.

A document that is not marked `authoritative` must not be used as product runtime authority. Candidate documents may be used only for validation or implementation spikes explicitly named by an acceptance report.

`ValidateSpecSet` must fail before promotion when any exported implementation-relevant artifact lacks a volatility class, an activation-controlled artifact has no owner spec, a volatile production row is embedded in a stable core section without being marked example-only or `TODO:`, or a reference, ADR, or archive document is used as runtime authority.

## Archival Policy

Archived documents are historical reference only and never implementation authority.

## Definition of Done

| ID | Criterion |
| --- | --- |
| `000-AC-001` | Every implementation-relevant behavior has exactly one owning spec. |
| `000-AC-002` | Every active spec declares imports, exports, dependencies, non-scope, and Definition of Done. |
| `000-AC-003` | Archived and reference documents are not cited as implementation authority. |
| `000-AC-004` | Research reports, ADRs, navigation files, and manifest files are marked non-authoritative for runtime behavior. |
| `000-CLEANUP-AC-001` | The file contains no banned reference class. |
| `000-CLEANUP-AC-002` | `SpecStatus` is total for `draft`, `candidate`, `authoritative`, `superseded`, `archived`, `accepted_rationale`, `inactive_deferred`, and `active_standard`. |
| `000-CLEANUP-AC-003` | No export name contains a banned reference class. |
| `000-CLEANUP-AC-004` | The registry has one row for each target file and no row for decomposition-control artifacts. |
| `000-CLEANUP-AC-005` | Promotion to `authoritative` depends only on registry state, owner review, and passing validation evidence. |
| `000-SCHEMA-PATCH-AC-001` | The owner registry names `040` as the sole owner of exported core record schemas. |
| `000-SCHEMA-PATCH-AC-002` | No downstream spec is registered as owner of a 040 core record field schema. |
| `000-SCHEMA-PATCH-AC-003` | Promotion to `authoritative` depends on passing 040 schema validation rows in `120`. |
| `000-VOLATILITY-AC-001` | Every implementation-relevant artifact has exactly one volatility class. |
| `000-VOLATILITY-AC-002` | Activation-controlled artifacts reference an owner stable core contract. |
| `000-VOLATILITY-AC-003` | Stable core specs do not contain production-active volatile row instances. |
| `000-VOLATILITY-AC-004` | Research, ADR, reference, and archive documents remain non-runtime even when cited as evidence. |

| `000-STATUS-CONSISTENCY-AC-001` | `ValidateSpecSet` fails domain/owner closure-state contradictions with `DOMAIN_OWNER_STATUS_CONTRADICTION`. |
| `000-STATUS-CONSISTENCY-AC-002` | `ValidateSpecSet` fails runtime restatement in `domain.md` with `DOMAIN_RUNTIME_RESTATEMENT`. |
| `000-STATUS-CONSISTENCY-AC-003` | `ValidateSpecSet` fails owner-spec runtime contradictions with `OWNER_SPEC_CONTRADICTION`. |
| `000-STATUS-CONSISTENCY-AC-004` | ADR status values in files and registry rows are members of `SpecStatus` and use `accepted_rationale` for accepted non-runtime rationale. |
| `000-STATUS-CONSISTENCY-AC-005` | Every manifest path has exactly one registry row or explicit non-registry reason. |

## Open Questions

Open questions marked `TODO:` block authoritative status for the affected contract. A downstream implementation must not resolve a `TODO:` by inference.
