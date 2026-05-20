---
doc_id: CADASTRE-NLSPEC-020
title: Lakehouse Feeds and Table State
doc_type: nlspec
status: candidate
---

## Authority

This document owns the contracts listed in `Exports`. Other Cadastre NLSpecs may import those contracts by exact name and must not restate them. This document has implementation authority only after the document registry marks it `authoritative` and its acceptance criteria pass.

## Purpose

Define how Cadastre reads lakehouse-resident raw feeds, imports raw records, references table state, and preserves replay inputs.

## Explicit Non-Scope

- Source authority interpretation.
- Parser-to-silver behavior.
- Identity resolution.
- Graph projection or apply behavior.
- Package trust.

## Imports

- `DirectSourceProhibition`
- `AuthorityClass`

- `RawRecord`
- `ComputeRawRecordId`
- `CoreRecordValidationAlgorithm`
- `ActivationControlledArtifactRef`
- `EvidenceRef`
- `EvidenceArtifactIdKindRegistry`

## Exports

- `RawSupplierProfile`
- `LakehouseFeedProfile`
- `LakehouseFeedProfileSchema`
- `LakehouseFeedFeasibilityAssessment`
- `LakehouseFeedCategoryClosureRow`
- `LakehouseFeedCategoryClosureRowSet`
- `LakehouseFeedAvailabilityCheck`
- `RawFeedManifest`
- `LakehouseReadPolicy`
- `RawRecordImportRun`
- `LakehouseReadCompletenessReceipt`
- `UpstreamCompletenessEvidence`
- `LakehouseTableProfile`
- `LakehouseSnapshotRef`
- `DatasetVersionRef`
- `LakehouseCommitRef`
- `ReplayRetentionPolicy`
- `TableMaintenancePolicy`
- `ReplayRetentionDecision`
- `CrossTableCommitProfile`
- `CatalogBranchPromotionPolicy`
- `FeedStageLifecycleEventDerivation`

## Feed Read Contract

A production feed read must start from an active `LakehouseFeedProfile`. The profile must name the supplier class, read target kind, scope keys, schema refs, object or table refs, package bindings, replay-relevant configuration hashes, and redacted access-reference hashes.

A production feed read must validate the profile against `LakehouseFeedProfileSchema` and must resolve exactly one active `LakehouseFeedCategoryClosureRow` from an active `LakehouseFeedCategoryClosureRowSet` before reading or importing raw records. The phrase `profile permits` is not runtime behavior. Every profile-dependent branch must resolve to a named profile field, an active activation-controlled artifact ref, a deterministic branch result, and a validation row.

| Read target kind | Required reference | Output allowed |
| --- | --- | --- |
| `table_snapshot` | `LakehouseSnapshotRef` | Raw feed rows and read completeness receipt. |
| `dataset_version` | `DatasetVersionRef` | Raw feed rows and read completeness receipt. |
| `object_batch` | `RawFeedManifest` object refs | Raw feed objects and read completeness receipt. |
| `partition_set` | `RawFeedManifest` partition refs | Raw feed rows and read completeness receipt. |
| `manifest_list` | `RawFeedManifest` | Manifest validation result and read completeness receipt. |

`LakehouseFeedAvailabilityCheck` must validate catalog, table, object, partition, manifest, and schema availability without enterprise source calls or observation-producing side effects.

### LakehouseFeedProfileSchema

`LakehouseFeedProfileSchema` is the stable schema for every `LakehouseFeedProfile`. A profile with a missing required field must fail before lakehouse availability checks, feed reads, raw import, completeness evaluation, absence evaluation, cleanup, graph expiry, retraction, or watermark advancement.

| Field | Required | Default or omission behavior | Rule |
| --- | ---: | --- | --- |
| `feed_profile_id` | Yes | none | Stable profile identifier scoped to `020`; must not encode a private source route. |
| `feed_category` | Yes | none | Closed category token from `LakehouseFeedCategoryClosureRequirementTable`; unknown tokens fail closed until an active category closure row set and validation rows exist. |
| `source_category` | Yes | none | Vendor-neutral source category; private vendor/product names are forbidden in public profile bytes. |
| `source_dataset` | Yes | none | Vendor-neutral dataset token used by `060` authority, coverage, staleness, and completeness rows. |
| `supplier_profile_ref` | Yes | none | `030.ActivationControlledArtifactRef` with `artifact_class = raw_supplier_profile`. |
| `read_target_kind` | Yes | none | One of `table_snapshot`, `dataset_version`, `object_batch`, `partition_set`, or `manifest_list`; omission fails with `LAKEHOUSE_FEED_PROFILE_SCHEMA_INCOMPLETE`. |
| `read_policy_ref` | Yes | none | Active `030.ActivationControlledArtifactRef` with `artifact_class = lakehouse_read_policy`. |
| `scope_key_schema` | Yes | none | Canonical JSON object declaring scope keys, type, bounds, and redaction behavior. Empty scope key schema is forbidden for production feeds. |
| `absence_sensitive_domains` | No | `[]` | Canonically sorted domain tokens. Empty means the feed may preserve positive observations but must not support absence-sensitive effects. |
| `partial_read_policy` | No | `positive_records_only` | Closed enum: `reject_read`, `positive_records_only`, `declared_subset_required`. The default permits positive raw import from known partial reads only and forbids absence-sensitive effects. |
| `empty_scope_policy` | No | `not_authoritative_for_absence` | Closed enum: `not_authoritative_for_absence`, `empty_complete_requires_060`, `reject_empty_scope`. |
| `availability_check_required` | No | `true` | `false` is allowed only for validation fixtures and must not be used for production reads. |
| `availability_refresh_policy` | No | `fail_if_stale` | Closed enum: `fail_if_stale`, `refresh_before_read`; refresh may validate lakehouse artifacts only. |
| `upstream_completeness_required` | No | derived | Materializes to `true` when `absence_sensitive_domains` is non-empty or the category row requires upstream evidence; otherwise `false`. |
| `coverage_profile_refs` | No | `[]` | Refs to active `060.CoverageDimensionProfile` rows required for absence-sensitive domains. Missing required refs block effects. |
| `source_authority_profile_refs` | No | `[]` | Refs to active `060.SourceAuthorityProfileRow` row sets. Missing refs block absence-sensitive effects. |
| `source_staleness_policy_refs` | No | `[]` | Refs to active `060.SourceStalenessPolicy` rows required by category and effect. |
| `parser_mapping_refs` | No | `[]` | Parser and mapping artifact refs required before raw records may advance past import into parsing or normalization. |
| `fixture_refs` | Yes | none | Non-empty refs to `120.LakehouseFeedFixture` rows covering the category and target kind. |
| `validation_refs` | Yes | none | Non-empty refs to passing validation rows for profile schema, branch behavior, category closure, and private-binding leak rejection. |
| `activation_scope` | Yes | none | Vendor-neutral scope in which the profile may affect output. It must not contain concrete tenant inventories, routes, credentials, or private source lists. |

Profile validation must materialize defaults before checksum computation. Unknown fields fail before activation unless the owning profile schema version declares an extension map.

### LakehouseFeedCategoryClosureRow

`LakehouseFeedCategoryClosureRowSet` is an activation-controlled artifact represented by `030.ActivationControlledArtifactRef` with `artifact_class = lakehouse_feed_category_closure_row_set`. The row set instantiates the stable category-closure contract; it must not define new read target kinds, receipt states, authority classes, omission states, or effect tokens.

