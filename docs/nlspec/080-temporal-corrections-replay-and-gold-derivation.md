---
doc_id: CADASTRE-NLSPEC-080
title: Temporal Corrections, Replay, and Gold Derivation
doc_type: nlspec
status: candidate
---

## Authority

This document owns the contracts listed in `Exports`. Other Cadastre NLSpecs may import those contracts by exact name and must not restate them. This document has implementation authority only after the document registry marks it `authoritative` and its acceptance criteria pass.

## Purpose

Define temporal semantics, bitemporal facts, late arrivals, corrections, replay equivalence, and deterministic gold derivation.

## Explicit Non-Scope

- Graph apply implementation.
- Backend query translation.
- Package trust.
- External schema mappings.

## Imports

- `RawRecord`
- `CadastreSilverObservation`
- `GoldFact`
- `SourceAuthorityProfile`
- `LakehouseSnapshotRef`
- `DatasetVersionRef`
- `VersionManifest`
- `IdentityDecision`
- `TelemetryReplayExclusionPolicy`
- `TelemetryRuntimeState`

- `GoldFactSchema`
- `ComputeGoldFactKeyId`
- `ComputeGoldFactId`
- `CoreRecordValidationAlgorithm`
- `ActivationControlledArtifactRef`
- `GoldFactSubjectRefKindRegistry`
- `GoldFactObjectValueKindRegistry`
- `030.ScopeSelector`
- `030.ActivationScope`
- `030.ScopeSelectorContext`
- `030.ResolveScopedRow`
- `030.ActivationControlledRowSchema`
- `030.ActivationControlledRowField`
- `030.ActivationControlledRowRef`
- `030.ActivationControlledRowSetSchema`
- `020.SourceDatasetCatalogRow`

## Exports

- `TemporalSemanticsPolicy`
- `KnowledgeTimeImportPolicy`
- `TemporalObservationTimeResolution`
- `LateArrivalPolicy`
- `BitemporalQueryMode`
- `GoldFactCorrectionPolicy`
- `GoldFactChangeSet`
- `CorrectionSnapshotRefPolicy`
- `ReplayEquivalencePolicy`
- `ReplayInputSufficiencyCheck`
- `GraphRebuildEquivalencePolicy`
- `EventSequenceValidationCorpus`
- `ResolveFactTime`
- `EvaluateLateArrival`
- `ApplyGoldCorrection`
- `DeriveFacts`
- `ComputeReplayEquivalenceChecksum`
- `LifecycleEvidenceReplayEquivalence`
- `GoldFactPredicateContractRow`
- `MVPGoldFactPredicateContractRowSetClosure`
- `MVPGoldFactStructuredValueSchemaCatalog`
- `GoldFactPredicateBlockRowSemantics`

## Temporal Axes

Cadastre must keep source event time, source observation time, supplier collection time, supplier delivery time, lakehouse commit time, table snapshot time, CDC connector time, CDC heartbeat time, schema-history time, graph apply time, replay time, and platform current time distinct.

A `GoldFact` candidate may be created only after `ResolveFactTime` emits exactly one `TemporalObservationTimeResolution` row or a deterministic temporal error.

## Temporal and replay artifact activation boundary

Temporal, gold, correction, late-arrival, and replay algorithms are stable core contracts. Source/fact-specific temporal rows, correction rows, late-arrival rows, snapshot-ref rows, replay output-class rows, and event-sequence validation corpora are activation-controlled artifacts. Runtime state records produced by these contracts must appear in `VersionManifest` when they can affect output, replay, graph projection, API output, export output, audit reconstruction, or validation acceptance.

| Artifact or contract | Volatility class | Required handling |
| --- | --- | --- |
| `TemporalSemanticsPolicy` | `activation_controlled_artifact` | Required before fact-time resolution; one active row must cover each gold-producing tuple or derivation must emit `TEMPORAL_POLICY_UNRESOLVED`. |
| `KnowledgeTimeImportPolicy` | stable mode definitions plus activation-controlled policy rows | Production default is `current_import`; historical known-time reconstruction is rejected unless an active row permits it and required persisted source-known-time evidence validates. |
| `TemporalObservationTimeResolution` | `runtime_state_record` | Exactly one row, or exactly one deterministic temporal error, must exist before any `GoldFact` candidate is created. |
| `LateArrivalPolicy` | `activation_controlled_artifact` | Required before late evidence routing; omitted rows block route selection with `LATE_ARRIVAL_POLICY_MISSING`. |
| `BitemporalQueryMode` | `stable_core_contract` | Query mode semantics are stable; optional visibility rows may narrow output but must not redefine bitemporal axes. |
| `GoldFactCorrectionPolicy` | `activation_controlled_artifact` | Required before correction behavior; missing rows block with `CORRECTION_POLICY_MISSING`. |
| `GoldFactChangeSet` | `runtime_state_record` | System-of-record correction change record; no existing `GoldFact` row may be mutated in place. |
| `CorrectionSnapshotRefPolicy` | `activation_controlled_artifact` | Required for every correction class; old and new snapshot refs plus table-set checksum are mandatory. |
| `ReplayEquivalencePolicy` | stable checksum algorithm plus activation-controlled output-class rows | Missing output-class row blocks replay with `REPLAY_POLICY_ARTIFACT_MISSING`. |
| `ReplayInputSufficiencyCheck` | `stable_core_contract` | Preflight algorithm; required refs are determined by owner output class rows and `030.VersionManifest`. |
| `GraphRebuildEquivalencePolicy` | split ownership with `090`; row instances activation-controlled where output-specific | Graph rebuild equivalence must use `080` replay failure precedence and `090` graph-specific included fields. |
| `EventSequenceValidationCorpus` | `activation_controlled_artifact` | Required validation artifact for temporal resolution, late arrival, correction, replay, graph handoff, no-op, and error cases. |
| `ResolveFactTime`, `EvaluateLateArrival`, `ApplyGoldCorrection`, `DeriveFacts`, `ComputeReplayEquivalenceChecksum` | `stable_core_contract` | Algorithms validate required activation refs before output and emit deterministic no-op or error records for omitted cases. |

Production promotion must fail when any `TemporalSemanticsPolicy`, `GoldFactCorrectionPolicy`, `LateArrivalPolicy`, `ReplayEquivalencePolicy`, `CorrectionSnapshotRefPolicy`, or assertion-state transition behavior contains a blocking placeholder row.

### StructuredInputRepositoryTemporalPolicyHandoff

Repository-authored temporal semantics policies, correction policies, late-arrival policies, replay output-class rows, predicate contract rows, event-sequence corpora, assertion transition rows, and gold derivation row catalogs are inert until materialized, exact-snapshot validated, package-set activated when package-supplied, and included in `030.VersionManifest`.

Repository snapshots may appear in replay provenance and validation diagnostics, but they must not satisfy replay equivalence, correction authorization, temporal policy selection, predicate contract selection, assertion transition authority, or gold derivation authority by themselves.

`ReplayInputSufficiencyCheck` must require structured-input snapshot refs, file manifest checksums, validation refs, materialization refs when packaged, package-set refs when package-supplied, and owner artifact refs whenever output-affecting temporal, correction, late-arrival, replay, predicate, assertion, or event-sequence rows were repository-authored.

For repository-authored artifacts, `ReplayInputSufficiencyCheck` must also require repository template contract refs when required, producer CI validation refs when accepted, maintenance tool contract and invocation refs when policy rows or corpora are generated, publication manifest refs when artifacts are published, candidate sync record refs when imported by sync, and package release plus package-set refs when package-supplied.

If replay output depends on a repository-authored artifact and any workflow ref is missing, stale, checksum-mismatched, package-set-mismatched, unmanifested, or private-leaking, replay must fail before output with the most specific repository workflow error. Sync-only replay attempts and publication-only replay attempts must emit no replay output, correction output, gold derivation output, graph handoff output, watermark mutation, or validation pass.

| Error code | Required use |
| --- | --- |
| `TEMPORAL_REPOSITORY_POLICY_INACTIVE` | Repository-authored temporal, correction, replay, predicate, assertion, or event-sequence policy row exists only in Git or lacks active materialized refs. |
| `REPLAY_REPOSITORY_REF_MISSING` | Replay input sufficiency requires structured-input refs but snapshot, validation, materialization, package, or manifest refs are omitted. |
| `REPLAY_REPOSITORY_VALIDATION_STALE` | Replay, correction, temporal, or event-sequence validation was not run against the exact repository snapshot and policy checksum used for output. |
| `REPLAY_REPOSITORY_TOOL_REF_MISSING` | Replay depends on generated repository-authored artifact bytes but the maintenance tool contract or invocation ref is missing. |
| `REPLAY_REPOSITORY_PUBLICATION_MANIFEST_MISMATCH` | Publication manifest does not match replay, temporal, predicate, assertion, corpus, materialization, package, or validation refs. |
| `REPLAY_REPOSITORY_SYNC_NONAUTHORITY` | Candidate sync record is used as replay, correction, temporal, predicate, assertion, event-sequence, or gold derivation authority. |

### TemporalPolicyScopeSelectorContext

`TemporalPolicyScopeSelectorContext` is the owner context for `TemporalSemanticsPolicy.source_scope_selector`. It instantiates `030.ScopeSelectorContext`; time fields, time-input precedence, fallback behavior, temporal error precedence, and known-time rules remain owned by `080`.

Default temporal scope matching is exact. `subset_allowed_dimension_keys` defaults to `[]`, and no broad source-category fallback is implicit. A temporal policy that omits a request dimension fails with the owner-mapped `TEMPORAL_POLICY_UNRESOLVED` unless an active context row explicitly permits that dimension to be subset-matched.

Shared selector ambiguity maps to `TEMPORAL_POLICY_UNRESOLVED` with ambiguity context. The ambiguity context may include selector checksums, context ref, row family, and redacted dimension keys; it must not include raw private selector values.

## ResolveFactTime Algorithm

```text
ResolveFactTime(observation, temporal_policy_row_set, knowledge_time_policy_row_set, version_manifest):
1. Validate required temporal and knowledge-time artifact refs through `030.ActivationControlledArtifactRef`.
2. Apply exact non-scope predicates for `source_dataset`, `observation_type`, `fact_type`, `predicate`, and `temporal_mode`, then call `030.ResolveScopedRow` over `TemporalSemanticsPolicy.source_scope_selector` using `TemporalPolicyScopeSelectorContext`.
3. If zero rows resolve, emit `TEMPORAL_POLICY_UNRESOLVED` and do not create a `GoldFact` candidate.
4. If more than one maximal row remains under `030.CompareScopeSpecificity`, emit `TEMPORAL_POLICY_UNRESOLVED` with ambiguity context and do not create a `GoldFact` candidate.
5. Validate `valid_interval_model`, `valid_time_input_precedence`, `time_quality_requirements`, and the selected `KnowledgeTimeImportPolicy` row.
6. Select the first valid-time input in precedence order that is authorized by the row, present, parseable, unambiguous, and quality-sufficient.
7. Reject supplier collection time, supplier delivery time, table snapshot time, lakehouse commit time, CDC offset time, heartbeat time, schema-history time, graph apply time, replay time, and current platform time unless the resolved row explicitly permits that exact field for the requested temporal mode.
8. Apply `absent_time_behavior`, `malformed_time_behavior`, `ambiguous_time_behavior`, and `fallback_behavior` exactly as declared by the resolved row. Omitted fallback means deterministic temporal error.
9. Select `known_from` using `KnowledgeTimeImportPolicy`. Production default `current_import` uses the persisted Cadastre import knowledge time from evidence, not a live wall-clock read.
10. Reject reconstructed historical known time unless `KnowledgeTimeImportPolicy.as_known_then_allowed = true` and every required persisted source-known-time evidence ref validates.
11. Materialize `valid_to` using `valid_to_rule`. Null `valid_to` means open interval only when the row permits an open interval.
12. Emit exactly one `TemporalObservationTimeResolution` row with selected input path/value, resolved valid interval, resolved known time, fallback flag, quality, error code when any, checksum, policy refs, source refs, and `VersionManifest` ref.
13. Include the temporal policy row ref, knowledge-time policy row ref, temporal resolution ref, temporal resolution checksum, and every selected source evidence ref in `VersionManifest` before gold output.
```

Implicit current-time fallback is forbidden.

## Gold Derivation

`DeriveFacts` must validate required temporal, authority, correction, late-arrival, replay, predicate-contract, and schema artifact refs before output. It must consume silver observations, identity decisions, source authority decisions, coverage assertions, temporal resolutions, active derivation profiles, active `GoldFactPredicateContractRow` refs, and absence derivation results where applicable. It must resolve exactly one active predicate contract row before `gold_fact_key_id` computation. It must compute `gold_fact_key_id` through `040.ComputeGoldFactKeyId` from subject, predicate, object/value, and valid interval only after `GoldFact.subject_ref`, `GoldFact.object_value`, and `GoldFact.object_kind` pass the `040` one-of rules and the selected predicate contract. It must compute immutable `gold_fact_id` through `040.ComputeGoldFactId` from `gold_fact_key_id`, `known_from`, assertion state, authority row, temporal resolution, evidence set, and correction policy. It must emit `GoldFact`, `GoldFactChangeSet`, explicit no-op, conflict, or deterministic error. Every emitted `GoldFact` must pass `040.GoldFactSchema` before persistence or graph projection.

When a candidate gold fact depends on missing evidence, stale evidence, source-declared deletion, cleanup, retraction, source-history no-change, control result absence, vulnerability fixed state, DNS absence, DHCP/IPAM lease expiry, flow non-observation, cloud-resource disappearance, or CDC tombstone evidence, `DeriveFacts` must validate the selected `020.SourceDatasetCatalogRow` ref/checksum, call `060.DeriveAbsenceOrUnknown`, and include the resulting `AbsenceDerivationResult` ref before emitting any absence, retraction, cleanup-derived correction, graph handoff, or watermark-affecting output.

### AbsenceSensitiveGoldCandidateMatrix