| Field | Required | Default or omission behavior | Rule |
| --- | ---: | --- | --- |
| `row_id` | Yes | none | Stable row ID scoped to `LakehouseFeedCategoryClosureRowSet`; required for refs, checksums, and validation output. |
| `row_version` | Yes | none | Immutable owner version. |
| `source_dataset_allowlist_ref` | Yes | none | Ref to the vendor-neutral source dataset catalog or deterministic block proof used by the row. If no source-dataset catalog exists, activation fails. |
| `category_activation_state` | Yes | none | Closed enum: `active_for_effects`, `active_positive_only`, `deterministically_blocked`. |
| `feed_category` | Yes | none | Must be one category from `LakehouseFeedCategoryClosureRequirementTable`. |
| `allowed_read_target_kinds` | Yes | none | Non-empty array of closed read target kinds unless `deterministic_block_code` blocks the category. |
| `absence_sensitive_domains` | No | `[]` | Domains whose negative or not-observed outputs require `060` completeness, authority, coverage, and staleness gates. |
| `required_upstream_evidence_classes` | No | `[]` | Supplier evidence classes that must be present before `060` may evaluate effects. |
| `required_coverage_domains` | No | `[]` | `060.CoverageDimensionProfile` domains required for effects. |
| `required_authority_refs` | No | `[]` | Exact `060.SourceAuthorityProfileRow` refs or row-set refs required by the category. |
| `required_staleness_refs` | No | `[]` | Exact `060.SourceStalenessPolicy` refs required by the category. |
| `allowed_effects` | No | `[]` | Closed effect tokens imported from `060`: `absence`, `cleanup`, `retraction`, `graph_expiry`, `watermark`. Empty means positive observations only. |
| `blocked_effects` | No | all effects not in `allowed_effects` | Map from blocked effect token to deterministic block code. Missing map entries fail closure validation. |
| `source_authority_closure_matrix_ref` | Required when any absence-sensitive effect is allowed | null only for positive-only or category-blocked rows | Ref to the active `060.SourceAuthorityClosureMatrixRowSet` or validation result for this category, effect, and scope. |
| `closure_row_checksum` | Yes | none | SHA-256 over the canonical row bytes after defaults materialize. |
| `required_060_closure_refs` | Required when `allowed_effects` is non-empty | `[]` only when the row permits positive observations only | Canonically sorted refs to required `060` row-set artifacts for completeness, authority, coverage, staleness, progress signals, control results, source history, absence policy, and watermark policy. |
| `effect_closure_requirements` | Yes | none | Total map over `absence`, `cleanup`, `retraction`, `graph_expiry`, and `watermark`. For each allowed effect, the row must name exact required `060` artifact classes, row-set refs, validation row IDs, and manifest inclusion requirements. For each blocked effect, the row must name a deterministic block code and mutation-prohibition validation ref. |
| `visibility_profile_refs` | Required when category depends on permissions or hidden objects | `[]` | Exact `060.SupplierCollectionVisibilityProfile` refs required for permission-sensitive absence. |
| `control_result_mapping_required` | Required for `control_evaluation` | `false` for all other categories | `true` requires an active `060.ControlResultMappingRowSet` before control output. |
| `source_history_retention_required` | Required for `source_history` and history-backed cloud inventory | `false` | `true` requires `060.SourceHistoryRetentionProfile`. |
| `default_missing_row_result` | Yes | none | Deterministic result when a required upstream object, partition, row, or source-scope record is missing. It must not be `absence` by default. |
| `deterministic_block_code` | Required when category is blocked | null | Error or no-op code used when the category exists but is intentionally blocked for MVP. |
| `validation_refs` | Yes | none | Non-empty refs to category-specific positive and negative validation rows. |

A missing active row for a known feed category fails with `LAKEHOUSE_FEED_CATEGORY_ROW_MISSING`. A known category that product governance removes from MVP must still have an active closure row with `allowed_effects = []`, a deterministic block code, and validation refs.

`020.allowed_effects` is necessary but not sufficient. It permits `060` to evaluate the effect; it must not authorize the effect without exact active `060` closure rows.

`effect_closure_requirements` must be total for every closed effect token. Missing allowed-effect refs, missing blocked-effect codes, missing mutation-prohibition refs, or missing manifest inclusion rules fail closure validation and must not be interpreted as positive-only behavior.

### Feed category to source-authority closure handoff

A feed category with non-empty `absence_sensitive_domains` must not enter active production scope unless each absence-sensitive effect has exactly one of these closure paths:

| Closure path | Required behavior |
| --- | --- |
| validated closure row chain | The category names a validated `060.SourceAuthorityClosureMatrix` row chain for the effect, including authority, completeness, coverage, staleness, progress-signal, control-result, history, absence, and watermark artifacts when applicable. |
| deterministic block row | The category has `allowed_effects = []`, a deterministic block code, validation refs, and mutation-prohibition evidence proving no absence-sensitive effect can occur. |

Missing source-authority closure rows must block absence, cleanup, retraction, graph expiry, and watermark advancement. Missing rows must not be interpreted as permission to use broad source-category authority, default lakehouse read completeness, provider freshness, destination cleanup, graph derived-view state, or implementation-local policy.

### MVPFeedCategoryClosureCatalogCompleteness

The active `LakehouseFeedCategoryClosureRowSet` must contain exactly one row for every category in `LakehouseFeedCategoryClosureRequirementTable`.

| Condition | Required behavior |
| --- | --- |
| Known category row missing | Fail with `LAKEHOUSE_FEED_CATEGORY_ROW_MISSING` before feed read/import. |
| Known category row duplicated | Fail with the owner duplicate-row validation code before feed read/import. |
| Category outside MVP but known to the table | Use `category_activation_state = deterministically_blocked` or `category_activation_state = active_positive_only` with all absence-sensitive effects blocked. |
| Missing row for any known category | Must not mean positive-only, absence, cleanup, graph expiry, retraction, or watermark permission. |
| Deterministic block row selected | Emit no raw-import side effect beyond owner-declared diagnostics and mutation-prohibition evidence for absence-sensitive effects. |

## Raw Import Contract

`RawRecordImportRun` must supply every field required by `040.RawRecordSchema` and must import lakehouse raw feed rows or objects into deterministic `RawRecord` records that pass `040.ValidateCoreRecord` before commit. The import run must record feed profile ID, read policy ID, feed manifest ID, source dataset, scope keys, payload hash algorithm, import package artifact, input table or object refs, and every input required by `040.ComputeRawRecordId`.

RawRecord ID computation is owned by `040.ComputeRawRecordId`. `020` supplies feed, target, manifest, source dataset, scope, supplier identity, payload hash, and import profile inputs. `020` must not define a competing raw ID input order or collision policy and must import `RAW_RECORD_ID_COLLISION` from `040.CoreRecordErrorCodeSet`.

## Completeness Receipt Contract

`LakehouseReadCompletenessReceipt` proves only that Cadastre read the declared feed target. It is necessary but not sufficient for source-level absence, retraction, cleanup, graph expiry, or watermark advancement.

| Receipt state | Meaning | May authorize absence by itself |
| --- | --- | --- |
| `read_complete` | Cadastre read the declared lakehouse feed target completely. | No. |
| `read_partial_known_gap` | Known partition, object, or row range was not read. | No. |
| `read_partial_unknown_gap` | Feed target may be incomplete but the missing scope is unknown. | No. |
| `read_unavailable` | Declared lakehouse target could not be read. | No. |
| `schema_unavailable` | Required schema ref was unavailable. | No. |
| `manifest_invalid` | Manifest checksum, count, or shape failed validation. | No. |

Supplier evidence must be persisted as `UpstreamCompletenessEvidence` with an explicit `authority_limit`. Supplier evidence is diagnostic-only unless imported by `060` through an active completeness profile.

## Table State Contract

Every authoritative read that can affect production output, replay, graph rebuild, or maintenance eligibility must emit a `LakehouseSnapshotRef` or `DatasetVersionRef`. Every authoritative write attempt, success, failure, rollback, restore, or maintenance commit must emit a `LakehouseCommitRef`.

| Reference | Required purpose |
| --- | --- |
| `LakehouseTableProfile` | Names table format, catalog binding, replay requirement, maintenance policy, schema compatibility policy, and checksum behavior. |
| `LakehouseSnapshotRef` | Names table-format-native snapshot, metadata/log, schema, partition, delete/tombstone, catalog, and checksum identity. |
| `DatasetVersionRef` | Names logical dataset or table-set version consumed or produced by a run. |
| `LakehouseCommitRef` | Names attempted or completed write and its table-format-native commit evidence. |

### EvidenceRef lakehouse artifact handoff

`020` runtime records that are persisted Cadastre records may be referenced by `EvidenceRef.artifact_id.kind = cadastre_record_ref` with `artifact_checksum` equal to the referenced record checksum. Lakehouse external artifacts that are not Cadastre records must use `EvidenceRef.artifact_id.kind = lakehouse_artifact_ref` with `artifact_checksum` computed over immutable bytes or owner-declared canonical metadata bytes.

| Lakehouse artifact or record | Required `EvidenceRef.artifact_id.kind` | Required checksum basis |
| --- | --- | --- |
| `RawFeedManifest` persisted as a Cadastre record | `cadastre_record_ref` | Referenced record checksum. |
| `RawFeedManifest` referenced manifest or object bytes | `lakehouse_artifact_ref` | Immutable manifest or object bytes, or owner-declared canonical metadata bytes. |
| `LakehouseSnapshotRef` | `cadastre_record_ref` | Referenced record checksum. |
| `DatasetVersionRef` | `cadastre_record_ref` | Referenced record checksum. |
| `LakehouseCommitRef` | `cadastre_record_ref` | Referenced record checksum. |
| `LakehouseReadCompletenessReceipt` | `cadastre_record_ref` | Referenced record checksum. |
| `UpstreamCompletenessEvidence` | `cadastre_record_ref` | Referenced record checksum. |
| Raw object ref inside `RawFeedManifest` | `lakehouse_artifact_ref` | Immutable object bytes or owner-declared canonical metadata bytes. |
| Partition ref inside `RawFeedManifest` | `lakehouse_artifact_ref` | Immutable partition manifest bytes or owner-declared canonical metadata bytes. |