| Evidence class | `required_source_dataset_catalog_row_ref` | `required_source_closure_matrix_ref` | `required_absence_derivation_result_ref` | `required_effect` | `requested_effect_manifest_requirement` | `default_when_missing` | `mutation_allowed` | `validation_family` |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Missing expected row/object | Required catalog row or deterministic source-dataset block row | Exact `060.SourceAuthorityClosureMatrixRow` or deterministic block row | Required | `absence` | selected `020` row, selected `060` row, `AbsenceDerivationResult`, and consulted policy refs in `VersionManifest` | owner `060` blocking reason; emit no fact | No | `120-SOURCE-CLOSURE-*`; `120-SOURCE-DATASET-CATALOG-*`; `120-TEMPORAL-CORRECTION-*` |
| Source-declared delete | Required catalog row or deterministic source-dataset block row | Exact closure row for selected predicate/effect | Required | `retraction` or `cleanup` as requested by derivation row | selected `020` row, selected `060` row, `AbsenceDerivationResult`, and correction policy refs in `VersionManifest` | no correction | No | `120-SOURCE-CLOSURE-*`; `120-SOURCE-DATASET-CATALOG-*`; `120-TEMPORAL-CORRECTION-*` |
| Cleanup candidate | Required catalog row or deterministic source-dataset block row | Exact closure row for `cleanup` | Required | `cleanup` | selected `020` row, closure row, `AbsenceDerivationResult`, cleanup policy, and mutation-prohibition refs in `VersionManifest` | no cleanup-derived correction | No | `120-SOURCE-CLOSURE-*`; `120-SOURCE-DATASET-CATALOG-*`; `120-TEMPORAL-CORRECTION-*` |
| Graph expiry candidate | Required catalog row or deterministic source-dataset block row | Exact closure row for `graph_expiry` | Required | `graph_expiry` | selected `020` row, closure row, `AbsenceDerivationResult`, graph handoff refs, and `090` gate refs in `VersionManifest` | handoff effect `none` | No | `120-GRAPH-HANDOFF-*`; `120-SOURCE-CLOSURE-*`; `120-SOURCE-DATASET-CATALOG-*` |
| Vulnerability fixed state | Required catalog row or deterministic source-dataset block row | Vulnerability closure row chain | Required | `absence` or `retraction` as predicate row permits | selected `020` row, closure row, coverage/staleness refs, and `AbsenceDerivationResult` in `VersionManifest` | unknown/no-op | No | `120-SOURCE-CLOSURE-*`; `120-SOURCE-DATASET-CATALOG-*` |
| Control result absence or not checked | Required catalog row or deterministic source-dataset block row | Control-result closure row chain | Required | `absence` or control-state output | selected `020` row, closure row, `ControlResultMappingRow`, and `AbsenceDerivationResult` refs in `VersionManifest` | no pass/fail default | No | `120-SOURCE-CLOSURE-*`; `120-SOURCE-DATASET-CATALOG-*`; `120-ERROR-REGISTRY-*` |
| DNS TTL expiry | Required catalog row or deterministic source-dataset block row | DNS closure row chain | Required when predicate effect is requested | requested predicate effect only | selected `020` row, closure row, staleness refs, and `AbsenceDerivationResult` refs in `VersionManifest` | stale/unknown, not deletion | No | `120-SOURCE-CLOSURE-*`; `120-SOURCE-DATASET-CATALOG-*` |
| DHCP/IPAM lease expiry | Required catalog row or deterministic source-dataset block row | DHCP/IPAM closure row chain | Required when predicate effect is requested | requested predicate effect only | selected `020` row, closure row, lease/staleness refs, and `AbsenceDerivationResult` refs in `VersionManifest` | expired assignment or unknown, not host absence | No | `120-SOURCE-CLOSURE-*`; `120-SOURCE-DATASET-CATALOG-*` |
| Flow non-observation | Required catalog row or deterministic source-dataset block row | Flow closure row chain only if a future active row permits it | Required when future row permits | blocked by default | selected `020` row, closure row, and mutation-prohibition refs in `VersionManifest` | unknown/no edge | No | `120-SOURCE-CLOSURE-*`; `120-SOURCE-DATASET-CATALOG-*`; `120-GRAPH-PROFILE-CLOSURE-*` |
| Cloud disappearance | Required catalog row or deterministic source-dataset block row | Cloud inventory/source-history closure row chain | Required | `absence`, `cleanup`, or `graph_expiry` as row permits | selected `020` row, closure row, source-history refs, and `AbsenceDerivationResult` refs in `VersionManifest` | unknown/no cleanup | No | `120-SOURCE-CLOSURE-*`; `120-SOURCE-DATASET-CATALOG-*`; `120-GRAPH-HANDOFF-*` |
| Source-history no-change | Required catalog row or deterministic source-dataset block row | Source-history retention and coverage closure row chain | Required | no-change proof | selected `020` row, closure row, source-history retention refs, coverage refs, and `AbsenceDerivationResult` refs in `VersionManifest` | unknown/no proof | No | `120-SOURCE-CLOSURE-*`; `120-SOURCE-DATASET-CATALOG-*` |
| CDC tombstone | Required catalog row or deterministic source-dataset block row | CDC replay state plus exact source closure row | Required | `retraction` or `cleanup` only when row permits | selected `020` row, CDC replay state refs, closure row, and `AbsenceDerivationResult` refs in `VersionManifest` | no retraction/cleanup | No | `120-TEMPORAL-CORRECTION-*`; `120-SOURCE-CLOSURE-*`; `120-SOURCE-DATASET-CATALOG-*` |

Every row in this matrix must validate the selected `020.SourceDatasetCatalogRow` or deterministic source-dataset block row and route through `060.DeriveAbsenceOrUnknown` before fact creation, correction, graph handoff, or watermark-affecting output. Missing, blocked, ambiguous, inactive, checksum-mismatched, unvalidated, or unmanifested source-dataset refs select the default behavior and must not be reinterpreted by `080`.

#### MVPSourceDatasetGoldBlockCompanionMatrix

| Blocked candidate | Required source-dataset row or block ref | Required source-authority closure | Default behavior | Mutation allowed | Validation family |
| --- | --- | --- | --- | --- | --- |
| `directory_inventory_gold_output_blocked` | Selected `020.directory_inventory` catalog row or `DIRECTORY_INVENTORY_GOLD_OUTPUT_BLOCKED` deterministic block row. | No gold closure unless exact future owner rows exist. | Emit explicit no-op or owner error; no `GoldFact`, `GoldFactChangeSet`, graph handoff, absence outcome, cleanup, graph expiry, watermark, membership, nonmembership, or deletion. | No | `120-GOLD-PREDICATE-CATALOG-*`; `120-SOURCE-DATASET-CATALOG-*`; `120-IDENTITY-CLOSURE-*`. |
| `source_history_no_change_proof_blocked` | Selected `020.source_history` catalog row or `SOURCE_HISTORY_NO_CHANGE_PROOF_BLOCKED` deterministic block row. | Exact retention, coverage, staleness, completeness, authority, absence, validation, package-set, and manifest refs are required to unblock. | Emit unknown/no-op; no no-change proof, disappearance proof, negative output, cleanup, retraction, graph expiry, watermark, or compliance negative output. | No | `120-SOURCE-CLOSURE-*`; `120-SOURCE-DATASET-CATALOG-*`; `120-GOLD-PREDICATE-CATALOG-*`. |
| `network_flow_non_observation_blocked` | Selected `020.network_flow` catalog row or `FLOW_ABSENCE_BLOCKED_MVP` deterministic block row. | Future exact flow absence closure only; MVP default is blocked. | Emit unknown/no-op; no absence edge, graph expiry, cleanup, retraction, watermark, or theoretical reachability. | No | `120-SOURCE-CLOSURE-*`; `120-GRAPH-PROFILE-CLOSURE-*`; `120-OCSF-DIRECTION-*`. |
| `future_reachability_blocked` | Exact `020.future_reachability` deterministic block row. | None while `200` remains inactive deferred. | Emit no production output and no production reachability validation pass. | No | reachability-prohibition rows; `120-SOURCE-DATASET-CATALOG-*`. |

### DerivationRuleBundle handoff from 130

`130` derivation artifacts may supply derivation rules and candidates only. `080` remains the sole gold derivation path. A package, analysis rule, registry artifact, enrichment profile, or derived-edge rule must not persist `GoldFact` bytes directly.

| Input from `130` | Required `080` handling |
| --- | --- |
| `DerivationRuleBundle` active ref | May be consumed only as a derivation profile or rule artifact after activation ref validation. |
| Candidate gold output | Must enter `DeriveFacts`; raw `GoldFact` bytes emitted by a package or analysis rule are rejected before persistence. |
| Derived graph edge requesting `gold_fact_via_080` | Must satisfy temporal resolution, source authority, completeness, correction policy, replay policy, schema validation, and validation refs before any fact output. |
| Missing authority/completeness/temporal refs | Deterministic owner error or no-op before fact ID computation. |
| Replay | Use `ReplayInputSufficiencyCheck` and `ComputeReplayEquivalenceChecksum` before any replay output. |

`080` emits graph handoff metadata only and never mutates graph state.

Correction candidates whose mutation depends on absence, cleanup, retraction, source delete, fixed state, no-change, TTL expiry, lease expiry, missing flow, cloud disappearance, or CDC tombstone must reject before `GoldFactChangeSet` creation when the source-dataset catalog row ref is missing, blocked, ambiguous, inactive, checksum-mismatched, package-set-mismatched, unvalidated, or omitted from `030.VersionManifest`.

### GoldFactPredicateContractRow

`GoldFactPredicateContractRow` is the row-level semantic contract for one exact `GoldFact.fact_type` and `GoldFact.predicate`. It imports the closed `040` one-of kind registries and owns only predicate-specific permission, null-object permission, structured object schema refs, correction behavior refs, source authority refs, temporal refs, and replay inclusion.

| Field | Required | Default or omission behavior | Rule |
| --- | ---: | --- | --- |
| `row_id` | Yes | none | Stable row ID scoped to the active predicate-contract row set. |
| `fact_type` | Yes | none | Exact token. Wildcards are forbidden. |
| `predicate` | Yes | none | Exact token. Wildcards are forbidden. |
| `allowed_subject_ref_kinds` | Yes | none | Non-empty subset of `040.GoldFactSubjectRefKindRegistry`. |
| `allowed_object_value_kinds` | Yes | none | Non-empty subset of `040.GoldFactObjectValueKindRegistry`. |
| `object_kind_required_value` | Yes | none | Must equal `GoldFact.object_kind`; `GoldFact.object_kind` must equal `GoldFact.object_value.kind`. |
| `null_object_policy` | No | `forbidden` | Closed enum: `forbidden`, `allowed`. `allowed` permits only `object_value.kind = null_value`. |
| `structured_value_schema_refs` | Required when `structured_value` is allowed | `[]` when `structured_value` is not allowed | Non-empty active schema refs when `allowed_object_value_kinds` contains `structured_value`. |
| `identity_like_string_policy` | No | `reject` | Closed enum: `reject`, `allow_bounded_non_identity_string`. Default rejects IP, hostname, DNS, PTR, provider key, mapped target, graph key, and source-native identity strings. |
| `required_source_authority_refs` | Yes | none | Exact `060.SourceAuthorityProfileRow` refs or row-set refs required before output. |
| `required_temporal_policy_refs` | Yes | none | Exact temporal policy refs required before fact-time resolution. |
| `required_correction_policy_refs` | Yes | none | Exact correction policy refs required for correction-producing facts. |
| `required_replay_policy_refs` | Yes | none | Exact replay policy row refs included in replay equivalence. |
| `validation_refs` | Yes | none | Non-empty refs to `120` predicate-contract positive, negative, null, structured, identity-like-string, and replay rows. |
| `activation_scope` | Yes | none | Scope in which the row may affect output. |
| `lifecycle_status` | Yes | none | Production use requires `active`. |

`DeriveFacts` must resolve exactly one active `GoldFactPredicateContractRow` for the candidate `fact_type` and `predicate` before computing `gold_fact_key_id`. Zero matching rows fail with `GOLD_FACT_PREDICATE_CONTRACT_MISSING`. More than one equally specific row fails with `GOLD_FACT_PREDICATE_CONTRACT_AMBIGUOUS`. Disallowed subject kinds, disallowed object kinds, forbidden null objects, missing structured schema refs, and rejected identity-like strings fail before ID computation. If the one-of shape itself is invalid, `040.CORE_ONE_OF_INVALID` owns the failure.

`structured_value_schema_refs` must not be used as an unbounded object escape hatch. A structured object value may be emitted only when the selected predicate row names the exact schema ref and the schema ref is active, checksum-valid, and included in `VersionManifest`.

When a candidate `structured_value` is derived from `CadastreSilverObservation.normalized_fields` governed by OCSF, the selected `GoldFactPredicateContractRow` must name the exact structured schema ref, the producing `050.ProfileResolutionManifest` ref, and the producing external schema profile ref. When authority is required, an exact `060.SourceAuthorityProfileRow` and, when applicable, exact `060.ExternalSchemaAuthoritySignalMappingRow` must also validate. OCSF object presence, object absence, observables, enrichments, `unmapped`, status, severity, and confidence must not satisfy `structured_value_schema_refs` or object authority by themselves.

Predicate contract refs and predicate contract checksums are replay-affecting for `gold` and `gold_correction` output classes.

### MVPGoldFactPredicateContractRowSetClosure

`MVPGoldFactPredicateContractRowSetClosure` is the public MVP predicate-catalog closure. It instantiates `GoldFactPredicateContractRow` without redefining the row schema.

| Field | Required value or behavior |
| --- | --- |
| `row_set_id` | `gfp-mvp-gold-fact-predicate-contracts-v1`. |
| `row_set_lifecycle_status` | Production use requires `active`. |
| `row_set_checksum` | Required before production output. It is SHA-256 over the materialized row set after row defaults materialize and rows sort by ascending lexical `row_id`. |
| `row_checksums` | Required for every active row and deterministic block row. Concrete checksum bytes belong in activation-controlled material, not in this core spec. |
| `package_set_ref` | Required when any row, structured schema row, block row, or row-set envelope is package-supplied. |
| `validation_refs` | Must include `120-GOLD-PREDICATE-CATALOG-*`, `120-SOURCE-CLOSURE-*`, `120-SOURCE-DATASET-CATALOG-*`, `120-GRAPH-PROFILE-CLOSURE-*`, `120-VERSION-MANIFEST-*`, and `120-ERROR-REGISTRY-*` rows that cover the selected production scope. |
| `version_manifest_refs` | `030.VersionManifest.included_refs` must include the row-set ref/checksum, every selected row ref/checksum, every selected block row ref/checksum, every structured schema ref/checksum, every validation ref, and package-set refs when package-supplied. |

A production gold-derivation run in MVP must classify every candidate `fact_type` and `predicate` by exactly one row in the active-row inventory below or by exactly one deterministic block row in `GoldFactPredicateDeterministicBlockInventory`. Missing, ambiguous, inactive, checksum-mismatched, package-set-mismatched, unvalidated, unmanifested, or `TODO:`-bearing rows fail before `gold_fact_key_id` computation and emit the most specific `080` predicate-catalog error.

Common defaults for every active row in this section are:

| Default | Required behavior |
| --- | --- |
| `null_object_policy` | `forbidden`. |
| `identity_like_string_policy` | `reject`. |
| top-level `string_value` for identity-like object values | Forbidden. IP addresses, hostnames, DNS names, PTR names, provider keys, mapped targets, graph keys, source-native IDs, user names, group names, and account names must use permitted reference kinds or structured schemas. |
| `structured_value` | Permitted only when the selected row names the exact active schema ref and checksum in `MVPGoldFactStructuredValueSchemaCatalog`. |
| wildcard matching | Forbidden for fact type, predicate, subject kind, object kind, entity type, identifier type, and schema ref. |
| selected refs | Every selected row, schema, authority row, temporal row, correction row, replay row, validation row, and package-set ref must appear in `VersionManifest`. |

#### MVPGoldFactPredicateActiveRowInventory

| Required row ID | Fact type | Predicate | Subject rule | Object rule | Additional required behavior |
| --- | --- | --- | --- | --- | --- |
| `gfp-mvp-host-id-resolved-to-canonical-v1` | `host_identity_link` | `resolved_to_canonical` | `source_asset_ref` or `identifier_ref`; host-scoped only | `canonical_entity_ref`; entity type `host` | Resolver eligibility for the object canonical entity must pass `070`. |
| `gfp-mvp-host-id-has-identifier-v1` | `host_identity_link` | `has_identifier` | `canonical_entity_ref`; entity type `host` | `identifier_ref`; bounded identifier type | Identifier type must be one active bounded host identifier type under `070.IdentifierScope`. |
| `gfp-mvp-host-attr-has-os-v1` | `host_attribute_fact` | `has_os` | `canonical_entity_ref`; entity type `host` | `structured_value`; schema `080.struct.os_descriptor.v1` | Structured schema ref and checksum are replay-affecting. |
| `gfp-mvp-host-attr-lifecycle-v1` | `host_attribute_fact` | `has_lifecycle_state` | `canonical_entity_ref`; entity type `host` | `structured_value`; schema `080.struct.host_lifecycle_state.v1` | Structured schema ref and checksum are replay-affecting. |
| `gfp-mvp-host-attr-management-v1` | `host_attribute_fact` | `has_management_state` | `canonical_entity_ref`; entity type `host` | `structured_value`; schema `080.struct.host_management_state.v1` | Structured schema ref and checksum are replay-affecting. |
| `gfp-mvp-host-ip-had-ip-v1` | `host_ip_assignment_fact` | `had_ip` | `canonical_entity_ref`; entity type `host` | `identifier_ref`; identifier type `ip_address` | IP textual strings are forbidden as object `string_value`. |
| `gfp-mvp-host-dns-has-dns-name-v1` | `host_dns_name_fact` | `has_dns_name` | `canonical_entity_ref`; entity type `host` | `identifier_ref`; identifier type `dns_name` | DNS textual strings are forbidden as object `string_value`. |
| `gfp-mvp-host-dns-resolved-to-ip-v1` | `host_dns_name_fact` | `resolved_to_ip` | `identifier_ref`; identifier type `dns_name` | `identifier_ref`; identifier type `ip_address` | Subject and object identifier scopes must be valid under `070`. |
| `gfp-mvp-host-vuln-has-vulnerability-v1` | `host_vulnerability_fact` | `has_vulnerability` | `canonical_entity_ref`; entity type `host` | `structured_value`; schema `080.struct.vulnerability_instance_key.v1` | Coverage-sensitive; requires exact `060` coverage and authority rows before positive or negative output. |
| `gfp-mvp-host-software-runs-software-v1` | `host_software_fact` | `runs_software` | `canonical_entity_ref`; entity type `host` | `structured_value`; schema `080.struct.software_instance_key.v1` | Software identity strings must remain inside the structured schema. |
| `gfp-mvp-control-failed-v1` | `host_control_state_fact` | `failed_control` | `canonical_entity_ref`; entity type `host` | `structured_value`; schema `080.struct.control_evaluation_key.v1` | Requires exact `060.ControlResultMappingRow` before output. |
| `gfp-mvp-control-passed-v1` | `host_control_state_fact` | `passed_control` | `canonical_entity_ref`; entity type `host` | `structured_value`; schema `080.struct.control_evaluation_key.v1` | Requires exact `060.ControlResultMappingRow` before output. |
| `gfp-mvp-control-unknown-v1` | `host_control_state_fact` | `control_unknown` | `canonical_entity_ref`; entity type `host` | `structured_value`; schema `080.struct.control_evaluation_key.v1` | Requires exact `060.ControlResultMappingRow` before output. |
| `gfp-mvp-identity-member-of-v1` | `identity_membership_fact` | `member_of` | `canonical_entity_ref`; entity type `user`, `service_account`, or `group` | `canonical_entity_ref`; entity type `group` | Both member and group refs require qualifying `070.IdentityDecision` output. |
| `gfp-mvp-user-logged-on-to-v1` | `user_host_activity_fact` | `logged_on_to` | `canonical_entity_ref`; entity type `user` or `service_account` | `canonical_entity_ref`; entity type `host` | User/service-account and host refs require qualifying identity decisions. |
| `gfp-mvp-user-used-device-v1` | `user_host_activity_fact` | `used_device` | `canonical_entity_ref`; entity type `user` or `service_account` | `canonical_entity_ref`; entity type `host` | User/service-account and host refs require qualifying identity decisions. |
| `gfp-mvp-flow-observed-connection-v1` | `observed_network_flow_fact` | `observed_connection` | `canonical_entity_ref`; entity type `host` | `canonical_entity_ref`; entity type `host` | Graph projection may consume only this row for MVP `observed_connection`. Direction still requires `050.FlowRoleEvidence` and `090` graph rules. |
| `gfp-mvp-exposure-observed-v1` | `exposure_fact` | `has_observed_exposure` | `canonical_entity_ref`; entity type `host` | `structured_value`; schema `080.struct.observed_exposure_key.v1` | Observed exposure only; must not imply theoretical reachability or service access. |

The active row inventory above is total for MVP positive predicate selection. A new positive fact type or predicate must not be emitted by adding an implementation-local row. It requires an owner-spec change, package policy coverage when package-supplied, validation rows, manifest inclusion, and promotion through `000.MVPActivationCatalogClosurePack`.

#### GoldFactPredicateDeterministicBlockInventory

| Block row ID | Blocked predicate family | Required no-output behavior |
| --- | --- | --- |
| `gfp-block-modeled-reachability-v1` | `modeled_reachability_fact` and any modeled reachability predicate | Emit no fact, no correction, no graph edge, no graph property, no watermark, and no API reachability claim. |
| `gfp-block-exposure-theoretical-reachability-v1` | `exposure_fact` / `has_theoretical_reachability` | Emit no fact, no correction, no graph edge, no graph property, no watermark, and no API reachability claim. |
| `gfp-block-flow-theoretical-reachability-v1` | `observed_network_flow_fact` / `has_theoretical_reachability` | Emit no fact, no correction, no graph edge, no graph property, no watermark, and no API reachability claim. |
| `gfp-block-host-ownership-v1` | `host_ownership_fact` / `owned_by` and owner-like host predicates | Emit no fact and no ownership label until an exact ownership authority contract is specified. |
| `gfp-block-directory-inventory-gold-output-v1` | Any gold predicate output attempted from `directory_inventory` alone | Emit no fact, no correction, no membership/nonmembership fact, no deletion, no graph edge, no cleanup, no graph expiry, no watermark, and no authorized-negative API output. |
| `gfp-block-source-history-no-change-v1` | Source-history no-change, disappearance, or outside-window negative proof without exact source-history closure rows | Emit no fact, no correction, no no-change proof, no cleanup, no graph expiry, no watermark, and no compliance negative output. |

A deterministic block row is selected only for its exact blocked family. It must not act as a wildcard row for unrelated predicates. Selection of a block row must produce explicit no-output evidence and mutation-prohibition evidence, and that evidence must be manifest-included before validation acceptance can pass.

### MVPGoldFactStructuredValueSchemaCatalog

`MVPGoldFactStructuredValueSchemaCatalog` is the MVP structured object schema catalog referenced by active predicate rows. It is an activation-controlled schema catalog. This section defines schema identity and required closure status only; concrete field tables and row checksums must be supplied by activation-controlled material before promotion.

| Schema ref | Used by predicate rows | Required closure status |
| --- | --- | --- |
| `080.struct.os_descriptor.v1` | `gfp-mvp-host-attr-has-os-v1` | TODO: add complete field table, field bounds, enum behavior, unknown-field policy, checksum inputs, and validation fixture refs. |
| `080.struct.host_lifecycle_state.v1` | `gfp-mvp-host-attr-lifecycle-v1` | TODO: add complete field table, lifecycle-state enum behavior, source-state mapping, checksum inputs, and validation fixture refs. |
| `080.struct.host_management_state.v1` | `gfp-mvp-host-attr-management-v1` | TODO: add complete field table, management-state enum behavior, source-state mapping, checksum inputs, and validation fixture refs. |
| `080.struct.vulnerability_instance_key.v1` | `gfp-mvp-host-vuln-has-vulnerability-v1` | TODO: add complete field table for scanner-neutral vulnerability instance identity, coverage refs, checksum inputs, and validation fixture refs. |
| `080.struct.software_instance_key.v1` | `gfp-mvp-host-software-runs-software-v1` | TODO: add complete field table for software identity, version/package fields, checksum inputs, and validation fixture refs. |
| `080.struct.control_evaluation_key.v1` | `gfp-mvp-control-failed-v1`, `gfp-mvp-control-passed-v1`, `gfp-mvp-control-unknown-v1` | TODO: add complete field table for control identity, target scope, evaluation source, external control-result mapping refs, checksum inputs, and validation fixture refs. |
| `080.struct.observed_exposure_key.v1` | `gfp-mvp-exposure-observed-v1` | TODO: add complete field table for observed-exposure identity, observation scope, non-reachability wording, checksum inputs, and validation fixture refs. |

`DeriveFacts` must reject `structured_value` before ID computation when the selected schema ref is absent, inactive, checksum-mismatched, package-set-mismatched, unmanifested, or `TODO:`-bearing. The required error is `GOLD_FACT_STRUCTURED_SCHEMA_MISSING` when no active schema ref exists and `GOLD_FACT_STRUCTURED_SCHEMA_CHECKSUM_MISMATCH` when the selected schema checksum differs from the active manifest entry.

### GoldFactPredicateBlockRowSemantics

A selected deterministic predicate block row must behave as follows.

| Output class | Required behavior when selected block row applies |
| --- | --- |
| `GoldFact` | Emit no fact. |
| `GoldFactChangeSet` | Emit no correction except an explicit no-op diagnostic when the validation row requires one. |
| Graph handoff | Emit `graph_handoff_effect = none`; no graph delta, graph property, graph edge, traversal class, or derived-view advancement. |
| Source authority or absence | Do not call absence authority as a substitute for the block row; no authorized-negative output is produced. |
| Watermark | Do not advance source, projection, graph, or presence-only watermarks from the blocked candidate. |
| API, export, health, audit | API-visible output may expose only the owner diagnostic permitted by `110`; it must not expose an authorized negative fact, reachability claim, ownership label, or graph mutation. |
| Manifest | Include block row ref/checksum, row-set ref/checksum, validation refs, mutation-prohibition proof, and package-set refs when package-supplied. |

Block-row selection has higher precedence than source authority selection for the blocked predicate family. A blocked predicate must not be reinterpreted as missing authority, unknown, absent, stale, or unsupported unless the selected block row explicitly routes to that owner diagnostic without mutation.

## Correction Contract

Gold corrections are append-only knowledge transitions. `ApplyGoldCorrection` must emit `GoldFactChangeSet` operations and must not mutate original `GoldFact` bytes. Effective `known_to` closure must be materialized from `GoldFactChangeSet` at read or replay time; base `GoldFact.known_to` remains the value written in the immutable record.

`ApplyGoldCorrection` must validate, in order, core schema, temporal resolution, source authority, absence derivation when the candidate is negative or cleanup-derived, correction policy, assertion-state transition row, snapshot-ref policy, replay input sufficiency when replaying, and graph handoff metadata. Schema validation fails before temporal resolution is consumed. Temporal resolution fails before gold ID computation. Source authority fails before fact creation. Missing snapshot refs fail before correction output. Graph handoff is emitted as metadata only and never mutates graph state.

When the graph handoff effect is `identity_split_handoff`, `ApplyGoldCorrection` must validate the `070.GraphCorrectionHandoff` ref, `split_identity_decision_ref`, `split_policy_ref`, `retained_canonical_entity_ref`, `new_canonical_entity_refs`, `split_partition_refs`, `affected_fact_refs` or `affected_fact_selection_checksum`, `resolver_explanation_checksum`, and `version_manifest_ref`. Missing or checksum-mismatched split handoff metadata must emit `REPLAY_INPUT_INSUFFICIENT` or the more specific owner code, must emit no correction output, and must emit no graph handoff effect other than `none`. `080` must not recompute resolver split partitions.

`fact_retraction`, `interval_split`, and cleanup-derived corrections require `AbsenceDerivationResult.absence_authorized = true` and `allowed_effects` containing the requested correction effect. Missing or unsafe source-authority closure produces no correction and must emit the most specific `060` blocking reason.

A correction consuming negative or cleanup-derived evidence must record the selected `060.SourceAuthorityClosureMatrixRow` ref or deterministic block row ref in the producing `VersionManifest`. A missing closure ref, missing `AbsenceDerivationResult`, or deterministic block row must produce no correction mutation, no graph handoff mutation, and no watermark-affecting output.

Identity split handoff failure precedence is:

| Failure | Required behavior |
| --- | --- |
| missing handoff ref | Emit `REPLAY_INPUT_INSUFFICIENT` or the more specific `070` owner error; emit no correction output. |
| checksum mismatch | Emit `REPLAY_INPUT_INSUFFICIENT` or the more specific owner error; emit no correction output. |
| split partition omitted | Emit no graph handoff effect except `none`. |
| resolver explanation checksum omitted | Emit no correction output and no projection handoff. |

## Assertion-State Transition Matrix

Every correction policy must define a total transition matrix across default assertion states and correction events. This spec defines the default matrix in `### Assertion-state transition matrix`. An omitted state/event pair is invalid and must emit `GOLD_CORRECTION_TRANSITION_UNDEFINED` with no `GoldFact`, no graph handoff effect except `none`, and no watermark advancement.

`unsafe_negative` evidence never emits absence, retraction, cleanup, graph expiry, or watermark advancement. It must produce no mutation and must preserve evidence only through diagnostic, quarantine, or `unknown` output permitted by `060`.

## Late Arrival

Late evidence must be routed through `EvaluateLateArrival` using an active late-arrival policy artifact, temporal resolution, source authority, source completeness, coverage, staleness, absence authorization when applicable, and watermark state. Silently discarding late authoritative evidence in production is forbidden.

Late authoritative evidence after the cutoff must be accepted as a correction candidate by default. Late non-authoritative evidence after the cutoff must be quarantined or preserved as evidence only by default. No late-arrival route may advance a source, projection, graph, or presence-only watermark unless `060.ProjectionWatermarkPolicy` permits the exact effect and a `WatermarkCommitRecord` is emitted.