Raw payload bytes must not be inlined into `EvidenceRef`. Object-store path alone is not sufficient evidence identity; `artifact_checksum` is mandatory. Mutable table refs such as `latest`, branch name alone, unpinned catalog ref, unresolved object prefix, and unpinned partition prefix must not satisfy `EvidenceRef`. Output-affecting reads and writes must still be represented by `LakehouseSnapshotRef`, `DatasetVersionRef`, or `LakehouseCommitRef`.

## Feed and Table Volatility Classification

Feed and table contracts split stable behavior from activatable profiles and runtime state.

| Artifact or record | Volatility class | Authority class | Required production handling |
| --- | --- | --- | --- |
| `RawSupplierProfile` | `activation_controlled_artifact` | supporting evidence | Must be active before dependent feed profiles execute. |
| `LakehouseFeedProfile` | `activation_controlled_artifact` | supporting evidence | Must be referenced by `030.ActivationControlledArtifactRef`. |
| `LakehouseFeedProfileSchema` | `stable_core_contract` | runtime data input | Owned by `020`; package or profile rows must not redefine it. |
| `LakehouseFeedCategoryClosureRow` | `activation_controlled_artifact` | supporting evidence | Row inside an active `LakehouseFeedCategoryClosureRowSet`; must instantiate the stable closure schema only. |
| `LakehouseFeedCategoryClosureRowSet` | `activation_controlled_artifact` | supporting evidence | Must be active before category-dependent feed reads or absence-sensitive effects. |
| `LakehouseFeedAvailabilityCheck` | `runtime_state_record` | supporting evidence | Must be recorded when required by the active profile. |
| `LakehouseFeedFeasibilityAssessment` | `runtime_state_record` | activation evidence | Must record activation readiness or deterministic blocking reasons for the feed profile and category row. |
| `RawFeedManifest` | `runtime_state_record` | supporting evidence | Must be included in `VersionManifest` when output-affecting. |
| `LakehouseReadPolicy` | `activation_controlled_artifact` | supporting evidence | Must be active and manifest-recorded before read. |
| `RawRecordImportRun` | `runtime_state_record` | supporting evidence | Must record deterministic import inputs and refs. |
| `LakehouseReadCompletenessReceipt` | `runtime_state_record` | supporting evidence | Necessary but not sufficient for absence, cleanup, retraction, graph expiry, or watermark. |
| `UpstreamCompletenessEvidence` | `runtime_state_record` | supporting evidence | May be consumed only through `060` policy. |
| `LakehouseTableProfile` | `activation_controlled_artifact` | system of record table governance | Must be active before production table read/write. |
| `LakehouseSnapshotRef` | `runtime_state_record` | table-state evidence | Must identify table-format-native read state. |
| `DatasetVersionRef` | `runtime_state_record` | table-state evidence | Must identify logical dataset or table-set version. |
| `LakehouseCommitRef` | `runtime_state_record` | table-state evidence | Must identify attempted or completed write state. |
| `ReplayRetentionPolicy` | `activation_controlled_artifact` | replay governance | Must be active before retention-sensitive maintenance. |
| `TableMaintenancePolicy` | `activation_controlled_artifact` | maintenance governance | Must be active before destructive or rewrite maintenance. |
| `ReplayRetentionDecision` | `runtime_state_record` | maintenance evidence | Must be persisted for each maintenance candidate set. |
| `CrossTableCommitProfile` | `activation_controlled_artifact` | table-set governance | Must be active when coherent table-set reads are required. |
| `CatalogBranchPromotionPolicy` | `activation_controlled_artifact` | catalog promotion governance | Must be active when catalog versioning controls production visibility. |

Production feed reads and table maintenance decisions must fail closed when any required activation-controlled artifact is inactive, checksum-mismatched, outside activation scope, missing validation refs, or omitted from `VersionManifest`.

## Maintenance Safety

`TableMaintenancePolicy` must evaluate snapshot expiration, vacuum, cleaner, orphan deletion, checkpoint or transaction-log cleanup, object garbage collection, restore, rollback, compaction deletion, and catalog garbage collection before execution.

`ReplayRetentionDecision` must refuse destructive maintenance when candidate deletion or rewrite would invalidate any protected production `VersionManifest`, graph rebuild, replay window, legal hold, or retention policy.

## Feed Read Algorithm

```text
ReadLakehouseFeed(profile, read_policy, target_ref):
1. Validate `030.ActivationControlledArtifactRef` for `LakehouseFeedProfile`, `RawSupplierProfile`, `LakehouseReadPolicy`, `LakehouseFeedCategoryClosureRowSet`, and every required table, retention, maintenance, cross-table, or catalog policy artifact.
2. Validate `profile` against `LakehouseFeedProfileSchema` after default materialization.
3. Fail with `LAKEHOUSE_FEED_PROFILE_SCHEMA_INCOMPLETE` when any required profile field, including `read_target_kind`, is omitted or invalid.
4. Resolve exactly one active `LakehouseFeedCategoryClosureRow` for `profile.feed_category` and fail with `LAKEHOUSE_FEED_CATEGORY_ROW_MISSING` when no row exists.
5. Fail with the row's `deterministic_block_code` when the category row is intentionally blocked for MVP.
6. Fail with `LAKEHOUSE_PROFILE_BRANCH_UNRESOLVED` when a branch depends on an unmaterialized profile field, missing closure row field, missing validation ref, or missing owner-defined result.
7. Verify no direct enterprise source endpoint, private route, credential, or environment-specific source list is referenced.
8. Verify `LakehouseFeedAvailabilityCheck` is current when `availability_check_required = true`; if stale, apply `availability_refresh_policy`.
9. Validate `target_ref` against `read_policy`, `profile.read_target_kind`, `allowed_read_target_kinds`, and checksum rules.
10. For a declared subset read, validate an active `030.DeclaredDAGSubsetProfile`; otherwise fail with `LAKEHOUSE_DECLARED_SUBSET_REQUIRED`.
11. Read only declared table snapshots, dataset versions, objects, partitions, or manifests.
12. Map every missing, unreadable, omitted, empty, or invalid target condition through `ReadTargetBranchPolicy`.
13. Persist read checkpoints as `StageStateRecord` when output-affecting.
14. Persist `RawRecordImportRun` inputs.
15. Persist only positive `RawRecord` rows that pass `040.ValidateCoreRecord` and are permitted by `partial_read_policy`; otherwise persist deterministic import errors.
16. Persist `LakehouseReadCompletenessReceipt` with the branch-selected receipt state.
17. Persist `VersionManifest` refs for every output-affecting activation artifact, profile, policy, target, schema, state, manifest, feasibility assessment, and category closure row set.
18. Return no absence, cleanup, retraction, graph expiry, or watermark result from this algorithm.
```

### FeedStageLifecycleEventDerivation

`FeedStageLifecycleEventDerivation` maps feed-read branch outcomes into `030.StageExecutionLifecycleMachine.v1` events. It must not restate the generic stage lifecycle transition matrix. Every feed-read branch must emit a `030.ProcessingStageLifecycleResult` or fail before production output.

| Feed read outcome | Lifecycle event | Terminal state expectation | Required side effects |
| --- | --- | --- | --- |
| Profile schema missing or invalid | `nonretryable_error` | `failed_nonisolated` unless stage isolation permits `failed_isolated` | No read, no raw import, no watermark. |
| No active category closure row | `nonretryable_error` | `failed_nonisolated` unless stage isolation permits `failed_isolated` | No read, no raw import. |
| Category closure deterministic no-op block | `stage_no_output` | `no_op` | Emit block/no-op evidence only. |
| Category closure deterministic error block | `nonretryable_error` | `failed_nonisolated` unless isolated | Emit owner error; no production output. |
| Branch unresolved | `nonretryable_error` | `failed_nonisolated` unless isolated | No read. |
| Declared subset missing | `nonretryable_error` | `failed_nonisolated` unless isolated | No subset output. |
| Declared subset output forbidden | `forbidden_output` | `failed_nonisolated` | No output commit. |
| Missing table, dataset, object, partition, or schema refs before read | `nonretryable_error` unless read policy declares the condition retryable | `failed_nonisolated` or `retry_wait` | No raw import until retry or success. |
| Manifest invalid | `nonretryable_error` | `failed_nonisolated` unless isolated | Receipt or diagnostic only; no raw import commit. |
| Partial known gap with positive records allowed | `stage_success` | `succeeded` | Persist raw positives, receipt, and `blocked_effects = [absence, cleanup, retraction, graph_expiry, watermark]`. |
| Partial unknown gap with positive records allowed | `stage_success` | `succeeded` | Persist raw positives, receipt, and `blocked_effects = [absence, cleanup, retraction, graph_expiry, watermark]`. |
| Empty target scope under `not_authoritative_for_absence` | `stage_success` | `succeeded` | Persist receipt and diagnostic; absence and watermark remain blocked. |
| Empty target scope under `reject_empty_scope` | `nonretryable_error` | `failed_nonisolated` unless isolated | No read output. |
| Successful full read | `stage_success` | `succeeded` | Persist raw rows or manifest validation, receipt, and state records. |
| Successful manifest-list validation that produces no raw rows | `stage_no_output` or `stage_success` as profile declares | `no_op` or `succeeded` | Persist receipt and validation refs; no absence-sensitive effects unless `060` later permits them. |