## Replay Contract

Production replay must validate required replay policy artifact refs, then run `ReplayInputSufficiencyCheck`, resolve the active `ReplayEquivalencePolicy` output-class row, apply global replay failure precedence, decide replay mode, and run `ComputeReplayEquivalenceChecksum` before any replay output is written.

`ReplayEquivalencePolicy` defines included fields, excluded volatile fields, hash algorithm, canonical ordering, owner-specific included/excluded field handoff, failure precedence, and shadow-output behavior by output class. Output-specific field selection for `graph_delta`, `graph_apply`, and `graph_rebuild` is imported from `090`; field selection for `api_response` is imported from `110`; field selection for `export_projection` is imported from `050`; field selection for `analysis_output` is imported from `130`. `080` owns checksum algorithm, preflight ordering, and failure precedence.

### LakehouseTableStateReplayInputHandoff

Replay and correction preflight must treat lakehouse table-state refs, retention decisions, and cross-table consistency as first-class replay inputs. The table below is additive to the owner output-class rows in `ReplayEquivalencePolicy` and `030.VersionManifestCompletenessMatrix`.

| Output class | Additional required lakehouse inputs |
| --- | --- |
| `raw` | feed profile, read policy, raw supplier profile, raw manifest, object/partition/table refs, schema refs, and read checkpoint refs. |
| `silver` | raw refs plus read policy, manifest, schema refs, and active external schema refs. |
| `gold` | temporal refs, authority refs, absence refs, source-dataset refs, and table snapshot refs when the candidate depends on table state. |
| `gold_correction` | old `LakehouseSnapshotRef`, new `LakehouseSnapshotRef`, old/new `LakehouseCommitRef` when write-affecting, table-set checksum, `ReplayRetentionPolicy`, retention-protection proof, and schema compatibility refs. |
| `graph_rebuild` | graph rebuild manifest, table snapshot refs, dataset version refs, table-set checksum, retention protection, and schema compatibility refs. |
| `validation_acceptance` | all owner row refs, expected output checksums, fixture checksums, and package-set refs where applicable. |

Lakehouse replay failure precedence is:

```text
missing table-state ref
mutable-only ref
retention-ineligible
schema-incompatible
cross-table checksum mismatch
authority mismatch
temporal mismatch
side-effect mismatch
generic checksum mismatch
```

A more specific lakehouse failure must be emitted before `REPLAY_INPUT_INSUFFICIENT` or generic checksum mismatch.

### TelemetryReplayExclusionHandoff

Telemetry runtime identifiers and exporter state are excluded volatile fields by default. `ComputeReplayEquivalenceChecksum` must exclude trace IDs, span IDs, sampled flags, runtime duration, runtime start time, exporter queue state, exporter delivery result, Collector state, dropped telemetry counts, diagnostic display order, and telemetry backend-generated IDs unless an active `140.TelemetryReplayExclusionPolicy` and owner output-class row explicitly include a bounded telemetry diagnostic field for health, API, audit, or validation output.

Telemetry exclusion must not hide output-affecting telemetry policy changes. When telemetry policy affects health, API diagnostics, audit diagnostics, validation diagnostics, or operator-visible telemetry runtime state, `ReplayInputSufficiencyCheck` must require the relevant `140` policy refs and runtime state refs through `030.VersionManifest`.

A replay that includes trace IDs, span IDs, backend telemetry IDs, exporter queue state, or sampling decisions in an authoritative domain output checksum must fail with `TELEMETRY_REPLAY_FIELD_FORBIDDEN`.

### Lifecycle evidence in replay equivalence

Replay must not silently substitute lifecycle decisions. Lifecycle evidence is part of replay equivalence when it gates output, activation, graph apply, watermark eligibility, package selection, or validation acceptance.

| Replay output class | Lifecycle refs included in replay checksum | Excluded fields |
| --- | --- | --- |
| production run output | machine ID, machine version, machine checksum, transition evidence refs, selected transition row IDs, transition kinds | transition `created_at` for ID; mutable audit-only fields |
| package activation output | package lifecycle machine checksum, package-set activation evidence, failure/rollback/quarantine transition evidence refs | wall-clock display fields |
| graph apply output | graph apply machine checksum, graph apply transition refs, idempotency key, committed-batch evidence refs | backend physical IDs and runtime duration |
| validation acceptance output | validation machine checksum, validation report refs, acceptance transition refs | validation execution wall-clock duration |
| run_lock_governed_output | `RunLockLeasePolicy` checksum, `RunLockSet` checksum, lock operation evidence refs, recovery evidence refs, commit guard refs, fencing token set, idempotency key, lock lifecycle transition evidence refs | diagnostic display timestamp, runtime duration, non-output telemetry correlation fields |

## Deterministic Side Effects

Runtime randomness, wall-clock reads, generated IDs, unordered iteration, external calls, or backend-discovered values must not affect production output unless a declared `DeterministicSideEffectRecord` captures the value and replay behavior.

## Temporal and Replay Contract Details

### TemporalSemanticsPolicy catalog

`TemporalSemanticsPolicy` is the activation-controlled row interface for fact-time resolution. A production temporal row must cover exactly one tuple: `source_dataset`, `observation_type`, `fact_type`, `predicate`, `source_scope_selector`, and `temporal_mode`. Wildcards are forbidden. A missing row emits `TEMPORAL_POLICY_UNRESOLVED` before `TemporalObservationTimeResolution` or `GoldFact` output.

| Field | Required | Default or omission behavior | Rule |
| --- | ---: | --- | --- |
| `policy_row_id` | Yes | none | Stable row ID scoped to the active temporal row set. |
| `source_dataset` | Yes | none | Exact vendor-neutral dataset. Wildcards are forbidden. |
| `observation_type` | Yes | none | Exact `050`/`040` observation type. Wildcards are forbidden. |
| `fact_type` | Yes | none | Exact fact type. Wildcards are forbidden. |
| `predicate` | Yes | none | Exact predicate. Wildcards are forbidden. |
| `source_scope_selector` | Yes | none | `030.ScopeSelector`; broad category fallback is forbidden. |
| `temporal_mode` | Yes | none | Closed owner token selecting event-time, state-assertion-time, source-interval, or explicit policy mode. |
| `valid_interval_model` | Yes | none | One of `instant`, `half_open_interval`, `closed_source_interval_imported_as_half_open`, or `open_until_replaced`. |
| `valid_time_input_precedence` | Yes | none | Ordered source field paths and source metadata paths allowed to become valid time. Empty is invalid unless row emits deterministic block. |
| `valid_to_rule` | Yes | none | One of `source_end_time`, `duration_seconds`, `open_interval`, `close_on_replacement`, or deterministic error. |
| `time_quality_requirements` | Yes | none | Required source time quality, timezone, parser quality, and ambiguity rules. |
| `fallback_behavior` | Yes | `deterministic_error` | Closed enum: `deterministic_error`, `use_authorized_alternate_source_time`, `emit_unknown_valid_time`; current platform time is not a fallback. |
| `malformed_time_behavior` | Yes | `deterministic_error` | Must emit temporal error unless row permits `emit_unknown_valid_time`. |
| `ambiguous_time_behavior` | Yes | `deterministic_error` | Must emit temporal error unless row defines a deterministic disambiguation rule and validation refs. |
| `absent_time_behavior` | Yes | `deterministic_error` | Must not use current platform time. |
| `knowledge_time_policy_ref` | Yes | none | Active `KnowledgeTimeImportPolicy` row ref. |
| `late_arrival_policy_ref` | Yes | none | Active `LateArrivalPolicy` row ref for observations covered by this row. |
| `validation_refs` | Yes | none | Non-empty refs covering present, absent, malformed, ambiguous, unauthorized, fallback, replay, and manifest cases. |
| `lifecycle_status` | Yes | none | Production use requires `active`. |

Default temporal decision table:

| Candidate time input | Default fact-time eligibility | Required behavior |
| --- | --- | --- |
| authorized source interval | allowed as `source_interval` | Use only when row lists the interval fields and interval quality validates. |
| authorized source event time | allowed as `state_assertion` | Use only when row lists the event field and ambiguity rules pass. |
| supplier collection time | forbidden | May become fact time only when the row explicitly permits the exact field and validation refs prove the use. |
| supplier delivery time | forbidden | Same as supplier collection time. |
| table snapshot time | forbidden | May be table-state evidence only. |
| lakehouse commit time | forbidden | May be write-state evidence only. |
| CDC offset or log position | forbidden | May be replay state only. |
| CDC heartbeat time | forbidden | May be liveness diagnostic only. |
| schema-history time | forbidden | May be replay sufficiency input only. |
| graph apply time | forbidden | May be derived-view evidence only. |
| replay time | forbidden | May be replay execution evidence only. |
| current platform time | forbidden | May not become fact time or known time unless captured in persisted evidence before derivation and an active row permits the exact use. |

### KnowledgeTimeImportPolicy

`KnowledgeTimeImportPolicy` is the activation-controlled row interface for selecting `known_from`. Production default is `current_import`.

| Field | Required | Default or omission behavior | Rule |
| --- | ---: | --- | --- |
| `policy_row_id` | Yes | none | Stable row ID scoped to the active row set. |
| `mode` | Yes | `current_import` | One of `current_import`, `historical_import_valid_time_only`, `reconstructed_source_known_time`, or `rejected_reconstruction`. |
| `as_known_then_allowed` | No | `false` | `true` is valid only for `reconstructed_source_known_time`. |
| `source_known_time_evidence_refs` | Required when reconstruction allowed | `[]` | Persisted evidence refs, not live source calls. |
| `current_import_time_source` | Yes for `current_import` | persisted import evidence | Must be persisted before derivation and included in `VersionManifest`. |
| `malformed_known_time_behavior` | Yes | `rejected_reconstruction` | Deterministic error or rejected reconstruction. |
| `validation_refs` | Yes | none | Non-empty refs for current import, historical valid-time import, reconstruction success, and reconstruction rejection. |
| `lifecycle_status` | Yes | none | Production use requires `active`. |

| Mode | Required behavior |
| --- | --- |
| `current_import` | `known_from` is Cadastre import knowledge time from persisted evidence. |
| `historical_import_valid_time_only` | Historical valid time may be imported; historical known time is not reconstructed. |
| `reconstructed_source_known_time` | Allowed only when `as_known_then_allowed = true`, source-known-time evidence refs validate, and validation rows pass. |
| `rejected_reconstruction` | Default when source-known-time evidence is missing, stale, ambiguous, malformed, or untrusted. |

### TemporalObservationTimeResolution schema

`TemporalObservationTimeResolution` records the selected time interpretation for one observation before any `GoldFact` candidate exists.

| Field | Required | Rule |
| --- | ---: | --- |
| `temporal_resolution_id` | Yes | Deterministic ID over observation ref, policy refs, selected inputs, resolved interval, resolved known time, fallback flag, and resolution quality. |
| `raw_record_refs` | Yes | Canonically sorted raw refs that supplied time evidence. |
| `silver_observation_ref` | Yes | Exact observation ref. |
| `temporal_policy_ref` | Yes | Active `TemporalSemanticsPolicy` row ref and checksum. |
| `knowledge_time_policy_ref` | Yes | Active `KnowledgeTimeImportPolicy` row ref and checksum. |
| `selected_valid_time_input_path` | Yes | Canonical field path or deterministic error token. |
| `selected_valid_time_input_value` | Required when selected | Canonical value after parsing; raw source value remains in evidence only. |
| `resolved_valid_from` | Yes | RFC3339 UTC or null only when policy permits unknown valid start. |
| `resolved_valid_to` | Yes | RFC3339 UTC or null only when policy permits open interval. |
| `resolved_known_from` | Yes | RFC3339 UTC from selected knowledge-time policy. |
| `resolved_known_to` | Yes | Null for new candidate knowledge interval unless correction materialization supplies a closure. |
| `fallback_used` | Yes | Boolean. True only when policy permits a named fallback. |
| `resolution_quality` | Yes | Closed owner token: `source_time_exact`, `source_interval_exact`, `authorized_fallback`, `unknown_per_policy`, or `temporal_error`. |
| `error_code` | Yes | Null only when resolution succeeds; otherwise the most specific temporal error. |
| `resolution_checksum` | Yes | SHA-256 over canonical row bytes excluding only this field. |
| `version_manifest_ref` | Yes | Manifest containing policy refs, selected evidence refs, and checksums. |

### BitemporalQueryMode catalog

| Query mode | Default valid time | Default known time | Assertion-state visibility | Page-token scope | Ordering | Audit behavior |
| --- | --- | --- | --- | --- | --- | --- |
| `current` | current platform time supplied by query service | current platform time supplied by query service | active and policy-visible stale | query checksum plus auth context | owner-defined canonical sort | audit optional by API class |
| `valid_at` | supplied | current platform time supplied by query service | active/stale/conflicted per policy | includes valid time | canonical sort | audit required for auditor |
| `known_at` | current platform time supplied by query service | supplied | as-known facts only | includes known time | canonical sort | audit required |
| `corrected_history` | supplied or interval | supplied or current query time | includes superseded/retracted when requested | includes both times and assertion filter | canonical sort | audit required |

Query service current time is query evaluation context only. It must not be reused as `GoldFact.valid_time`, `GoldFact.known_time`, or correction evidence time.

### GoldFactCorrectionPolicy

`GoldFactCorrectionPolicy` is the activation-controlled row interface for comparable fact selection and correction behavior.

| Field | Required | Default or omission behavior | Rule |
| --- | ---: | --- | --- |
| `policy_row_id` | Yes | none | Stable row ID scoped to the active correction policy row set. |
| `fact_type` | Yes | none | Exact fact type; wildcards are forbidden. |
| `predicate` | Yes | none | Exact predicate; wildcards are forbidden. |
| `comparable_key_fields` | Yes | `gold_fact_key_id` plus declared semantic fields | Must define prior candidate comparability without inspecting graph state. |
| `prior_fact_selection` | Yes | none | Sort by valid interval overlap, `known_from`, authority priority, evidence checksum, then lexical `gold_fact_id` unless row narrows the order. |
| `overlap_behavior` | Yes | none | One of `replace_overlapping_known_state`, `split_valid_interval`, `conflict_on_overlap`, `no_op_duplicate`. |
| `interval_splitting_behavior` | Yes | `error_blocked` | Required when valid intervals partially overlap. |
| `confidence_only_change_behavior` | Yes | `no_op_duplicate` when canonical confidence bytes match; `insert_fact` when changed and permitted | Confidence comparison uses `040.DecimalPrecisionPolicy.confidence_0_1`. |
| `delete_evidence_behavior` | Yes | `error_blocked` | Requires `060.AbsenceDerivationResult` permitting `retraction`. |
| `cleanup_evidence_behavior` | Yes | `error_blocked` | Requires `060.AbsenceDerivationResult` permitting `cleanup`. |
| `stale_behavior` | Yes | `mark_stale` only when `060.SourceStalenessPolicy` permits stale state materialization | Otherwise `error_blocked`. |
| `conflict_behavior` | Yes | `mark_conflicted` | Resolution requires explicit conflict-resolution evidence. |
| `duplicate_behavior` | Yes | `no_op_duplicate` | Duplicate candidates emit a `GoldFactChangeSet`; they do not create active facts. |
| `sort_keys` | Yes | none | Total deterministic sort for prior facts and operation ordering. |
| `snapshot_policy_ref` | Yes | none | Active `CorrectionSnapshotRefPolicy` row ref. |
| `validation_refs` | Yes | none | Non-empty refs for replacement, retraction, split, conflict, stale, duplicate, confidence-only, and error cases. |
| `lifecycle_status` | Yes | none | Production use requires `active`. |