Feed-related `030.ProcessingStageLifecycleResult` usage must add `feed_receipt_state_ref`. The field must reference the persisted `LakehouseReadCompletenessReceipt` when the feed stage reaches `succeeded`, `no_op`, or an isolated failure after receipt emission.

`020` remains non-authoritative for absence, cleanup, retraction, graph expiry, and watermark authorization. Those effects may be unblocked only by `060` after the feed stage result records the blocked effects.

## Feed and Table State Contract Details

### LakehouseFeedFeasibilityAssessment

`LakehouseFeedFeasibilityAssessment` is exported by this spec. It is a runtime state record that proves activation readiness or records deterministic blocking reasons. A feed may become active only when the assessment exists, references the active feed profile and category closure row, and has `activation_result = pass`; otherwise activation is blocked.

| Field | Required | Default or omission behavior | Rule |
| --- | ---: | --- | --- |
| `assessment_id` | Yes | none | Deterministic ID over feed profile ref, category row ref, policy refs, readiness statuses, and blocking reasons. |
| `feed_profile_ref` | Yes | none | Active `LakehouseFeedProfile` ref. |
| `feed_category_row_ref` | Yes | none | Active `LakehouseFeedCategoryClosureRow` ref from the active row set. |
| `read_policy_ref` | Yes | none | Active `LakehouseReadPolicy` ref. |
| `payload_fidelity_status` | Yes | none | Closed token: `pass`, `blocked`, `not_applicable`. |
| `metadata_sufficiency_status` | Yes | none | Must pass when required target refs, schema refs, manifest refs, and supplier lineage are required. |
| `timestamp_sufficiency_status` | Yes | none | Must pass before downstream temporal processing; `unknown` is not a pass. |
| `identifier_sufficiency_status` | Yes | none | Must pass when raw records can affect identity, target selectors, graph projection, or source authority. |
| `replayability_status` | Yes | none | Must pass when target refs, object refs, state refs, and manifest refs are retained for replay. |
| `fixture_coverage_status` | Yes | none | Must pass when `fixture_refs` cover category, target kind, branch behavior, and private-binding leak cases. |
| `completeness_evidence_status` | Yes | none | Must pass for absence-sensitive domains and may be `not_applicable` only when `absence_sensitive_domains = []`. |
| `parser_mapping_readiness_status` | Yes | none | Must pass before parser, mapping, normalization, or downstream output stages can use the feed. |
| `activation_result` | Yes | none | `pass` only when every required status is `pass` or explicitly `not_applicable`; otherwise `blocked_until_all_pass`. |
| `blocking_reasons` | Yes | `[]` | Required non-empty when `activation_result != pass`; sorted by blocking code then artifact ref. |

`activation_result = pass` is forbidden when any required dimension is omitted, `blocked`, `unknown`, stale, checksum-mismatched, outside activation scope, or missing validation refs.

### RawSupplierProfile

| Supplier class | Public/private status | Allowed metadata | Prohibited private fields | Validation behavior |
| --- | --- | --- | --- | --- |
| `external_transport_supplier` | public vendor-neutral class | supplier class, feed scope, delivery batch ref, redacted access-ref hash | concrete vendor route, credential, tenant host list | Reject private values with `PRIVATE_BINDING_LEAK`. |
| `lakehouse_export_supplier` | public vendor-neutral class | export window, object refs, schema refs, upstream completeness refs | source credentials, source API URLs | Require manifest validation. |
| `validation_fixture_supplier` | public redacted class | fixture ID, source category, redaction summary, checksum | raw private payload unless redacted fixture permits | Validation-only, no production evidence. |
| `private_bound_supplier` | private implementation artifact only | none in public docs | all concrete bindings | Public artifact must fail if present. |

### ReadTargetBranchPolicy

The read target branch table is total for production feed reads. A branch not covered by this table must fail with `LAKEHOUSE_PROFILE_BRANCH_UNRESOLVED` before read, import, completeness evaluation, absence evaluation, cleanup, retraction, graph expiry, or watermark advancement.

| Condition | Required receipt state | Required error or no-op | Output classes | VersionManifest requirement |
| --- | --- | --- | --- | --- |
| `read_target_kind` omitted or outside the closed enum | none | `LAKEHOUSE_FEED_PROFILE_SCHEMA_INCOMPLETE` | none | rejected profile validation refs only |
| `feed_category` has no active closure row | none | `LAKEHOUSE_FEED_CATEGORY_ROW_MISSING` | none | rejected profile and row-set refs |
| Category closure row has `deterministic_block_code` | none | row-defined deterministic block/no-op code | none | category row ref and validation refs |
| `table_snapshot` missing `LakehouseSnapshotRef`, schema ref, or required table profile ref | `read_unavailable` or `schema_unavailable` as applicable | hard read failure | receipt and diagnostics only | failed target refs and read policy ref |
| `dataset_version` missing `DatasetVersionRef`, schema refs, or table-set checksum | `read_unavailable` or `schema_unavailable` as applicable | hard read failure | receipt and diagnostics only | failed target refs and read policy ref |
| Manifest checksum, count, object shape, partition shape, or schema ref invalid | `manifest_invalid` | `manifest_invalid` | manifest validation result and receipt only | manifest ref and validation error refs |
| Manifest-listed object or partition is unreadable after retry and its identity is known | `read_partial_known_gap` | no absence, cleanup, retraction, graph expiry, or watermark | positive raw records only when `partial_read_policy = positive_records_only` | missing object or partition ref and checkpoint refs |
| Missing scope is suspected but the missing object, partition, row range, or table region cannot be identified | `read_partial_unknown_gap` | no absence, cleanup, retraction, graph expiry, or watermark | diagnostics and any already committed positive raw records permitted by policy | partial state and blocking reason refs |
| Declared subset requested without active `030.DeclaredDAGSubsetProfile` | none | `LAKEHOUSE_DECLARED_SUBSET_REQUIRED` | none | subset request ref and rejected profile refs |
| Declared subset emits an output forbidden by `030.DeclaredDAGSubsetProfile` | none | `030.DECLARED_DAG_SUBSET_OUTPUT_FORBIDDEN` | none | subset profile ref and stage output refs |
| Empty target scope with `empty_scope_policy = not_authoritative_for_absence` | `read_complete` | `LAKEHOUSE_EMPTY_SCOPE_NOT_AUTHORITATIVE` | receipt and diagnostics; no absence-sensitive effects | empty scope evidence and profile ref |
| Empty target scope with `empty_scope_policy = empty_complete_requires_060` | `read_complete` | no feed-read error; `060` must decide effects | receipt only | upstream evidence refs and completeness profile refs |
| Empty target scope with `empty_scope_policy = reject_empty_scope` | none | hard read failure | diagnostics only | profile and target refs |
| Successful full read of declared target | `read_complete` | none | raw rows or manifest validation, receipt, state records | all target, manifest, policy, category, and state refs |

### LakehouseFeedCategoryClosureRequirementTable

This table defines the MVP feed-category closure catalog. It does not activate a category row by itself. Each non-blocked category must have exactly one active `LakehouseFeedCategoryClosureRow` in the active row set before production use. Unknown or future categories fail closed until an active row set, `060` completeness rows, and `120` validation rows exist.

| Feed category | Allowed read target kinds in active row | Absence-sensitive uses | Required upstream evidence classes | Required `060` coverage domains | Default missing-row behavior | MVP result |
| --- | --- | --- | --- | --- | --- | --- |
| `endpoint_inventory` | Any closed read target kind when the active row permits it. | Endpoint non-observation, cleanup, graph expiry, watermark. | scope enumeration, permission visibility, collection window, failed-scope evidence. | endpoint | `not_authoritative_for_absence` | Requires active closure row. |
| `configuration_inventory` | Any closed read target kind when the active row permits it. | Configuration absence, stale configuration, cleanup, watermark. | configuration scope, collection window, failed-item evidence, source permission state. | endpoint, cloud inventory, or control as row declares | `not_authoritative_for_absence` | Requires active closure row. |
| `vulnerability_scan` | Any closed read target kind when the active row permits it. | Vulnerability absence, fixed-state absence, scan watermark. | scanner target scope, credential/auth status, plugin/check set, scan window, failed-target and exclusion evidence. | vulnerability | `not_authoritative_for_absence` | Requires active closure row. |
| `control_evaluation` | Any closed read target kind when the active row permits it. | Control pass, fail, unknown, not checked, not applicable, watermark. | benchmark/check scope, applicability evidence, evaluation status, result mapping refs. | control | `not_authoritative_for_absence` | Requires active closure row. |
| `directory_inventory` | Any closed read target kind when the active row permits it. | Principal, group, device, and directory-object absence. | tenant/domain scope, page or delta completion, hidden-object permission state, failed-scope evidence. | directory | `not_authoritative_for_absence` | Requires active closure row. |
| `directory_membership` | Any closed read target kind when the active row permits it. | Group non-membership, user non-membership, membership cleanup, watermark. | tenant/domain, group, member type, direct/transitive mode, hidden-membership permission, page/delta completion, AD primary-group evidence. | directory | `not_authoritative_for_absence` | Requires active closure row. |
| `dns_record_set` | Any closed read target kind when the active row permits it. | DNS absence and stale DNS state. | zone/source scope, authoritative source evidence, TTL basis, collection window. | DNS | `not_authoritative_for_absence` | Requires active closure row. |
| `dhcp_ipam_assignment` | Any closed read target kind when the active row permits it. | Lease absence, assignment absence, host cleanup, watermark. | DHCP/IPAM scope, lease window, authoritative-system evidence, failed-scope evidence. | DHCP/IPAM | `not_authoritative_for_absence` | Requires active closure row. |
| `network_flow` | Any closed read target kind when the active row permits it. | Positive observed-flow evidence only by default. | sensor scope, collection point, time window, role evidence, packet/flow field completeness. | flow | `unknown`; missing flow must not imply no flow. | Requires active closure row; absence effects default blocked. |
| `cloud_asset_inventory` | Any closed read target kind when the active row permits it. | Cloud resource absence, deletion inference, graph expiry, watermark. | account/project/subscription, region, resource type, permission visibility, source-history evidence when required. | cloud inventory | `not_authoritative_for_absence` | Requires active closure row. |
| `source_history` | Any closed read target kind when the active row permits it. | No-change proof only within supported source-native history window. | history window, retention profile, query scope, outside-window evidence. | source history | `unknown`; outside-window no-result is not proof. | Requires active closure row. |
| `future_reachability` | none | none in MVP. | inactive deferred reachability evidence only. | deferred reachability | deterministic no-op/block. | Blocked for MVP with `REACHABILITY_DEFERRED_OUTPUT_FORBIDDEN` or owner no-op. |

### FeedCategoryToSourceAuthorityClosureRequirements

This table maps every feed category in `LakehouseFeedCategoryClosureRequirementTable` to the `060` artifact classes required before an absence-sensitive effect may execute. The table contains no concrete vendor, product, tenant, route, scanner-site, zone, account, or private inventory rows.

| Feed category | Required `060` artifact classes before absence-sensitive effects | Required `120` validation row family | Additional condition |
| --- | --- | --- | --- |
| `endpoint_inventory` | `LakehouseFeedCompletenessProfileRowSet`, `SourceAuthorityProfileRowSet`, `CoverageDimensionProfile`, `SourceStalenessPolicy`, `ProgressSignalInterpretationPolicy`, `AbsenceDerivationPolicy`, `ProjectionWatermarkPolicy` | `120-SOURCE-CLOSURE-*` | `SupplierCollectionVisibilityProfile` is required when enrollment or inventory visibility is permission-limited. |
| `configuration_inventory` | `LakehouseFeedCompletenessProfileRowSet`, `SourceAuthorityProfileRowSet`, `CoverageDimensionProfile`, `SourceStalenessPolicy`, `ProgressSignalInterpretationPolicy`, `AbsenceDerivationPolicy`, `ProjectionWatermarkPolicy` | `120-SOURCE-CLOSURE-*` | Coverage domain is selected by the active row for the affected configuration fact. |
| `vulnerability_scan` | `LakehouseFeedCompletenessProfileRowSet`, `SourceAuthorityProfileRowSet`, `CoverageDimensionProfile`, `SourceStalenessPolicy`, `ProgressSignalInterpretationPolicy`, `AbsenceDerivationPolicy`, `ProjectionWatermarkPolicy` | `120-SOURCE-CLOSURE-*` | Scanner target, credential, plugin/check, and scan-window coverage must resolve through `060`. |
| `control_evaluation` | `LakehouseFeedCompletenessProfileRowSet`, `SourceAuthorityProfileRowSet`, `CoverageDimensionProfile`, `SourceStalenessPolicy`, `ControlResultMappingRowSet`, `ProgressSignalInterpretationPolicy`, `AbsenceDerivationPolicy`, `ProjectionWatermarkPolicy` | `120-SOURCE-CLOSURE-*` | Pass, fail, unknown, not checked, and not applicable output require control-result mapping. |
| `directory_inventory` | `LakehouseFeedCompletenessProfileRowSet`, `SourceAuthorityProfileRowSet`, `CoverageDimensionProfile`, `SourceStalenessPolicy`, `SupplierCollectionVisibilityProfile`, `ProgressSignalInterpretationPolicy`, `AbsenceDerivationPolicy`, `ProjectionWatermarkPolicy` | `120-SOURCE-CLOSURE-*` | Hidden-object and limited-information states block negative output unless exact rows authorize. |
| `directory_membership` | `LakehouseFeedCompletenessProfileRowSet`, `SourceAuthorityProfileRowSet`, `CoverageDimensionProfile`, `SourceStalenessPolicy`, `SupplierCollectionVisibilityProfile`, `ProgressSignalInterpretationPolicy`, `AbsenceDerivationPolicy`, `ProjectionWatermarkPolicy` | `120-SOURCE-CLOSURE-*` | Direct/transitive mode, hidden membership, page completion, delta reset, and AD primary-group handling must resolve through exact rows. |
| `dns_record_set` | `LakehouseFeedCompletenessProfileRowSet`, `SourceAuthorityProfileRowSet`, `CoverageDimensionProfile`, `SourceStalenessPolicy`, `ProgressSignalInterpretationPolicy`, `AbsenceDerivationPolicy`, `ProjectionWatermarkPolicy` | `120-SOURCE-CLOSURE-*` | TTL expiry maps to stale or unknown unless exact rows authorize a narrower effect. |
| `dhcp_ipam_assignment` | `LakehouseFeedCompletenessProfileRowSet`, `SourceAuthorityProfileRowSet`, `CoverageDimensionProfile`, `SourceStalenessPolicy`, `ProgressSignalInterpretationPolicy`, `AbsenceDerivationPolicy`, `ProjectionWatermarkPolicy` | `120-SOURCE-CLOSURE-*` | Lease expiry maps to stale or expired assignment state, not host absence, unless exact rows authorize. |
| `network_flow` | `LakehouseFeedCompletenessProfileRowSet`, `SourceAuthorityProfileRowSet`, `CoverageDimensionProfile`, `SourceStalenessPolicy`, `ProgressSignalInterpretationPolicy`, `AbsenceDerivationPolicy`, `ProjectionWatermarkPolicy` | `120-SOURCE-CLOSURE-*` | Missing flow defaults to `unknown`; `watermark` may be allowed only by exact rows, and absence effects remain blocked unless exact rows authorize a future narrower effect. |
| `cloud_asset_inventory` | `LakehouseFeedCompletenessProfileRowSet`, `SourceAuthorityProfileRowSet`, `CoverageDimensionProfile`, `SourceStalenessPolicy`, `SupplierCollectionVisibilityProfile`, `ProgressSignalInterpretationPolicy`, `AbsenceDerivationPolicy`, `ProjectionWatermarkPolicy` | `120-SOURCE-CLOSURE-*` | `SourceHistoryRetentionProfile` is required when source-history no-change or disappearance is consulted. |
| `source_history` | `LakehouseFeedCompletenessProfileRowSet`, `SourceAuthorityProfileRowSet`, `CoverageDimensionProfile`, `SourceStalenessPolicy`, `SourceHistoryRetentionProfile`, `ProgressSignalInterpretationPolicy`, `AbsenceDerivationPolicy`, `ProjectionWatermarkPolicy` | `120-SOURCE-CLOSURE-*` | Outside-window no-result must not become no-change proof; no-change proof requires source-history coverage plus retention. |
| `future_reachability` | deterministic block row only | `120-SOURCE-CLOSURE-*` deterministic block row only | MVP behavior is deterministic no-op or deferred reachability error; activation of any read target or absence-sensitive effect is prohibited. |