### GoldFactChangeSet operation vocabulary

| Operation | Required behavior | Graph handoff default |
| --- | --- | --- |
| `insert_fact` | Persist a new immutable `GoldFact` and record source refs. | `reproject_fact_key` |
| `close_known_interval` | Materialize effective prior `known_to` closure without mutating prior fact bytes. | `reproject_fact_key` |
| `split_valid_interval` | Emit new facts for split segments and close/supersede affected prior intervals. | `reproject_fact_key` |
| `mark_superseded` | Mark prior known state superseded through change-set materialization. | `reproject_fact_key` |
| `mark_retracted` | Emit authorized retraction state. | `expire_projected_object` when `060` authorizes `graph_expiry`; otherwise `none`. |
| `mark_stale` | Emit stale known state only when staleness policy permits. | `conflict_visibility_update` only when graph profile permits stale visibility; otherwise `none`. |
| `mark_conflicted` | Emit conflicted state with conflict evidence. | `conflict_visibility_update` |
| `resolve_conflict` | Emit resolved state and close conflicted known state. | `reproject_fact_key` |
| `no_op_duplicate` | Persist no-op change evidence and no active fact. | `none` |
| `error_blocked` | Persist deterministic error or diagnostic and no fact mutation. | `none` |

### CorrectionSnapshotRefPolicy

| Correction class | Old snapshot role | New snapshot role | Table-set checksum | Retention protection | Mutable-ref behavior | Status |
| --- | --- | --- | --- | --- | --- | --- |
| `fact_replacement` | required prior fact snapshot | required candidate fact snapshot | required | old and new snapshots protected | reject with `MUTABLE_BRANCH_REF_FOR_REPLAY` | active default |
| `fact_retraction` | required prior fact snapshot | required retraction evidence snapshot | required | old and new snapshots protected | reject with `MUTABLE_BRANCH_REF_FOR_REPLAY` | active default |
| `interval_split` | required prior interval snapshot | required split candidate snapshot | required | old and new snapshots protected | reject with `MUTABLE_BRANCH_REF_FOR_REPLAY` | active default |
| `confidence_only_change` | required prior confidence snapshot | required new evidence snapshot | required | old and new snapshots protected | reject with `MUTABLE_BRANCH_REF_FOR_REPLAY` | active default |
| `stale_state_transition` | required prior source-state snapshot | required staleness-evidence snapshot | required | old and new snapshots protected | reject with `MUTABLE_BRANCH_REF_FOR_REPLAY` | active default |
| `conflict_mark` | required prior fact snapshot | required conflict evidence snapshot | required | old and new snapshots protected | reject with `MUTABLE_BRANCH_REF_FOR_REPLAY` | active default |
| `conflict_resolution` | required conflict-state snapshot | required resolution evidence snapshot | required | old and new snapshots protected | reject with `MUTABLE_BRANCH_REF_FOR_REPLAY` | active default |
| `duplicate_no_op` | required comparable prior snapshot | required duplicate candidate snapshot | required | old and new snapshots protected | reject with `MUTABLE_BRANCH_REF_FOR_REPLAY` | active default |

Every correction output must include old and new `LakehouseSnapshotRef` refs, table-set checksum, retention-protection proof, and immutable dataset/version refs in `VersionManifest`. Mutable branch, tag, latest, or workspace-only refs are rejected before correction output.

`CorrectionSnapshotRefPolicy` row precision for lakehouse correction closure is:

| Field | Required | Default or omission behavior | Rule |
| --- | ---: | --- | --- |
| `policy_id` | Yes | none | Stable policy ID. |
| `policy_version` | Yes | none | Immutable version included in row checksum. |
| `correction_class` | Yes | none | One correction class from the table above. |
| `old_snapshot_roles` | Yes | none | Canonical set of required old snapshot roles. |
| `new_snapshot_roles` | Yes | none | Canonical set of required new snapshot roles. |
| `required_table_set_checksum` | No | `true` | Corrections affecting table state require old/new table-set checksums. |
| `old_commit_ref_required` | No | `true` when write-affecting | Missing required old commit ref rejects before output. |
| `new_commit_ref_required` | No | `true` when write-affecting | Missing required new commit ref rejects before output. |
| `retention_protection_required` | No | `true` | Missing retention-protection proof rejects before output. |
| `schema_compatibility_check_required` | No | `true` | Missing or failing compatibility check rejects before output. |
| `mutable_ref_behavior` | Yes | `reject_mutable_branch_tag_latest` | Mutable branch, tag, latest, or workspace-only refs are forbidden. |
| `validation_refs` | Yes | none | Non-empty refs covering missing, mutable, retention, schema, and table-set cases. |
| `activation_scope` | Yes | none | `030.ActivationScope`. |
| `lifecycle_status` | Yes | none | Production selection requires `active`. |
| `row_checksum` | Yes | none | SHA-256 over canonical row bytes after defaults materialize. |

Closed `mutable_ref_behavior` value for MVP is:

```text
reject_mutable_branch_tag_latest
```

### LateArrivalPolicy

Default route table:

| Evidence class | Arrival relative to cutoff | Authority result | Default route | Correction behavior | Quarantine behavior | Watermark effect |
| --- | --- | --- | --- | --- | --- | --- |
| positive authoritative | before or at cutoff | exact authority row passes | `normal_gold_derivation` | derive normally | none | none unless `060` permits |
| positive authoritative | after cutoff | exact authority row passes | `accept_as_correction` | evaluate through `ApplyGoldCorrection` | none unless correction gates fail | none unless `060` permits |
| negative authoritative | before or at cutoff | exact authority and absence gates pass | `normal_gold_derivation` | derive absence/correction through `060` and correction policy | none | none unless `060` permits |
| negative authoritative | after cutoff | exact authority and absence gates pass | `accept_as_correction` | evaluate retraction/cleanup only when requested effect gates pass | none unless correction gates fail | none unless `060` permits |
| non-authoritative positive | any | missing or non-authoritative row | `quarantine_or_evidence_only` | no correction | preserve evidence or diagnostic | none |
| non-authoritative negative | any | missing, unsafe, or weak negative | `quarantine_or_evidence_only` | no absence, no retraction, no cleanup | preserve evidence or diagnostic | none |
| unresolved temporal | any | temporal resolution missing or error | `quarantine_or_evidence_only` | no correction | preserve temporal error | none |

Defaults: `allowed_lateness_seconds = 0`; authoritative after-cutoff action is `accept_as_correction`; non-authoritative after-cutoff action is `quarantine_or_evidence_only`; production discard is forbidden; watermark effect is `none` unless `060` permits a `WatermarkCommitRecord` for the exact target and effect.

### EvaluateLateArrival Algorithm

```text
EvaluateLateArrival(observation, temporal_resolution, late_arrival_policy, authority_context, completeness_context, coverage_context, staleness_context, watermark_context):
1. Validate late-arrival policy ref through `030.ActivationControlledArtifactRef`.
2. Reject with `TEMPORAL_RESOLUTION_REQUIRED` when `temporal_resolution` is missing and the observation can affect gold, correction, replay, graph projection, or audit.
3. Validate source authority rows for the candidate fact type, predicate, dataset, scope, and requested effect.
4. For negative, stale, cleanup, delete, source-history no-change, or tombstone evidence, call `060.DeriveAbsenceOrUnknown` before route selection.
5. Validate completeness, coverage, staleness, progress-signal, source-history, and supplier-visibility refs required by `060`.
6. Compare resolved valid/known times with policy cutoff and `allowed_lateness_seconds`.
7. Select exactly one route from the default route table or an active row that covers the exact dataset/fact/predicate/scope.
8. If selected route would discard production evidence, emit `LATE_ARRIVAL_DISCARD_FORBIDDEN` and preserve evidence as quarantine or diagnostic output.
9. Emit late-arrival route state and include route, policy, temporal, authority, completeness, coverage, staleness, absence, and watermark refs in `VersionManifest`.
```

### ReplayEquivalencePolicy

| Output class | Included fields | Excluded volatile fields | Hash algorithm | Canonical ordering | Shadow-output rules | Status |
| --- | --- | --- | --- | --- | --- | --- |
| `raw` | feed profile refs, feed manifest refs, raw IDs, payload hashes, import profile, supplier metadata refs | run correlation ID, execution duration | `sha256` | canonical raw record ID | compare only; no production write | active default |
| `silver` | raw refs, parser profile, mapping bundle, external schema profile, normalized payload checksum, source-extension rule refs | diagnostic correlation ID, execution duration | `sha256` | observation ID | compare only | active default |
| `identity` | evidence refs, resolver profile, candidate profile, identity decision bytes, resolver explanation checksum | reviewer display labels, request correlation ID | `sha256` | decision ID | compare only | active default |
| `gold` | temporal resolution, authority refs, predicate contract refs and checksums, evidence refs, fact bytes, absence refs, correction policy refs, consulted `060` row refs | processing timestamp, run correlation ID | `sha256` | fact ID then known interval | compare only | active default |
| `gold_correction` | `GoldFactChangeSet`, correction policy, predicate contract refs and checksums, snapshot refs, table-set checksum, temporal refs, authority refs, transition row, graph handoff effect | execution timestamp, diagnostic display order | `sha256` | changeset ID then operation order | compare only | active default |
| `graph_delta` | imported `090` delta included fields, graph handoff refs, projection profile checksum, edge semantics row-set checksum, output eligibility row-set checksum, property mapping refs, graph correction handoff refs, resolver explanation checksum when split-driven, and graph delta checksum | backend transient IDs, runtime duration | `sha256` | delta ID | compare only | active default |
| `graph_apply` | imported `090` apply included fields, idempotency key, apply profile, backend taxonomy mapping profile, query translation profile, schema profile, backend evidence, index consistency refs, and derived-view state refs | backend physical IDs, runtime duration | `sha256` | apply result ID | compare only | active default |
| `graph_rebuild` | imported `090` rebuild included fields, rebuild manifest, projection profile checksum, edge semantics row-set checksum, output eligibility row-set checksum, backend taxonomy mapping profile, query translation profile, schema fingerprint, index consistency, derived-view state, and output checksum | backend import job ID, runtime duration | `sha256` | rebuild manifest ID | compare only | active default |
| `api_response` | imported `110` response included fields, authorization/redaction refs, page-token policy, output checksum | request correlation ID, display timestamp | `sha256` | owner-defined response ID | compare only | active default |
| `export_projection` | imported `050` projection included fields, projection profile, input refs, mapping refs, redaction refs, loss manifest checksum, output checksum | export job ID, request correlation ID, execution duration, display timestamp | `sha256` | export projection ID | compare only | active default |
| `analysis_output` | imported `130` analysis included fields, analysis rule bundle, graph derived-view refs, authorization/redaction refs, output checksum | request correlation ID, UI display label, runtime duration | `sha256` | analysis output ID | compare only | active default |
| `validation_acceptance` | validation matrix refs, fixture checksums, expected/actual checksums, validation lifecycle evidence, acceptance report bytes | validation execution duration, display timestamp | `sha256` | validation row ID then fixture ID | compare only | active default |
| `telemetry_health_diagnostic` | `140.TelemetryRuntimeState` refs when health/API/audit/validation output depends on telemetry state, telemetry policy refs, health mapping policy ref, redaction refs, output checksum | trace ID, span ID, sampled flag, exporter queue state unless health-visible, runtime duration, backend telemetry IDs, display timestamp | `sha256` | telemetry runtime state ref then policy ref | compare only | active default when `140` telemetry refs are active and validation rows pass |
| `run_lock_operation` | operation kind, idempotency key, input checksum, lock keys, prior and new lock record checksums, fencing token set, lock-store time at decision, result, error code, and version manifest ID | runner-local timestamp, diagnostic display timestamp, runtime duration, trace/span IDs | `sha256` | operation evidence ID then lock key | compare only | active default when `030` run-lock refs are active and validation rows pass |

Global replay failure precedence:

1. Missing replay policy row.
2. Missing activation-controlled artifact ref.
3. Inactive or checksum-mismatched artifact.
4. Mutable-only ref.
5. Retention-ineligible ref.
6. Schema or protocol incompatibility.
7. Authority, closure, or temporal policy mismatch.
8. Deterministic side-effect mismatch.
9. Output checksum mismatch.
10. Excluded volatile-field difference.

### DeterministicSideEffectPolicy

| Side-effect kind | Default |
| --- | --- |
| runtime randomness | rejected unless recorded |
| wall-clock read | rejected unless policy names clock field and records value before output |
| generated ID | rejected unless derived by `040.CoreRecordIdPolicy` |
| unordered iteration | rejected |
| external call | rejected in production unless explicitly validation-only |
| telemetry trace/span ID | excluded from domain replay; rejected if used as output-affecting deterministic ID |
| telemetry sampling decision | excluded from domain replay; may affect only telemetry emission |
| telemetry exporter result | operational health input only when represented by `TelemetryRuntimeState` and manifest refs |
| telemetry Collector state | operational health input only when represented by `TelemetryRuntimeState` and manifest refs |
| backend-discovered value | replay from recorded value only when owner permits |
| lock-store time read | allowed only inside `030.RunLockOperationEvidence`; runner-local wall-clock time remains rejected for lock stale decisions. |

### Assertion-state transition matrix