### LakehouseReadPolicy payload limits

| Field | Required behavior | Default | Bounds |
| --- | --- | --- | --- |
| `max_payload_byte_length` | Maximum raw payload bytes a read/import step may accept for one `RawRecord`. | `104857600` bytes | Minimum `1`; maximum `1073741824`; feed profile may set a lower bound. |
| `raw_byte_canonicalization_mode` | Byte handling before `canonical_payload_hash` computation. | `none` | Closed enum: `none`; future modes require new schema version and fixtures. |
| `payload_ref` | Must reference declared table, object, partition, manifest, or dataset inputs. | none | Must not contain credentials, concrete private routes, or raw payload bytes. |

Raw payload bytes remain in raw/bronze lakehouse storage. `RawRecord` persists bounded metadata, refs, byte lengths, checksums, visibility state, and lineage only.

### RawFeedManifest schema

| Field | Type | Required | Default | Rule |
| --- | --- | ---: | --- | --- |
| `manifest_id` | string | Yes | none | Deterministic ID from canonical manifest bytes. |
| `feed_profile_id` | string | Yes | none | Must reference active `LakehouseFeedProfile`. |
| `supplier_profile_id` | string | Yes | none | Must reference active `RawSupplierProfile`. |
| `read_target_kind` | enum | Yes | none | One of the declared read target kinds. |
| `object_refs` | array | Required for object targets | `[]` | Each row includes ref, byte count, checksum, media type. |
| `partition_refs` | array | Required for partition targets | `[]` | Each row includes partition key, bounds, schema ref, checksum. |
| `schema_refs` | array | Yes | none | Every payload schema used by the manifest. |
| `record_count` | integer | Yes | `0` | Non-negative. |
| `byte_total` | integer | Yes | `0` | Non-negative. |
| `time_bounds` | object | Yes | none | Declared source, supplier, and lakehouse time bounds with quality. |
| `supplier_lineage` | object | Yes | none | Supplier batch, run, and redacted access-reference hashes. |
| `upstream_completeness_refs` | array | No | `[]` | References only; no absence authority by itself. |
| `hash_algorithm` | enum | Yes | `sha256` | Only `sha256` is active for MVP. |
| `manifest_checksum` | string | Yes | none | SHA-256 over canonical manifest bytes excluding this field. |

#### RawFeedManifest nested object schemas

`RawFeedManifest` nested objects must use the following closed shapes before manifest ID or checksum computation. Omitted optional arrays default to `[]`; omitted required nested objects fail with `RAW_FEED_MANIFEST_INVALID`.

##### `object_refs[]`

| Field | Required | Rule |
| --- | ---: | --- |
| `object_ref_id` | Yes | Stable object reference scoped to the manifest; not an object-store credential or private route. |
| `object_uri_hash` | Yes | SHA-256 lowercase hex over the redacted canonical URI reference. |
| `byte_count` | Yes | Non-negative `uint64`. |
| `checksum_algorithm` | Yes | Default and only active value `sha256`. |
| `object_checksum` | Yes | SHA-256 lowercase hex over object bytes after declared compression boundary. |
| `media_type` | Yes | Declared MIME-like media type token or `application/octet-stream`. |
| `compression` | Yes | Closed token; default `none`. |
| `encryption_ref` | No | Omitted when not encrypted; present value must be a redacted reference. |
| `redaction_state` | Yes | Closed token imported from `110` redaction policy. |

##### `partition_refs[]`

| Field | Required | Rule |
| --- | ---: | --- |
| `partition_key` | Yes | Canonical JSON object; keys sorted lexically. |
| `partition_bounds` | Yes | Canonical JSON object describing inclusive/exclusive bounds. |
| `schema_ref_id` | Yes | Must match one `schema_refs[].schema_ref_id`. |
| `partition_checksum` | Yes | SHA-256 lowercase hex over canonical partition inventory or table-format-native partition ref. |
| `row_count` | Yes | Non-negative `uint64`; `0` is valid only when the profile permits empty partitions. |
| `byte_count` | Yes | Non-negative `uint64`. |

##### `schema_refs[]`

| Field | Required | Rule |
| --- | ---: | --- |
| `schema_ref_id` | Yes | Stable manifest-local schema reference. |
| `schema_family` | Yes | Vendor-neutral schema family or `unknown`. |
| `schema_version` | Yes | Exact source schema version or `unknown`. |
| `schema_checksum` | Yes | SHA-256 lowercase hex over schema bytes or canonical schema descriptor. |
| `schema_uri_hash` | Yes | SHA-256 lowercase hex over the redacted schema URI reference. |

##### `time_bounds`

| Field | Required | Rule |
| --- | ---: | --- |
| `source_time_from` | No | RFC3339 UTC or omitted when unavailable. |
| `source_time_to` | No | RFC3339 UTC or omitted when unavailable. |
| `supplier_collection_from` | No | RFC3339 UTC or omitted when unavailable. |
| `supplier_collection_to` | No | RFC3339 UTC or omitted when unavailable. |
| `lakehouse_commit_time` | Yes | RFC3339 UTC table/object commit or manifest materialization time; not fact time by itself. |
| `quality` | Yes | Closed quality token; `unknown` is allowed only when profile permits. |

##### `supplier_lineage`

| Field | Required | Rule |
| --- | ---: | --- |
| `supplier_batch_id` | Yes | Supplier-scoped batch ID or canonical null sentinel when unavailable. |
| `supplier_run_id` | Yes | Supplier-scoped run ID or canonical null sentinel when unavailable. |
| `supplier_profile_id` | Yes | Must match the manifest's `supplier_profile_id`. |
| `redacted_access_ref_hash` | Yes | SHA-256 lowercase hex over the redacted access-reference descriptor. |

### ComputeRawFeedManifestId

```text
ComputeRawFeedManifestId(manifest):
1. Materialize manifest defaults.
2. Exclude `manifest_id` and `manifest_checksum`.
3. Serialize the remaining manifest object with `040.CanonicalJSON`.
4. Return `rfm_` plus SHA-256 lowercase hex over the UTF-8 canonical bytes.
5. If a prior different canonical manifest byte string has the same ID, emit `RAW_FEED_MANIFEST_ID_COLLISION` and commit no manifest or raw import output.
```

`manifest_checksum` must be SHA-256 lowercase hex over `040.CanonicalJSON` bytes for the materialized manifest excluding only `manifest_checksum`. `manifest_id` is included in `manifest_checksum`.

### RawRecord identity import

`020` imports `040.RawRecordSchema`, `040.ComputeRawRecordId`, and `040.CoreRecordValidationAlgorithm`. The former `020` raw ID input-order table is consolidated into `040.CoreRecordIdPolicy`. Raw import fixtures in `120` must compute expected raw IDs through `040.ComputeRawRecordId`.

| Import concern | Required behavior |
| --- | --- |
| ID input source | `RawRecordImportRun` supplies only the inputs named by `040.ComputeRawRecordId`. |
| Validation order | Schema validation and ID/checksum validation happen before raw persistence, absence evaluation, cleanup, or watermark decisions. |
| Collision behavior | `RAW_RECORD_ID_COLLISION` is imported from `040`; `020` must not define a local collision code. |
| Private binding | `payload_ref` and supplier metadata must fail with `PRIVATE_BINDING_LEAK` when they expose private routes, credentials, or tenant host lists. |

### LakehouseFeedAvailabilityFreshnessPolicy

| Target kind | TTL default | Invalidation trigger | Stale behavior |
| --- | ---: | --- | --- |
| `table_snapshot` | `24h` | schema ref change, table profile change, snapshot ref change | fail read with stale availability unless profile permits refresh. |
| `dataset_version` | `24h` | dataset version ref or table-set checksum change | fail read. |
| `object_batch` | `12h` | object list, object checksum, or manifest checksum change | revalidate before read. |
| `partition_set` | `12h` | partition set, schema ref, or manifest checksum change | revalidate before read. |
| `manifest_list` | `6h` | manifest checksum or schema ref change | revalidate before read. |

### CDC-shaped feed metadata mapping

| CDC metadata | Cadastre placement | Authority effect | Validation fixture requirement |
| --- | --- | --- | --- |
| source offset, LSN, or binlog position | `RawRecord.source_event_metadata` and `StageStateRecord` | replay/progress only | positive and offset mismatch cases |
| schema-history ref | `CDCReplayStateContract` | replay sufficiency only | missing and stale schema-history cases |
| tombstone/delete marker | raw event subtype plus temporal/correction input | no retraction without `060` and `080` policy | tombstone non-authority case |
| heartbeat | `StageStateRecord` diagnostic | no fact time, absence, cleanup, or watermark by itself | heartbeat no-authority case |
| transaction metadata | raw metadata and replay ordering input | ordering only if policy permits | out-of-order case |

### Feed-fixture coverage matrix

| Fixture class | Required fixture ID | Required owner validation row | Expected outcome |
| --- | --- | --- | --- |
| activation artifacts | `feed-020-activation-artifacts` | `val-020-activation-artifacts` | Inactive feed profile, stale read policy artifact, table profile checksum mismatch, maintenance policy omitted from `VersionManifest`, and replay retention policy inactive during maintenance fail before output or destructive action. |
| minimal valid object-batch manifest | `feed-020-valid-object-batch-manifest` | `val-020-manifest-valid-object-batch` | Manifest ID and checksum match expected TODO checksum. |
| malformed manifest | `feed-020-malformed-manifest` | `val-020-manifest-malformed` | `manifest_invalid`; no raw import commit. |
| manifest checksum mismatch | `feed-020-manifest-checksum-mismatch` | `val-020-manifest-checksum-mismatch` | `manifest_invalid`; no raw import commit. |
| omitted object for table snapshot | `feed-020-table-omitted-object` | `val-020-table-omitted-object` | Fail read unless declared subset profile permits partial. |
| omitted object for object batch | `feed-020-object-batch-known-gap` | `val-020-object-batch-known-gap` | `read_partial_known_gap`; no absence authority. |
| raw ID replay | `feed-020-raw-id-replay` | `val-020-raw-id-replay` | Expected raw ID computed through `040.ComputeRawRecordId`. |
| raw ID collision | `feed-020-raw-id-collision` | `val-020-raw-id-collision` | `RAW_RECORD_ID_COLLISION`; commit no colliding record. |
| no absence from manifest | `feed-020-no-absence-from-manifest` | `val-020-no-absence-from-manifest` | Missing object or row yields no absence without `060` gates. |
| CDC tombstone | `feed-020-cdc-tombstone-non-authority` | `val-020-cdc-tombstone-non-authority` | No retraction without `060` and `080` policy. |
| no direct source call | `feed-010-direct-source-call` | `neg-010-direct-source-call` | `DIRECT_SOURCE_CALL_FORBIDDEN`. |
| profile schema incomplete | `feed-020-profile-schema-incomplete` | `val-020-feed-profile-schema-incomplete` | `LAKEHOUSE_FEED_PROFILE_SCHEMA_INCOMPLETE`; no read. |
| missing category closure row | `feed-020-category-row-missing` | `val-020-category-row-missing` | `LAKEHOUSE_FEED_CATEGORY_ROW_MISSING`; no read. |
| unresolved profile branch | `feed-020-profile-branch-unresolved` | `val-020-profile-branch-unresolved` | `LAKEHOUSE_PROFILE_BRANCH_UNRESOLVED`; no output. |
| declared subset required | `feed-020-declared-subset-required` | `val-020-declared-subset-required` | `LAKEHOUSE_DECLARED_SUBSET_REQUIRED`; no output. |
| empty scope not authoritative | `feed-020-empty-scope-not-authoritative` | `val-020-empty-scope-not-authoritative` | `LAKEHOUSE_EMPTY_SCOPE_NOT_AUTHORITATIVE`; no absence or watermark. |
| feed category closure catalog | `feed-020-category-closure-*` | `val-020-category-closure-*` | Each MVP category has one positive or deterministic block row and one missing-row negative case. |

### Feed activation artifact errors

| Error code | Emitted when |
| --- | --- |
| `LAKEHOUSE_ACTIVATION_ARTIFACT_MISSING` | A required feed, table, read, retention, maintenance, cross-table, or catalog policy artifact ref is missing. |
| `LAKEHOUSE_ACTIVATION_ARTIFACT_INACTIVE` | A required artifact exists but lifecycle status is not allowed for production execution. |
| `LAKEHOUSE_ACTIVATION_ARTIFACT_CHECKSUM_MISMATCH` | A required artifact checksum differs from the active artifact ref or manifest. |
| `LAKEHOUSE_FEED_PROFILE_SCHEMA_INCOMPLETE` | A feed profile omits a required `LakehouseFeedProfileSchema` field, uses an invalid target kind, or leaves a required branch input unresolved. |
| `LAKEHOUSE_FEED_CATEGORY_ROW_MISSING` | No active `LakehouseFeedCategoryClosureRow` covers the profile's `feed_category`. |
| `LAKEHOUSE_PROFILE_BRANCH_UNRESOLVED` | A feed read branch still depends on unmaterialized profile or closure-row policy. |
| `LAKEHOUSE_DECLARED_SUBSET_REQUIRED` | A declared subset read is requested or required but no active `030.DeclaredDAGSubsetProfile` covers the request. |
| `UPSTREAM_COMPLETENESS_EVIDENCE_REQUIRED` | The profile or category row requires upstream completeness evidence and the manifest or evidence refs omit it. |
| `LAKEHOUSE_EMPTY_SCOPE_NOT_AUTHORITATIVE` | An empty target scope is read under the default empty-scope policy and must not be interpreted as source absence. |

### LakehouseErrorRegistryFragment

This owner fragment feeds `110.GenerateErrorCodeRegistry`. `110` owns the generated caller-visible registry. This table must not render API output by itself. Rows with `TODO:` cells block authoritative promotion and must be resolved by the owning domain before `110-ERROR-REGISTRY-TOTAL-AC-001` can pass.

| error_code | owner_spec | severity | retry_class | caller_visible_fields | audit_visible_fields | redaction_rule | owner_context_schema_ref | fixture_ref |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `LAKEHOUSE_ACTIVATION_ARTIFACT_MISSING` | `020` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `020.LakehouseErrorContext` | `error-registry-020-lakehouse-activation-artifact-missing` |
| `LAKEHOUSE_ACTIVATION_ARTIFACT_INACTIVE` | `020` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `020.LakehouseErrorContext` | `error-registry-020-lakehouse-activation-artifact-inactive` |
| `LAKEHOUSE_ACTIVATION_ARTIFACT_CHECKSUM_MISMATCH` | `020` | `error` | `retry_after_owner_repair` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `020.LakehouseErrorContext` | `error-registry-020-lakehouse-activation-artifact-checksum-mismatch` |
| `LAKEHOUSE_FEED_PROFILE_SCHEMA_INCOMPLETE` | `020` | `error` | `caller_correctable` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `020.LakehouseErrorContext` | `error-registry-020-lakehouse-feed-profile-schema-incomplete` |
| `LAKEHOUSE_FEED_CATEGORY_ROW_MISSING` | `020` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `020.LakehouseErrorContext` | `error-registry-020-lakehouse-feed-category-row-missing` |
| `LAKEHOUSE_PROFILE_BRANCH_UNRESOLVED` | `020` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `020.LakehouseErrorContext` | `error-registry-020-lakehouse-profile-branch-unresolved` |
| `LAKEHOUSE_DECLARED_SUBSET_REQUIRED` | `020` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `020.LakehouseErrorContext` | `error-registry-020-lakehouse-declared-subset-required` |
| `UPSTREAM_COMPLETENESS_EVIDENCE_REQUIRED` | `020` | `blocked` | `retry_after_refresh` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `020.LakehouseErrorContext` | `error-registry-020-upstream-completeness-evidence-required` |
| `LAKEHOUSE_EMPTY_SCOPE_NOT_AUTHORITATIVE` | `020` | `diagnostic` | `none` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `020.LakehouseErrorContext` | `error-registry-020-lakehouse-empty-scope-not-authoritative` |
| `RAW_FEED_MANIFEST_INVALID` | `020` | `error` | `caller_correctable` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `020.LakehouseErrorContext` | `error-registry-020-raw-feed-manifest-invalid` |
| `RAW_FEED_MANIFEST_ID_COLLISION` | `020` | `error` | `retry_after_owner_repair` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `020.LakehouseErrorContext` | `error-registry-020-raw-feed-manifest-id-collision` |

### LakehouseErrorContext

`LakehouseErrorContext` is the owner context schema for `020` feed, manifest, table-state, and upstream-completeness error rows.