| Prior state | Event | Required output | Error/no-op | Graph handoff effect |
| --- | --- | --- | --- | --- |
| `active` | `duplicate` | `no_op_duplicate` change set | no new fact | `none` |
| `active` | `authoritative_replace` | close prior known interval and insert replacement | error if policy or snapshot refs missing | `reproject_fact_key` |
| `active` | `authorized_retraction` | mark retracted when `060` permits retraction | error if authority/absence gates fail | `expire_projected_object` only when `060` permits `graph_expiry`; otherwise `none` |
| `active` | `staleness_expiry` | mark stale only when staleness policy permits | error when stale materialization not permitted | `conflict_visibility_update` or `none` per graph profile |
| `active` | `authoritative_refresh` | insert refreshed active state or no-op duplicate | no-op when canonical bytes match | `reproject_fact_key` |
| `active` | `conflict_detected` | mark conflicted | error if conflict policy missing | `conflict_visibility_update` |
| `active` | `unsafe_negative` | no mutation | owner `060` blocking reason | `none` |
| `active` | `explicit_no_op` | `no_op_duplicate` change set | none | `none` |
| `stale` | `duplicate` | `no_op_duplicate` change set | no new fact | `none` |
| `stale` | `authoritative_replace` | insert active replacement and close stale known state | error if policy or snapshot refs missing | `reproject_fact_key` |
| `stale` | `authorized_retraction` | mark retracted when `060` permits retraction | error if authority/absence gates fail | `expire_projected_object` only when permitted |
| `stale` | `staleness_expiry` | `no_op_duplicate` change set | already stale | `none` |
| `stale` | `authoritative_refresh` | insert active refreshed fact | none when gates pass | `reproject_fact_key` |
| `stale` | `conflict_detected` | mark conflicted | error if conflict policy missing | `conflict_visibility_update` |
| `stale` | `unsafe_negative` | no mutation | owner `060` blocking reason | `none` |
| `stale` | `explicit_no_op` | `no_op_duplicate` change set | none | `none` |
| `retracted` | `duplicate` | `no_op_duplicate` change set | no new fact | `none` |
| `retracted` | `authoritative_replace` | insert active fact only when policy permits reactivation | error when reactivation forbidden | `reproject_fact_key` |
| `retracted` | `authorized_retraction` | `no_op_duplicate` change set | already retracted | `none` |
| `retracted` | `staleness_expiry` | no mutation | stale does not apply to retracted state | `none` |
| `retracted` | `authoritative_refresh` | insert active fact only when policy permits reactivation | error when reactivation forbidden | `reproject_fact_key` |
| `retracted` | `conflict_detected` | mark conflicted only when conflict policy permits retracted conflict visibility | otherwise no mutation | `conflict_visibility_update` or `none` |
| `retracted` | `unsafe_negative` | no mutation | owner `060` blocking reason | `none` |
| `retracted` | `explicit_no_op` | `no_op_duplicate` change set | none | `none` |
| `superseded` | `duplicate` | `no_op_duplicate` change set | no new fact | `none` |
| `superseded` | `authoritative_replace` | insert newer replacement when comparable-key policy permits | error when policy missing | `reproject_fact_key` |
| `superseded` | `authorized_retraction` | mark retracted only for current comparable known state | no-op when prior superseded state is not current | `none` or `expire_projected_object` when current projection is affected and permitted |
| `superseded` | `staleness_expiry` | no mutation | superseded state is not stale materialization target | `none` |
| `superseded` | `authoritative_refresh` | no-op unless policy permits refresh of superseded history | no-op by default | `none` |
| `superseded` | `conflict_detected` | mark conflicted only when current comparable state is affected | otherwise no-op | `conflict_visibility_update` or `none` |
| `superseded` | `unsafe_negative` | no mutation | owner `060` blocking reason | `none` |
| `superseded` | `explicit_no_op` | `no_op_duplicate` change set | none | `none` |
| `conflicted` | `duplicate` | `no_op_duplicate` change set | no new fact | `none` |
| `conflicted` | `authoritative_replace` | resolve conflict and insert active fact when policy permits | error if conflict remains unresolved | `reproject_fact_key` |
| `conflicted` | `authorized_retraction` | mark retracted when `060` permits and conflict policy permits | error if unsafe | `expire_projected_object` only when permitted |
| `conflicted` | `staleness_expiry` | mark stale conflicted visibility only when policy permits | otherwise no mutation | `conflict_visibility_update` or `none` |
| `conflicted` | `authoritative_refresh` | resolve conflict or preserve conflicted state per policy | error if resolution row missing | `conflict_visibility_update` or `reproject_fact_key` |
| `conflicted` | `conflict_detected` | `no_op_duplicate` or update conflict evidence | no-op if same evidence | `conflict_visibility_update` |
| `conflicted` | `unsafe_negative` | no mutation | owner `060` blocking reason | `none` |
| `conflicted` | `explicit_no_op` | `no_op_duplicate` change set | none | `none` |
| `unknown` | `duplicate` | `no_op_duplicate` change set | no new fact | `none` |
| `unknown` | `authoritative_replace` | insert active fact when authority passes | error if authority missing | `reproject_fact_key` |
| `unknown` | `authorized_retraction` | emit authorized negative only when `060` permits | otherwise unknown/no mutation | `expire_projected_object` only when permitted |
| `unknown` | `staleness_expiry` | mark stale only when source staleness policy permits stale materialization | otherwise no mutation | `none` |
| `unknown` | `authoritative_refresh` | insert active fact when authority passes | none | `reproject_fact_key` |
| `unknown` | `conflict_detected` | mark conflicted when conflict policy permits | otherwise unknown/no mutation | `conflict_visibility_update` |
| `unknown` | `unsafe_negative` | no mutation | owner `060` blocking reason | `none` |
| `unknown` | `explicit_no_op` | `no_op_duplicate` change set | none | `none` |
| `no_op` | `duplicate` | `no_op_duplicate` change set | no fact state change | `none` |
| `no_op` | `authoritative_replace` | insert fact when policy permits candidate recovery | error if prior no-op cannot be promoted | `reproject_fact_key` |
| `no_op` | `authorized_retraction` | no mutation unless an active policy permits persisted negative no-op evidence | default no-op | `none` |
| `no_op` | `staleness_expiry` | no mutation | no fact state exists to stale | `none` |
| `no_op` | `authoritative_refresh` | insert fact when authority passes | no-op when candidate duplicate | `reproject_fact_key` |
| `no_op` | `conflict_detected` | mark conflicted only when a real comparable fact exists | default no-op | `none` |
| `no_op` | `unsafe_negative` | no mutation | owner `060` blocking reason | `none` |
| `no_op` | `explicit_no_op` | `no_op_duplicate` change set | none | `none` |

### Replay input sufficiency matrix

| Output class | Required refs | Failure code | Mutable-ref behavior | Retention-ineligible behavior | Schema mismatch behavior |
| --- | --- | --- | --- | --- | --- |
| activation-controlled policy artifacts | temporal, knowledge-time, late-arrival, correction, snapshot-ref, replay-equivalence, and output-class rows | `TEMPORAL_ARTIFACT_MISSING` or owner-specific artifact error | reject before replay output | reject before replay output | reject before replay output |
| raw | feed profile, manifest, object/table refs, import profile | `REPLAY_INPUT_INSUFFICIENT` | reject | reject | reject |
| silver | raw refs, parser/mapping, external schema artifact | `REPLAY_INPUT_INSUFFICIENT` | reject | reject | reject |
| identity | resolver profile row, evidence refs, identity decision refs, review case refs when applicable, resolver explanation checksum, graph correction handoff refs when split affects gold or graph output, and version manifest | `REPLAY_INPUT_INSUFFICIENT` | reject | reject | reject |
| gold | temporal, authority, absence, evidence, correction policy, version manifest | `REPLAY_INPUT_INSUFFICIENT` | reject | reject | reject |
| gold correction | old snapshot, new snapshot, table-set checksum, correction policy, transition row, graph handoff refs | `REPLAY_INPUT_INSUFFICIENT` | reject | reject | reject |
| graph delta | projection profile, graph handoff effect, delta refs, schema refs, resolver explanation checksum, and graph correction handoff refs when split affects projection | `REPLAY_INPUT_INSUFFICIENT` | reject | reject | reject |
| graph apply | apply profile, idempotency key, backend evidence, derived-view state | `REPLAY_INPUT_INSUFFICIENT` | reject | reject | reject |
| graph rebuild | rebuild manifest, rebuild equivalence row, index consistency, schema fingerprint | `REPLAY_INPUT_INSUFFICIENT` | reject | reject | reject |
| run-lock governed output | lock policy, lock set, operation evidence, recovery evidence when applicable, commit guard refs, lifecycle transition evidence | `REPLAY_INPUT_INSUFFICIENT` | reject | reject | reject |
| api/export/analysis/validation | owner output-class row, owner refs, authorization/redaction, expected checksum | `REPLAY_POLICY_ARTIFACT_MISSING` or `REPLAY_INPUT_INSUFFICIENT` | reject | reject | reject |

### Gold correction no-op and error behavior

| Omitted or unsafe case | Required result | Error/no-op code | Forbidden effects |
| --- | --- | --- | --- |
| missing temporal policy | temporal error before candidate | `TEMPORAL_POLICY_UNRESOLVED` | no gold, no correction, no graph handoff |
| missing temporal resolution | error before gold ID computation | `TEMPORAL_RESOLUTION_REQUIRED` | no gold, no correction, no graph handoff |
| missing authority row | error before fact creation | `GOLD_FACT_AUTHORITY_ROW_MISSING` or `SOURCE_AUTHORITY_ROW_MISSING` | no gold, no absence, no retraction |
| ambiguous authority row | error before fact creation | `SOURCE_AUTHORITY_ROW_AMBIGUOUS` | no gold, no absence, no retraction |
| missing correction policy | error before correction | `CORRECTION_POLICY_MISSING` | no correction, no graph handoff |
| missing transition row | error before correction | `GOLD_CORRECTION_TRANSITION_UNDEFINED` | no correction, no graph handoff |
| missing snapshot refs | error before correction | `CORRECTION_SNAPSHOT_REF_MISSING` | no correction, no graph handoff |
| mutable snapshot refs | replay/correction rejection | `MUTABLE_BRANCH_REF_FOR_REPLAY` | no correction, no replay output |
| duplicate candidate | explicit no-op changeset | `no_op_duplicate` | no active fact, no graph delta |
| unauthorized negative evidence | no mutation and preserve diagnostic | most specific `060` blocking reason | no absence, retraction, cleanup, graph expiry, watermark |
| unauthorized CDC tombstone | call `060.DeriveAbsenceOrUnknown`, emit imported `060.CDC_TOMBSTONE_RETRACTION_UNAUTHORIZED`, and preserve raw delete evidence | `060.CDC_TOMBSTONE_RETRACTION_UNAUTHORIZED` | no retraction, cleanup, graph expiry, watermark |
| confidence-only same value | explicit no-op changeset after canonical confidence comparison | `no_op_duplicate` | no active fact, no graph delta |
| stale without staleness policy | error or unknown per `060` | `SOURCE_STALENESS_POLICY_ROW_MISSING` | no stale fact, no retraction, no expiry |
| replay missing output-class row | replay blocked before output | `REPLAY_POLICY_ARTIFACT_MISSING` | no replay output |

### Graph handoff effects

`080` owns emitted graph handoff metadata only. `080` never mutates graph state. `090` owns graph delta projection, apply, and serving behavior.

When `identity_split_handoff` is emitted, `080` must validate the `070.GraphCorrectionHandoff` checksum before emitting the handoff effect. Checksum drift blocks correction and graph projection before output.

| Graph handoff effect | Emitted by `080` when | Required `090` handling |
| --- | --- | --- |
| `none` | no-op, blocked error, unsafe negative, or correction with no graph-visible impact | emit no graph delta |
| `reproject_fact_key` | active fact insert, replacement, refresh, interval split, conflict resolution, or supersession affects a projected fact key | recompute projected graph objects for the affected fact key |
| `expire_projected_object` | authorized retraction may expire graph object and `060` permits `graph_expiry` | emit expiry only through `090.GraphExpirySourceAuthorityGate` |
| `cleanup_projected_object` | authorized cleanup may remove projection object and `060` permits `cleanup` | emit cleanup only through `090.GraphExpirySourceAuthorityGate` |
| `conflict_visibility_update` | stale or conflicted assertion visibility changes without fact-key replacement | update visibility only when graph semantics and eligibility rows permit |
| `identity_split_handoff` | `070.GraphCorrectionHandoff` validates split decision ref, split policy ref, retained and new canonical entity refs, split partition refs, affected fact refs or affected fact-selection checksum, resolver explanation checksum, and version manifest ref | route through `090.ProjectGraphDeltas` only when the active `090.GraphProjectionProfile` supports the handoff; unsupported profile handoff emits no graph delta and records the owner error; missing or checksum-mismatched handoff refs fail before correction/projection with no graph mutation |

### TemporalApiHandoff

Temporal, correction, no-op, and replay failures are owner errors with audit implications. They must not collapse into source unknowns, compliance pass/fail states, or absence outcomes.

| Temporal or replay condition | Recommended `110` label or outcome | Required behavior |
| --- | --- | --- |
| temporal policy unresolved | `error` | Emit `TEMPORAL_POLICY_UNRESOLVED`; no gold candidate. |
| source time not authorized | `error` | Emit `SOURCE_TIME_NOT_AUTHORIZED`; no current-time fallback. |
| correction policy missing | `error` | Emit `CORRECTION_POLICY_MISSING`; no correction output. |
| duplicate no-op correction | no output change plus audit evidence | Emit no graph or compliance mutation; audit the no-op evidence. |
| replay input insufficient | replay/audit error, no source-state label | Emit `REPLAY_INPUT_INSUFFICIENT`; no production replay output. |
| replay checksum mismatch | replay/audit error, no source-state label | Emit `REPLAY_CHECKSUM_MISMATCH`; no output substitution. |
| stale known-state output | `source_stale` | Preserve source stale state and do not treat as derived-view lag. |

### Temporal artifact errors

| Error code | Emitted when |
| --- | --- |
| `TEMPORAL_POLICY_UNRESOLVED` | No exact active temporal policy row covers the source dataset, observation type, fact type, predicate, source scope, and temporal mode, or more than one equally specific row covers it. |
| `TEMPORAL_INTERVAL_MODEL_MISSING` | A temporal row omits `valid_interval_model` or `valid_to_rule`. |
| `SOURCE_TIME_NOT_AUTHORIZED` | Selected source time is absent from the row's authorized precedence or quality set. |
| `TEMPORAL_RESOLUTION_REQUIRED` | A gold, correction, replay, graph, or audit operation requires a temporal resolution row and none exists. |
| `TEMPORAL_ARTIFACT_MISSING` | Required temporal, knowledge-time, late-arrival, correction, snapshot-ref, or replay artifact ref is missing. |
| `TEMPORAL_ARTIFACT_INACTIVE` | Required temporal artifact is not active for production execution. |
| `TEMPORAL_ARTIFACT_CHECKSUM_MISMATCH` | Required temporal artifact checksum mismatches the active ref or manifest. |
| `LATE_ARRIVAL_POLICY_MISSING` | Late-arrival routing is requested without an active row. |
| `LATE_ARRIVAL_DISCARD_FORBIDDEN` | A route attempts to discard production evidence. |
| `CORRECTION_POLICY_MISSING` | Correction behavior is requested without an active `GoldFactCorrectionPolicy` row. |
| `GOLD_CORRECTION_TRANSITION_UNDEFINED` | No transition row covers the prior assertion state and correction event. |
| `CORRECTION_SNAPSHOT_REF_MISSING` | Old snapshot, new snapshot, table-set checksum, or retention-protection ref is missing. |
| `MUTABLE_BRANCH_REF_FOR_REPLAY` | Replay or correction uses mutable-only branch, tag, latest, or workspace refs. |
| `REPLAY_POLICY_ARTIFACT_MISSING` | Required replay equivalence or output-class policy artifact is absent before replay. |
| `REPLAY_INPUT_INSUFFICIENT` | Required replay input, sufficiency check, snapshot ref, retention ref, manifest ref, or owner ref is absent. |
| `REPLAY_CHECKSUM_MISMATCH` | Replay output checksum differs after all higher-precedence failures are excluded. |