| Field | Required | Rule |
| --- | ---: | --- |
| `context_schema_version` | Yes | Immutable `020` context schema version. |
| `owner_spec` | Yes | Must be `020`. |
| `error_code` | Yes | Must match the generated registry row. |
| `failure_class` | Yes | Closed token: `feed_profile`, `category_closure`, `manifest`, `declared_subset`, `upstream_completeness`, `empty_scope`, or `activation_artifact`. |
| `operation` | Yes | Feed read, manifest validation, raw import, declared-subset validation, or table-state operation. |
| `affected_record_type` | Yes | Feed profile, manifest, table-state ref, raw import run, or completeness receipt type. |
| `field_path` | Yes | Exact field path when applicable; null only for artifact-wide failures. |
| `feed_profile_ref` | No | Redacted feed profile ref when consulted. |
| `read_target_kind` | No | Closed read target token when known. |
| `source_dataset` | No | Vendor-neutral dataset token only. |
| `scope_selector_checksum` | No | Checksum of the scope selector; raw selector values are forbidden when private. |
| `manifest_refs` | No | Canonically sorted manifest, snapshot, dataset, or object refs. |
| `required_upstream_evidence_classes` | No | Closed evidence-class tokens required by the category row. |
| `blocking_reason` | Yes | Bounded reason that explains blocked or diagnostic output. |
| `validation_refs` | Yes | Exact `120` fixture refs. |
| `redaction_classes` | Yes | Private routes, credentials, host lists, raw payload bytes, and source-native IDs must map to `always_forbidden`. |

### LakehouseApiStateLabelGuidance

Profile, category, manifest, table-state, and activation errors render through `110` as `error` or health-blocked states. Partial read, declared-subset, manifest, and empty-scope errors must not render as authorized absence. Upstream completeness insufficiency must map through `060` before any absence-sensitive API, export, graph, or watermark effect.

### Acceptance Criteria

| ID | Criterion |
| --- | --- |
| `020-CLEANUP-AC-001` | No banned reference class remains. |
| `020-CLEANUP-AC-002` | `LakehouseReadCompletenessReceipt` remains necessary but not sufficient for source-level absence, retraction, cleanup, graph expiry, or watermark advancement. |
| `020-CLEANUP-AC-003` | Table maintenance remains governed by `ReplayRetentionPolicy`, `TableMaintenancePolicy`, and `ReplayRetentionDecision`. |
| `020-CLEANUP-AC-004` | No direct enterprise source-call behavior is introduced. |
| `020-SCHEMA-PATCH-AC-001` | `020` contains no normative duplicate of `RawRecord` field schema or ID input order. |
| `020-SCHEMA-PATCH-AC-002` | Raw import produces only records that pass `040.ValidateCoreRecord`. |
| `020-SCHEMA-PATCH-AC-003` | `LakehouseReadPolicy` declares max payload byte length and raw byte canonicalization mode. |
| `020-SCHEMA-PATCH-AC-004` | Raw payload refs cannot expose private routes, credentials, or raw payload bytes in public artifacts. |
| `020-DOMAIN-CONSISTENCY-AC-001` | `domain.md` routes raw feed manifest and raw record ID behavior to `020` and `040` without marking them unresolved or restating `040` ID input order. |
| `020-MANIFEST-AC-001` | `ComputeRawFeedManifestId` and `manifest_checksum` produce byte-identical values across replay from the same manifest bytes. |
| `020-VOLATILITY-AC-001` | A production feed read with an inactive `LakehouseFeedProfile` emits no `RawRecord`. |
| `020-VOLATILITY-AC-002` | Identical feed bytes with different active `LakehouseReadPolicy` checksums produce different `VersionManifest` refs or fail before output. |
| `020-VOLATILITY-AC-003` | Table maintenance fails before destructive action when the required policy artifact is inactive or omitted from `VersionManifest`. |
| `020-FEED-CLOSURE-AC-001` | Feed profile activation and production read fail with `LAKEHOUSE_FEED_PROFILE_SCHEMA_INCOMPLETE` when any required profile schema field is omitted, including `read_target_kind`. |
| `020-FEED-CLOSURE-AC-002` | Every active feed category either has an active `LakehouseFeedCategoryClosureRow` with validation refs or a deterministic MVP block row. |
| `020-FEED-CLOSURE-AC-003` | Every profile-dependent feed-read branch resolves to an explicit field, active artifact ref, receipt state, error/no-op, and validation row. |
| `020-FEED-CLOSURE-AC-004` | Omitted or invalid target-kind input never defaults to a table, object, partition, manifest, or dataset read. |
| `020-FEED-CLOSURE-AC-005` | Declared subset reads fail before output unless an active `030.DeclaredDAGSubsetProfile` covers requested outputs and effects. |
| `020-FEED-CLOSURE-AC-006` | Partial known gaps, partial unknown gaps, empty scopes under the default policy, and missing lakehouse rows never authorize absence, cleanup, retraction, graph expiry, or watermark advancement. |
| `020-SOURCE-CLOSURE-AC-001` | Every category row with non-empty `allowed_effects` declares `effect_closure_requirements`. |
| `020-SOURCE-CLOSURE-AC-002` | `allowed_effects` without matching `060` row refs permits no absence-sensitive effect. |
| `020-SOURCE-CLOSURE-AC-003` | Each feed category in `LakehouseFeedCategoryClosureRequirementTable` appears exactly once in `FeedCategoryToSourceAuthorityClosureRequirements`. |
| `020-SOURCE-CLOSURE-AC-004` | The active `LakehouseFeedCategoryClosureRowSet` contains exactly one row for every category in `LakehouseFeedCategoryClosureRequirementTable`. |
| `020-SOURCE-CLOSURE-AC-005` | Every category row's effect closure map is total across `absence`, `cleanup`, `retraction`, `graph_expiry`, and `watermark`. |
| `020-SOURCE-CLOSURE-AC-006` | `source_history` rows require `CoverageDimensionProfile` plus `SourceHistoryRetentionProfile` before no-change proof. |
| `020-SOURCE-CLOSURE-AC-007` | `future_reachability` resolves to a deterministic block row and emits no MVP runtime effect. |
| `020-SOURCE-CLOSURE-HANDOFF-AC-001` | Every active feed category that names an absence-sensitive domain has exact `060` closure refs and `120` validation refs, or a deterministic block row. Missing, ambiguous, inactive, or checksum-mismatched refs fail before read/import output can authorize absence-sensitive effects. |
| `020-EVIDENCE-LAKEHOUSE-AC-001` | `LakehouseSnapshotRef`, `DatasetVersionRef`, `LakehouseCommitRef`, `LakehouseReadCompletenessReceipt`, and `UpstreamCompletenessEvidence` evidence use `cadastre_record_ref` with referenced record checksum. |
| `020-EVIDENCE-LAKEHOUSE-AC-002` | Raw object, manifest, and partition evidence use `lakehouse_artifact_ref` with immutable bytes or owner-declared canonical metadata bytes. |
| `020-EVIDENCE-LAKEHOUSE-AC-003` | Missing artifact checksum, mutable `latest` refs, branch-only refs, unresolved prefixes, and raw payload bytes fail before `EvidenceRef` ID computation. |

| `020-LIFECYCLE-AC-001` | Every feed-read branch maps to exactly one `030.StageExecutionLifecycleMachine.v1` event, terminal-state expectation, and mutation-prohibition rule. |
| `020-LIFECYCLE-AC-002` | Partial known and partial unknown gaps may commit positive raw records and receipts but must record blocked absence, cleanup, retraction, graph-expiry, and watermark effects. |
| `020-LIFECYCLE-AC-003` | Feed lifecycle results for `succeeded`, `no_op`, and isolated receipt-emitting failures contain `feed_receipt_state_ref`. |

## Definition of Done

| ID | Criterion |
| --- | --- |
| `020-AC-001` | Raw import can be replayed from declared feed/table/object refs without enterprise source calls. |
| `020-AC-002` | Missing rows or objects never authorize absence without an imported completeness decision from `060`. |
| `020-AC-003` | Every production read and write has table-format-native refs when it can affect output or replay. |
| `020-AC-004` | Destructive maintenance is refused when any protected manifest, snapshot, replay window, or legal hold would be invalidated. |
| `020-AC-005` | Feed profile activation fails while feed feasibility, raw supplier profiles, read policy catalog, raw manifest schema, or feed fixture coverage is `TODO:` for the target feed. |
| `020-AC-006` | Every active production feed category has an active closure row set, a passing feasibility assessment, fixture refs, validation refs, and deterministic missing-row behavior. |

## Open Questions

Open questions marked `TODO:` block authoritative status for the affected contract. A downstream implementation must not resolve a `TODO:` by inference.

| ID | Question | Blocking scope | Required owner decision | Default until resolved |
| --- | --- | --- | --- | --- |
| `020-TODO-SOURCE-DATASET-CATALOG` | TODO: Provide the vendor-neutral `source_dataset` catalog or deterministic block rows for every `source_dataset` referenced by active feed profiles and source-authority closure rows. | Feed activation, source-authority closure, and validation. | Product governance plus `020`, `060`, and `120`. | Active absence-sensitive effects remain blocked. |