| `GOLD_FACT_PREDICATE_CONTRACT_MISSING` | No active predicate contract row covers the candidate fact type and predicate. |
| `GOLD_FACT_PREDICATE_CONTRACT_AMBIGUOUS` | More than one equally specific active predicate contract row covers the candidate fact type and predicate. |
| `GOLD_FACT_PREDICATE_CONTRACT_BLOCKED` | A deterministic predicate block row covers the candidate and forbids output. |
| `GOLD_FACT_PREDICATE_SUBJECT_KIND_FORBIDDEN` | The candidate subject reference kind is not allowed by the selected predicate row. |
| `GOLD_FACT_PREDICATE_OBJECT_KIND_FORBIDDEN` | The candidate object value kind or `object_kind` is not allowed by the selected predicate row. |
| `GOLD_FACT_PREDICATE_REFERENCE_ENTITY_TYPE_FORBIDDEN` | A canonical entity ref has an entity type not permitted by the selected predicate row. |
| `GOLD_FACT_PREDICATE_REFERENCE_IDENTIFIER_TYPE_FORBIDDEN` | An identifier ref has an identifier type not permitted by the selected predicate row. |
| `GOLD_FACT_PREDICATE_NULL_FORBIDDEN` | A candidate uses `null_value` where the selected predicate row has `null_object_policy = forbidden`. |
| `GOLD_FACT_PREDICATE_IDENTITY_STRING_FORBIDDEN` | A candidate encodes identity-like content as a `string_value` where the selected predicate row requires a reference or structured schema. |
| `GOLD_FACT_STRUCTURED_SCHEMA_MISSING` | A candidate structured object lacks an active exact schema ref named by the selected predicate row. |
| `GOLD_FACT_STRUCTURED_SCHEMA_CHECKSUM_MISMATCH` | A structured schema checksum mismatches the active manifest entry. |
| `GOLD_FACT_PREDICATE_ROW_TODO` | A selected predicate row, block row, or structured schema row contains a promotion-blocking `TODO:`. |
| `GOLD_FACT_PREDICATE_PACKAGE_SET_MISSING` | A package-supplied predicate row set, block row, or structured schema row lacks a package-set ref. |
| `GOLD_FACT_PREDICATE_MANIFEST_INCOMPLETE` | A selected predicate row, block row, structured schema row, validation row, or package-set ref is absent from `030.VersionManifest`. |

### TemporalErrorRegistryFragment

This owner fragment feeds `110.GenerateErrorCodeRegistry`. `110` owns the generated caller-visible registry. This table must not render API output by itself. Rows with `TODO:` cells block authoritative promotion and must be resolved by the owning domain before `110-ERROR-REGISTRY-TOTAL-AC-001` can pass.

| error_code | owner_spec | severity | retry_class | caller_visible_fields | audit_visible_fields | redaction_rule | owner_context_schema_ref | fixture_ref |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `TEMPORAL_POLICY_UNRESOLVED` | `080` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `080.TemporalErrorContext` | `error-registry-080-temporal-policy-unresolved` |
| `TEMPORAL_INTERVAL_MODEL_MISSING` | `080` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `080.TemporalErrorContext` | `error-registry-080-temporal-interval-model-missing` |
| `SOURCE_TIME_NOT_AUTHORIZED` | `080` | `error` | `caller_correctable` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `080.TemporalErrorContext` | `error-registry-080-source-time-not-authorized` |
| `TEMPORAL_RESOLUTION_REQUIRED` | `080` | `error` | `caller_correctable` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `080.TemporalErrorContext` | `error-registry-080-temporal-resolution-required` |
| `TEMPORAL_ARTIFACT_MISSING` | `080` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `080.TemporalErrorContext` | `error-registry-080-temporal-artifact-missing` |
| `TEMPORAL_ARTIFACT_INACTIVE` | `080` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `080.TemporalErrorContext` | `error-registry-080-temporal-artifact-inactive` |
| `TEMPORAL_ARTIFACT_CHECKSUM_MISMATCH` | `080` | `error` | `retry_after_owner_repair` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `080.TemporalErrorContext` | `error-registry-080-temporal-artifact-checksum-mismatch` |
| `LATE_ARRIVAL_POLICY_MISSING` | `080` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `080.TemporalErrorContext` | `error-registry-080-late-arrival-policy-missing` |
| `LATE_ARRIVAL_DISCARD_FORBIDDEN` | `080` | `error` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `080.TemporalErrorContext` | `error-registry-080-late-arrival-discard-forbidden` |
| `CORRECTION_POLICY_MISSING` | `080` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `080.TemporalErrorContext` | `error-registry-080-correction-policy-missing` |
| `GOLD_CORRECTION_TRANSITION_UNDEFINED` | `080` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `080.TemporalErrorContext` | `error-registry-080-gold-correction-transition-undefined` |
| `CORRECTION_SNAPSHOT_REF_MISSING` | `080` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `080.TemporalErrorContext` | `error-registry-080-correction-snapshot-ref-missing` |
| `MUTABLE_BRANCH_REF_FOR_REPLAY` | `080` | `error` | `caller_correctable` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `080.TemporalErrorContext` | `error-registry-080-mutable-branch-ref-for-replay` |
| `REPLAY_POLICY_ARTIFACT_MISSING` | `080` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `080.TemporalErrorContext` | `error-registry-080-replay-policy-artifact-missing` |
| `REPLAY_INPUT_INSUFFICIENT` | `080` | `blocked` | `retry_after_refresh` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `080.TemporalErrorContext` | `error-registry-080-replay-input-insufficient` |
| `REPLAY_CHECKSUM_MISMATCH` | `080` | `error` | `retry_after_owner_repair` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `080.TemporalErrorContext` | `error-registry-080-replay-checksum-mismatch` |

| `GOLD_FACT_PREDICATE_CONTRACT_MISSING` | `080` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `080.TemporalErrorContext` | `error-registry-080-gold-fact-predicate-contract-missing` |
| `GOLD_FACT_PREDICATE_CONTRACT_AMBIGUOUS` | `080` | `error` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `080.TemporalErrorContext` | `error-registry-080-gold-fact-predicate-contract-ambiguous` |
| `GOLD_FACT_PREDICATE_CONTRACT_BLOCKED` | `080` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `080.TemporalErrorContext` | `error-registry-080-gold-fact-predicate-contract-blocked` |
| `GOLD_FACT_PREDICATE_SUBJECT_KIND_FORBIDDEN` | `080` | `error` | `caller_correctable` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `080.TemporalErrorContext` | `error-registry-080-gold-fact-predicate-subject-kind-forbidden` |
| `GOLD_FACT_PREDICATE_OBJECT_KIND_FORBIDDEN` | `080` | `error` | `caller_correctable` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `080.TemporalErrorContext` | `error-registry-080-gold-fact-predicate-object-kind-forbidden` |
| `GOLD_FACT_PREDICATE_REFERENCE_ENTITY_TYPE_FORBIDDEN` | `080` | `error` | `caller_correctable` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `080.TemporalErrorContext` | `error-registry-080-gold-fact-predicate-reference-entity-type-forbidden` |
| `GOLD_FACT_PREDICATE_REFERENCE_IDENTIFIER_TYPE_FORBIDDEN` | `080` | `error` | `caller_correctable` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `080.TemporalErrorContext` | `error-registry-080-gold-fact-predicate-reference-identifier-type-forbidden` |
| `GOLD_FACT_PREDICATE_NULL_FORBIDDEN` | `080` | `error` | `caller_correctable` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `080.TemporalErrorContext` | `error-registry-080-gold-fact-predicate-null-forbidden` |
| `GOLD_FACT_PREDICATE_IDENTITY_STRING_FORBIDDEN` | `080` | `error` | `caller_correctable` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `080.TemporalErrorContext` | `error-registry-080-gold-fact-predicate-identity-string-forbidden` |
| `GOLD_FACT_STRUCTURED_SCHEMA_MISSING` | `080` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `080.TemporalErrorContext` | `error-registry-080-gold-fact-structured-schema-missing` |
| `GOLD_FACT_STRUCTURED_SCHEMA_CHECKSUM_MISMATCH` | `080` | `error` | `retry_after_owner_repair` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `080.TemporalErrorContext` | `error-registry-080-gold-fact-structured-schema-checksum-mismatch` |
| `GOLD_FACT_PREDICATE_ROW_TODO` | `080` | `blocked_validation` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `080.TemporalErrorContext` | `error-registry-080-gold-fact-predicate-row-todo` |
| `GOLD_FACT_PREDICATE_PACKAGE_SET_MISSING` | `080` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `080.TemporalErrorContext` | `error-registry-080-gold-fact-predicate-package-set-missing` |
| `GOLD_FACT_PREDICATE_MANIFEST_INCOMPLETE` | `080` | `blocked` | `retry_after_owner_repair` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `080.TemporalErrorContext` | `error-registry-080-gold-fact-predicate-manifest-incomplete` |

### TemporalErrorContext

`TemporalErrorContext` is the owner context schema for `080` temporal, replay, correction, late-arrival, predicate-contract, snapshot-ref, and event-sequence registry rows. It satisfies `110.OwnerErrorContextMinimumSchema` and must not expose raw payload bytes, private bindings, credentials, source-native identity values, raw temporal evidence values, or private fixture bytes to callers.

| Field | Required | Rule |
| --- | ---: | --- |
| `context_schema_version` | Yes | Immutable `080` context schema version. |
| `owner_spec` | Yes | Must be `080`. |
| `error_code` | Yes | Must match the generated registry row. |
| `failure_class` | Yes | Closed token: `temporal_policy`, `time_resolution`, `knowledge_time`, `late_arrival`, `correction_policy`, `correction_snapshot`, `replay`, `predicate_contract`, `event_sequence`, or `graph_rebuild_handoff`. |
| `operation` | Yes | Fact-time resolution, known-time import, late-arrival evaluation, gold correction, replay sufficiency check, replay checksum computation, predicate contract resolution, or event-sequence validation. |
| `affected_record_type` | Yes | Temporal policy, knowledge-time policy, temporal resolution row, late-arrival policy, correction policy, correction snapshot ref, replay policy, predicate contract row, event-sequence corpus, gold fact candidate, or version manifest. |
| `field_path` | Yes | Exact field path when applicable; null for artifact-wide failures. |
| `artifact_refs` | Yes | Canonically sorted refs to temporal policy rows, knowledge-time policy rows, late-arrival policy rows, correction policy rows, replay policy rows, correction snapshot refs, event-sequence corpus rows, predicate contract rows, temporal resolution refs, validation fixtures, or version manifests consulted by the error; empty only when no artifact was consulted. |
| `validation_refs` | Yes | Exact `120` temporal, correction, replay, and event-sequence fixture refs. |
| `redaction_classes` | Yes | Map every nested owner-context field to one `110.ErrorRedactionClassMatrix` class. Raw payload bytes, private bindings, credentials, source-native identity values, raw temporal values, sensitive evidence refs, and private fixture bytes must map to `always_forbidden`. |
| `blocking_reason` | Yes when generated row severity is `blocked` | Bounded reason; otherwise null or omitted. |
| `affected_record_ref` | No | Required when a persisted temporal, correction, replay, or gold candidate record was evaluated. |
| `policy_ref` | No | Required when a policy row was selected, missing, inactive, or checksum-mismatched. |
| `temporal_input_path` | No | Required for malformed, unauthorized, ambiguous, or missing time-input failures. |
| `replay_output_class` | No | Required for replay policy or checksum failures. |
| `snapshot_ref_class` | No | Required for correction snapshot and mutable-ref failures. |

### ActivationControlledRowSchemaPrecisionHandoff

The following `080` row families can affect fact-time resolution, known-time import, late-arrival routing, correction behavior, snapshot refs, replay equivalence, graph rebuild equivalence, predicate contracts, assertion-state transitions, or event-sequence validation. Each output-affecting family must use a complete `030.ActivationControlledRowField` table before production selection. Until the required table is present and non-`TODO`, `ValidateSpecSet` must classify the family as `blocked_validation`.

| row_family | production classification | required precision status |
| --- | --- | --- |
| `TemporalSemanticsPolicy` | output_affecting | TODO: add full field precision for valid interval model, valid time input precedence, time quality, fallback behavior, absent/malformed/ambiguous behavior, valid-to rule, validation refs, activation scope, and lifecycle status. |
| `KnowledgeTimeImportPolicy` | output_affecting | TODO: add full field precision for current import, reconstructed known-time permission, required evidence refs, and missing evidence behavior. |
| `LateArrivalPolicy` | output_affecting | TODO: add full field precision for route states, authority/completeness/coverage/staleness refs, quarantine refs, watermark refs, and errors. |
| `GoldFactCorrectionPolicy` | output_affecting | Source-effect correction candidates must carry selected `020.SourceDatasetCatalogRow`, `060.SourceAuthorityClosureMatrixRow`, and `AbsenceDerivationResult` refs before correction. Remaining non-source-effect correction event and assertion-transition precision remains blocked until separately closed. |
| `CorrectionSnapshotRefPolicy` | output_affecting | Closed for lakehouse correction closure by `CorrectionSnapshotRefPolicy` row precision above: old/new snapshot roles, table-set checksum, commit refs, retention protection, schema compatibility, mutable-ref rejection, validation refs, activation scope, lifecycle status, and row checksum are explicit. |
| `ReplayEquivalencePolicyOutputClassRow` | output_affecting | TODO: add full field precision for included fields, excluded volatile fields, checksum inclusion, failure precedence, and manifest requirements. |
| `GraphRebuildEquivalencePolicy` | output_affecting when graph rebuild is in scope | TODO: add full field precision for graph-specific included/excluded refs in the `080`/`090` handoff. |
| `GoldFactPredicateContractRow` | output_affecting | Closed for source-effect handoff by the existing field table for fact type, predicate, subject/object kind sets, null object policy, structured schema refs, source authority refs, checksum inputs, validation refs, activation scope, lifecycle status, and owner errors. Remaining non-source-effect predicate precision work remains blocked until separately closed. |
| `AssertionStateTransitionRow` | output_affecting | TODO: add total transition table precision over prior assertion state and correction event. |
| `EventSequenceValidationCorpus` | validation_affecting | TODO: add full field precision for sequence ID, ordered events, expected output/error/no-op, mutation prohibition, and fixture checksums. |

`valid_time_input_precedence` must use `ordered_sequence`. Fallback behavior must be a closed enum or owner union; omitted fallback must map to a deterministic temporal error. Replay included and excluded fields must use declared `ordered_sequence` or `canonical_set` semantics. Correction transition rows must be total over prior assertion state and correction event.

Gold derivation, correction, late-arrival routing, replay output, or graph rebuild handoff must fail before output when any selected `080` row family remains `TODO:`, uses a mutable-only ref, omits row checksum inputs, or omits selected row refs from `030.VersionManifest`.

### Acceptance Criteria

| ID | Criterion |
| --- | --- |
| `080-DERIVATION-RULE-BUNDLE-HANDOFF-AC-001` | A valid `130.DerivationRuleBundle` routes through `DeriveFacts`, missing authority/completeness/temporal refs block output before fact ID computation, and direct package-emitted `GoldFact` bytes fail before persistence. |
| `080-DERIVED-EDGE-GOLD-OUTPUT-AC-001` | `130.DerivedGraphEdgeRule.allowed_output_effect = gold_fact_via_080` must pass temporal, authority, completeness, correction, replay, schema, and validation gates before any fact output. |
| `080-API-HANDOFF-AC-001` | Temporal and replay handoff fixtures prove policy failures render as owner errors, duplicate no-op corrections render as no output change with audit evidence, replay failures do not become source-state labels, and stale known-state output renders as `source_stale`. |
| `080-CLEANUP-AC-001` | No banned reference class remains. |
| `080-CLEANUP-AC-002` | Source event time, observation time, supplier collection time, supplier delivery time, lakehouse commit time, table snapshot time, CDC time, graph apply time, replay time, and platform current time remain distinct. |
| `080-CLEANUP-AC-003` | `ResolveFactTime` rejects implicit current-time fallback for absent, malformed, ambiguous, or non-authoritative time. |
| `080-CLEANUP-AC-004` | Corrections remain append-only knowledge transitions and never mutate `GoldFact` rows in place. |
| `080-SCHEMA-PATCH-AC-001` | Every emitted `GoldFact` passes `040.GoldFactSchema`. |
| `080-SCHEMA-PATCH-AC-002` | `gold_fact_key_id` and `gold_fact_id` are both present and computed by the declared algorithms. |
| `080-SCHEMA-PATCH-AC-003` | Corrections never mutate original `GoldFact` bytes. |
| `080-SCHEMA-PATCH-AC-004` | Replay rejects gold output when any 040 schema, temporal, authority, evidence, or correction checksum mismatches. |
| `080-KNOWLEDGE-TIME-AC-001` | Production known time defaults to `current_import`, and historical known-time reconstruction is rejected unless policy and persisted source-known-time evidence permit it. |
| `080-REPLAY-SPLIT-AC-001` | Projection, API, and analysis replay rows import owner-specific included/excluded fields and use `080` only for checksum algorithm and preflight ordering. |
| `080-VOLATILITY-AC-001` | Inactive temporal policy, missing knowledge-time policy, correction policy checksum mismatch, late-arrival row manifest omission, and replay policy row absence fail before gold or replay output. |
| `080-TEMPORAL-POLICY-AC-001` | Every gold-producing source/fact tuple has exactly one active temporal policy row or deterministic blocker. |
| `080-TEMPORAL-RESOLUTION-AC-001` | Every GoldFact candidate has exactly one temporal resolution or deterministic temporal error before candidate creation. |
| `080-LATE-ARRIVAL-AC-001` | Late authoritative evidence after cutoff routes to correction evaluation and is not silently discarded. |
| `080-CORRECTION-MATRIX-AC-001` | Every default assertion-state and correction-event pair has exactly one output. |
| `080-CORRECTION-NOOP-AC-001` | Duplicate candidates and confidence-only same-value updates emit `no_op_duplicate` and no graph handoff. |
| `080-UNSAFE-NEGATIVE-AC-001` | Unauthorized negative evidence emits no absence, retraction, cleanup, graph expiry, or watermark advancement. |
| `080-SOURCE-DATASET-CATALOG-AC-001` | Absence-sensitive gold candidates reject before `GoldFact` or `GoldFactChangeSet` creation when selected source-dataset catalog refs are missing, blocked, ambiguous, inactive, checksum-mismatched, unvalidated, or unmanifested. |
| `080-SOURCE-DATASET-CATALOG-AC-002` | Every row in `AbsenceSensitiveGoldCandidateMatrix` includes selected `020.SourceDatasetCatalogRow` or deterministic source-dataset block refs in `VersionManifest`. |
| `080-CORRECTION-SNAPSHOT-AC-001` | Every correction class requires old snapshot ref, new snapshot ref, table-set checksum, retention protection, and mutable-ref rejection. |
| `080-REPLAY-OUTPUT-CLASS-AC-001` | Replay rows exist for raw, silver, identity, gold, gold correction, graph delta, graph apply, graph rebuild, API response, export projection, analysis output, and validation acceptance. |
| `080-TELEMETRY-REPLAY-AC-001` | Trace IDs, span IDs, sampling decisions, exporter state, Collector state, backend telemetry IDs, runtime duration, and dropped telemetry counts are excluded from authoritative domain replay checksums unless explicitly included for telemetry health diagnostics by `140` and manifest refs. |
| `080-RUNLOCK-REPLAY-AC-001` | Replay missing `030.RunLockOperationEvidence`, stale recovery evidence, or commit guard refs for a lock-governed output fails with `REPLAY_INPUT_INSUFFICIENT` before output. |
| `080-REPLAY-PRECEDENCE-AC-001` | Replay rejects missing, mutable-only, retention-ineligible, schema-incompatible, authority-mismatched, temporal-mismatched, side-effect-mismatched, or checksum-mismatched inputs according to global precedence before output. |
| `080-GRAPH-HANDOFF-AC-001` | Every correction graph handoff effect is emitted as metadata only, and `080` performs no graph mutation. |
| `080-SOURCE-CLOSURE-GOLD-AC-001` | Missing `AbsenceDerivationResult` blocks absence-sensitive `GoldFact` output. |
| `080-SOURCE-CLOSURE-CORRECTION-AC-001` | Source deletion evidence without exact `060` closure emits no retraction. |
| `080-SOURCE-CLOSURE-REPLAY-AC-001` | Replay rejects gold output when any consulted `060` row checksum differs. |
| `080-SOURCE-CLOSURE-MATRIX-AC-001` | Every row in `AbsenceSensitiveGoldCandidateMatrix` has event-sequence fixtures for success, missing refs, deterministic block, manifest omission, and checksum drift. |
| `080-IDENTITY-SPLIT-HANDOFF-AC-001` | A valid identity split handoff contains `split_identity_decision_ref`, `split_policy_ref`, `retained_canonical_entity_ref`, `new_canonical_entity_refs`, `split_partition_refs`, `affected_fact_refs` or `affected_fact_selection_checksum`, `resolver_explanation_checksum`, and `version_manifest_ref`. |
| `080-IDENTITY-SPLIT-HANDOFF-AC-002` | Missing identity split handoff metadata fails before correction or projection and emits no graph handoff effect except `none`. |
| `080-IDENTITY-SPLIT-HANDOFF-AC-003` | Checksum drift in `070.GraphCorrectionHandoff` or resolver explanation rejects replay before output. |
| `080-IDENTITY-SPLIT-HANDOFF-AC-004` | `080` treats identity split handoff as immutable metadata and never mutates graph state. |
| `080-GRAPH-DELTA-REPLAY-HANDOFF-AC-001` | Graph delta replay imports `090` profile, edge semantics, output eligibility, property mapping, graph correction handoff, resolver explanation, and delta checksum fields. |
| `080-GRAPH-HANDOFF-PROFILE-AC-001` | Unsupported active graph profile handoff emits no graph delta and does not restate graph projection behavior. |
| `080-GRAPH-HANDOFF-CHECKSUM-AC-001` | Identity split handoff checksum drift rejects replay before graph output. |
| `080-GRAPH-HANDOFF-DENIED-EXPIRY-AC-001` | Denied expiry or cleanup authorization emits no graph handoff effect other than no-op. |
| `080-GOLD-PREDICATE-CONTRACT-AC-001` | Missing predicate contract row blocks `GoldFact` output before `gold_fact_key_id` computation with `GOLD_FACT_PREDICATE_CONTRACT_MISSING`. |
| `080-GOLD-PREDICATE-CONTRACT-AC-002` | Disallowed subject kind, disallowed object kind, and `object_kind != object_value.kind` block output before ID computation. |
| `080-GOLD-PREDICATE-CONTRACT-AC-003` | `null_value` is accepted only when the selected predicate contract sets `null_object_policy = allowed`; otherwise it fails before ID computation. |
| `080-GOLD-PREDICATE-CONTRACT-AC-004` | Identity-like string object values are rejected when the predicate contract requires a reference kind. |
| `080-GOLD-PREDICATE-CONTRACT-AC-005` | `structured_value` requires one active structured schema ref named by the predicate contract and included in `VersionManifest`. |
| `080-OCSF-STRUCTURED-OBJECT-AC-001` | OCSF-derived structured object output fails before `gold_fact_key_id` computation unless `050.ProfileResolutionManifest`, `080.GoldFactPredicateContractRow.structured_value_schema_refs`, and required `060` authority refs validate and appear in `VersionManifest`. |
| `080-GOLD-PREDICATE-CONTRACT-REPLAY-AC-001` | Gold and gold-correction replay fail when the selected predicate contract checksum changes. |
| `080-GOLD-PREDICATE-CATALOG-CLOSURE-AC-001` | The active MVP predicate catalog contains exactly the 18 active rows in `MVPGoldFactPredicateActiveRowInventory` and the six deterministic block rows in `GoldFactPredicateDeterministicBlockInventory`; missing, duplicate, extra-active, inactive, `TODO`-bearing, or checksum-mismatched rows fail validation. |
| `080-GOLD-PREDICATE-CATALOG-CLOSURE-AC-002` | Every active predicate row uses only `040` subject/object kinds and rejects forbidden entity types, forbidden identifier types, forbidden nulls, and identity-like strings before ID computation. |
| `080-GOLD-PREDICATE-CATALOG-CLOSURE-AC-003` | Every structured object candidate uses one exact active schema ref from `MVPGoldFactStructuredValueSchemaCatalog`; missing or checksum-mismatched schema refs fail before `gold_fact_key_id` computation. |
| `080-GOLD-PREDICATE-CATALOG-CLOSURE-AC-004` | Deterministic block rows for modeled reachability, theoretical reachability, and ownership emit no fact, no correction, no graph handoff except `none`, no watermark, and no API authorized-negative output. |
| `080-GOLD-PREDICATE-CATALOG-CLOSURE-AC-005` | Gold replay and correction replay fail when the row-set checksum, selected row checksum, selected structured schema checksum, source authority ref, or manifest inclusion differs from the recorded `VersionManifest`. |
| `080-GOLD-PREDICATE-CATALOG-CLOSURE-AC-006` | Package-supplied predicate row sets and structured schema row sets fail activation when package-set refs or validation refs are missing. |
| `080-DOMAIN-RESOLVED-ROUTING-AC-001` | Domain rows for `CorrectionSnapshotRefPolicy` and `ReplayEquivalencePolicy` remain resolved only while this file has no owner-local `TODO:` for those stable contracts and the required `120` temporal, correction, and replay validation row families pass. |
| `080-LAKEHOUSE-REPLAY-INPUT-AC-001` | Replay rejects missing snapshot, commit, retention, schema compatibility, or table-set refs before generic mismatch. |
| `080-CORRECTION-LAKEHOUSE-REF-AC-001` | Corrections reject mutable-only old/new refs and missing table-set checksum before output. |
| `080-RETENTION-HANDOFF-AC-001` | Replay and correction preflight consume `ReplayRetentionPolicy` and retention-protection refs when maintenance could remove replay inputs. |

### Structured input temporal and replay acceptance criteria

| ID | Criterion |
| --- | --- |
| `080-STRUCTURED-INPUT-TEMPORAL-AC-001` | Git-authored temporal, correction, late-arrival, replay, predicate, assertion, or event-sequence rows do not affect fact time, known time, correction, replay, graph handoff, or gold derivation until active materialized refs and manifest refs exist. |
| `080-STRUCTURED-INPUT-REPLAY-AC-001` | Replay fails before output when structured-input-derived policy refs are omitted from `VersionManifest` or when validation is stale for the exact snapshot. |
| `080-STRUCTURED-INPUT-REPLAY-AC-004` | Missing tool invocation ref, stale producer CI, publication manifest mismatch, sync-only replay attempt, and repository-authored predicate row omission from `VersionManifest` fail before replay output. |

## Definition of Done

| ID | Criterion |
| --- | --- |
| `080-SCOPE-TEMPORAL-AC-001` | Exact temporal policy match, missing source-scope dimension, subset-disallowed temporal policy, ambiguous temporal rows, and `TEMPORAL_POLICY_UNRESOLVED` with no `GoldFact` candidate fixtures pass. |
| `080-SCOPE-TEMPORAL-AC-002` | Temporal policy selection includes selected selector checksum and context ref in `VersionManifest` when policy selection affects gold output. |
| `080-AC-001` | Current, valid-at, known-at, corrected-history, replay, and audit queries are reconstructable from persisted records. |
| `080-AC-002` | Every gold fact candidate has a temporal resolution row or deterministic temporal error. |
| `080-AC-003` | Corrections never mutate existing `GoldFact` rows in place. |
| `080-AC-004` | Replay rejects missing, mutable-only, retention-ineligible, schema-incompatible, authority-mismatched, temporal-mismatched, side-effect-mismatched, or checksum-mismatched inputs before writing output. |
| `080-AC-005` | Event-sequence validation covers temporal resolution, late arrival, correction, replay, side effects, no-op/error behavior, graph handoff, and graph rebuild equivalence. |
| `080-AC-006` | Authoritative promotion fails if any temporal, correction, late-arrival, replay, snapshot-ref, or assertion transition behavior contains a blocking placeholder row. |

## Open Questions

No owner-local temporal, correction, late-arrival, replay, correction-snapshot, no-op/error, or graph-handoff question remains open in this document. Concrete production fact catalogs, source datasets, predicates, private source bindings, and source-specific active row instances remain activation-controlled artifacts and validation-blocked when selected for production scope. Concrete production fact-type and predicate row catalogs are activation-controlled artifacts. Their absence blocks only the selected production scope and must not re-open the stable `080.CorrectionSnapshotRefPolicy` or `080.ReplayEquivalencePolicy` owner-local closure state.
