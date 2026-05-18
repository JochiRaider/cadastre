---
doc_id: CADASTRE-NLSPEC-030
title: Processing DAG, Lifecycle, and Versioning
doc_type: nlspec
status: candidate
---

## Authority

This document owns the contracts listed in `Exports`. Other Cadastre NLSpecs may import those contracts by exact name and must not restate them. This document has implementation authority only after the document registry marks it `authoritative` and its acceptance criteria pass.

## Purpose

Define deterministic execution order, output permissions, lifecycle machines, run locks, version manifests, declared subsets, and replay-relevant stage state.

## Explicit Non-Scope

- Domain algorithms for parser, identity, source authority, gold correction, graph apply, or package trust.
- Backend-specific implementation mechanisms.

## Imports

- `DirectSourceProhibition`
- `RawRecordImportRun`
- `LakehouseReadCompletenessReceipt`
- `LakehouseSnapshotRef`
- `DatasetVersionRef`
- `LakehouseCommitRef`

- `CoreRecordSchema`
- `CoreRecordValidationAlgorithm`
- `CoreRecordChecksumPolicy`

## Exports

- `ProcessingStageDAG`
- `LakehouseStageStep`
- `FeedStepPattern`
- `StageStateRecord`
- `CDCReplayStateContract`
- `DeterministicSideEffectRecord`
- `LifecycleStatus`
- `LifecycleStateMachineDefinition`
- `ProcessingStageLifecycleResult`
- `RunLockSet`
- `DeclaredDAGSubsetProfile`
- `VersionManifest`
- `ExecuteProcessingStageDAG`
- `ValidateLifecycleStateMachine`
- `ActivationControlledArtifactRef`

## Stage Graph Contract

`ProcessingStageDAG` must define an acyclic, deterministic stage graph. Every node must declare stage ID, package binding, input record classes, output record classes, required profiles, state record kinds, failure isolation behavior, lifecycle machine when production-affecting, and whether partial inputs are allowed.

| Stage class | Permitted production outputs |
| --- | --- |
| `feed_read` | `LakehouseReadCompletenessReceipt`, `StageStateRecord`, diagnostics. |
| `raw_import` | `RawRecord`, `RawRecordImportRun`, import diagnostics. |
| `parse` | Parse results and parser diagnostics. |
| `normalize` | `CadastreSilverObservation`, mapping diagnostics. |
| `identity` | `IdentityDecision`, `IdentityReviewCase`, resolver diagnostics. |
| `gold_derivation` | `GoldFact`, `GoldFactChangeSet`, temporal/correction diagnostics. |
| `graph_projection` | `GraphDeltaSet`, graph projection diagnostics. |
| `graph_apply` | `GraphApplyResult`, backend evidence rows, graph apply diagnostics. |
| `analysis` | `AnalysisFinding`, `AnalysisMetric`, risk workflow records only. |
| `validation` | `ValidationScenario` outputs, validation reports, diagnostics only. |

Any stage emitting a record class outside its permitted set must fail with `FORBIDDEN_STAGE_OUTPUT` before the output is committed.

Deterministic stage ordering rules:

| Condition | Required behavior |
| --- | --- |
| Duplicate `stage_id` | Reject the DAG with `DAG_STAGE_ID_DUPLICATE`. |
| Cycle detected | Reject the DAG with `DAG_CYCLE_DETECTED`. |
| Multiple ready stages | Execute ready stages in ascending lexical `stage_id` order. |
| Stage order replay | Same DAG bytes and requested subset must produce byte-identical ordered stage ID list. |

Failure-isolation precedence:

| Precedence | Condition | Required behavior |
| ---: | --- | --- |
| 1 | Core schema validation fails | Reject before owner algorithm execution. |
| 2 | Stage emits forbidden output | Reject before persistence. |
| 3 | Non-isolated stage failure | Stop all downstream stages and emit deterministic error. |
| 4 | Isolated stage failure | Continue only for declared outputs; watermarks must not advance. |

Every production stage output whose record class is imported from `040.CoreRecordSchema` must pass `040.ValidateCoreRecord` before commit and before external visibility. Schema validation failure must occur before source authority, identity, gold derivation, graph projection, API, export, or analysis behavior can consume the record. A package, parser, resolver, or projector must not bypass `040` by emitting package-local DTOs as production records.

## Lifecycle Contract

`LifecycleStateMachineDefinition` must define closed states, closed events, total transition coverage, deterministic guard precedence, artifact-derived state, illegal-transition behavior, idempotency rules, observability requirements, parent/child boundaries, and conformance tests.

`LifecycleStatus` values are shared names only. Runtime behavior must come from a declared lifecycle machine or a pure deterministic algorithm authority.

| Lifecycle status | Default production meaning |
| --- | --- |
| `draft` | Cannot affect production output. |
| `validated` | May be used only in validation or canary gates. |
| `canary` | May produce canary or shadow output only. |
| `active` | May affect production output when all dependencies are active. |
| `deprecated` | May remain in existing output but must not be newly selected unless policy permits. |
| `retired` | Must not execute or affect new output. |
| `quarantined` | Must not execute and must block dependent package-set activation. |

## Run Locks

`RunLockSet` must be acquired before any production output write. Lock keys must cover source/feed scope, completeness scope, coverage scope, projection scope, graph apply target, package set, and requested output class when concurrent writes could change observable output.

## Version Manifest Contract

`VersionManifest` must include every output-affecting profile, policy, package artifact, package-set manifest, source/feed config hash, stage state hash, lakehouse ref, completeness receipt, coverage assertion, schema artifact, temporal policy, resolver profile, graph profile, graph apply result, and acceptance report that can affect output or replay.

A production output without a complete immutable `VersionManifest` must fail with `VERSION_MANIFEST_INCOMPLETE`. `activation_artifact_refs` must be present in `VersionManifest.included_refs` for every output-affecting activation-controlled artifact.

When a run emits any `040` record, `VersionManifest` must include `core_record_schema_registry_checksum`, `core_record_schema_versions`, `core_record_checksum_policy_version`, `core_record_validation_result_refs`, and `core_record_error_refs` for every rejected record. Canary and shadow records may use null `version_manifest_id` only when this spec declares the execution mode and `040.CommonRecordHeader` permits it.

## Activation-Controlled Artifact Reference Contract

`ActivationControlledArtifactRef` is the canonical reference shape for every output-affecting activation-controlled artifact. Owner specs may require additional artifact payload fields, but they must not redefine this reference shape.

| Field | Required | Rule |
| --- | ---: | --- |
| `artifact_class` | Yes | Closed token from the artifact class registry below. |
| `owner_spec` | Yes | Stable core owner spec that exports the behavior instantiated by the artifact. |
| `artifact_id` | Yes | Stable artifact identifier scoped by owner spec and artifact class. |
| `artifact_version` | Yes | Owner-defined immutable version string. |
| `artifact_checksum` | Yes | SHA-256 over the canonical artifact payload or registry row set. |
| `lifecycle_status` | Yes | Must be allowed for the execution mode. |
| `activation_scope` | Yes | Scope in which the artifact may affect output. |
| `validation_refs` | Yes | Non-empty refs to passing validation rows required by the owner. |
| `package_set_ref` | Required when package-supplied | Immutable `ProductionPackageSetManifest` ref. |
| `supersedes` | No | Prior artifact refs replaced by this artifact. Default `[]`. |
| `effective_from` | No | RFC3339 UTC or null. Default null. |
| `effective_to` | No | RFC3339 UTC or null. Default null. |
| `artifact_payload_location_ref` | Yes | Ref to immutable payload bytes or registry snapshot. |
| `artifact_registry_ref` | Yes | Registry row or snapshot containing owner, class, lifecycle, checksum, and scope. |

Closed `artifact_class` values are:

```text
lakehouse_feed_profile
raw_supplier_profile
lakehouse_read_policy
lakehouse_table_profile
replay_retention_policy
table_maintenance_policy
external_schema_profile
external_schema_artifact
mapping_bundle
parser_profile
source_extension_field_rule_set
source_authority_profile
source_authority_row_set
source_staleness_policy
coverage_dimension_profile
progress_signal_policy
resolver_profile
candidate_generation_profile
temporal_semantics_policy
knowledge_time_policy
late_arrival_policy
gold_correction_policy
graph_projection_profile
graph_backend_profile
graph_read_model_schema_profile
graph_query_translation_profile
graph_apply_profile
analysis_rule_bundle
registry_governance_artifact
package_release
production_package_set
validation_matrix
lakehouse_feed_fixture
golden_corpus
```

### ValidateActivationControlledArtifactRef

```text
ValidateActivationControlledArtifactRef(ref, execution_scope, required_owner_spec):
1. If ref is missing, fail with `ACTIVATION_ARTIFACT_INCOMPLETE`.
2. If `ref.owner_spec != required_owner_spec`, fail with `ACTIVATION_ARTIFACT_OWNER_MISMATCH`.
3. If `artifact_class` is not in the closed registry, fail with `ACTIVATION_ARTIFACT_INCOMPLETE`.
4. If checksum validation fails, fail with the owner checksum code or `ACTIVATION_ARTIFACT_INCOMPLETE`.
5. If lifecycle status is not allowed for execution mode, fail with the owner lifecycle code or `ACTIVATION_ARTIFACT_INCOMPLETE`.
6. If activation scope does not cover execution_scope, fail with the owner scope code or `ACTIVATION_ARTIFACT_INCOMPLETE`.
7. If required validation refs are missing, fail with `ACTIVATION_ARTIFACT_INCOMPLETE`.
8. If package-set membership is required and mismatched, fail with the package owner code or `ACTIVATION_ARTIFACT_INCOMPLETE`.
9. If a superseded artifact is selected, fail with `ACTIVATION_ARTIFACT_INCOMPLETE`.
```

Failure precedence is missing ref, owner mismatch, invalid artifact class, checksum mismatch, lifecycle invalid, activation scope mismatch, validation refs missing, package set mismatch, then superseded artifact selected.

## Declared Subset Contract

A production stage subset may execute only with an active `DeclaredDAGSubsetProfile`. The subset profile must state allowed outputs, forbidden outputs, cleanup behavior, absence behavior, projection behavior, watermark behavior, and required validation rows.

## Execution Algorithm

```text
ExecuteProcessingStageDAG(dag, requested_scope, package_set):
1. Validate duplicate stage IDs, dag acyclicity, and active lifecycle state.
2. Resolve active package set, stage bindings, and all stage-required activation-controlled artifact refs.
3. Validate requested subset or full DAG mode.
4. Acquire RunLockSet.
5. For each stage in canonical topological order, sorting simultaneously ready stages by lexical `stage_id`:
   a. Validate imported profiles, activation-controlled artifact refs, and state refs before stage execution.
   b. Validate permitted outputs.
   c. Execute package role or pure product algorithm.
   d. Persist StageStateRecord for output-affecting state.
   e. Persist ProcessingStageLifecycleResult.
   f. Stop on non-isolated failure and emit deterministic error.
6. Emit VersionManifest with every output-affecting activation artifact ref before any output is externally visible.
7. Release locks only after commit refs and lifecycle results are persisted.
```

## DAG, Lifecycle, and Versioning Contract Details

### StageStateKindRegistry

| State kind | Required ref fields | Used by | Replay requirement |
| --- | --- | --- | --- |
| `feed_read_checkpoint` | feed profile, target ref, checkpoint checksum | `020`, `030` | Included in `VersionManifest`. |
| `object_checkpoint` | object ref, byte range, checksum | `020` | Included when object batch affects output. |
| `partition_checkpoint` | partition key, bounds, schema ref | `020` | Included when partition set affects output. |
| `cdc_offset` | source event metadata, offset value, schema-history ref | `020`, `080` | Included and non-authoritative for fact time. |
| `schema_history_ref` | schema-history artifact ref and checksum | `020`, `080` | Required for CDC replay sufficiency. |
| `deterministic_side_effect` | side-effect record ref and checksum | `080` | Replay from recorded value only. |
| `graph_apply_checkpoint` | delta-set checksum, applied batch IDs, backend evidence refs | `090` | Required for resume. |
| `watermark_checkpoint` | watermark kind, attempted value, commit record | `060`, `080`, `090` | Required before advancement. |
| `package_activation_ref` | package-set manifest and deployment revision | `100` | Required for production output. |
| `replay_ref` | replay input set, equivalence checksum, result | `080`, `120` | Required for replay output. |

### LifecycleStateMachineDefinition row schema

| Field | Required | Rule |
| --- | ---: | --- |
| `states` | Yes | Closed enum. |
| `events` | Yes | Closed enum. |
| `guards` | Yes | Deterministic ordered list. |
| `transition_table` | Yes | Total for every state/event pair unless pure deterministic algorithm is declared. |
| `illegal_transition_behavior` | Yes | Must emit owner-specific error and no state mutation. |
| `idempotency_rule` | Yes | Replayed transition must produce identical result or explicit no-op. |
| `artifact_derived_state_rule` | Yes | Defines which artifacts determine state. |
| `observability_fields` | Yes | Includes lifecycle ID, prior state, event, guard, result, error, checksums. |
| `parent_child_composition` | Yes | Parent must not depend on child in-memory state. |
| `validation_rows` | Yes | Must reference `120` rows. |

Lifecycle diagrams are representational unless generated from a declared lifecycle table.

### VersionManifestCompletenessMatrix

| Output class | Required refs |
| --- | --- |
| Raw import output | feed profile, read policy, raw feed manifest, raw import package, table/object refs, state records. |
| Any `040` core record output | core record schema registry checksum, core record schema versions, core record checksum policy version, validation result refs, rejected-record error refs. |
| Silver output | raw refs, parser package, mapping bundle, external schema profile, validation output. |
| Identity output | resolver profile, evidence scope, identity decision refs, review case refs when applicable. |
| Gold output | temporal policy, source authority, coverage, correction policy, source refs, snapshot refs. |
| Graph delta/apply | projection profile, graph delta set, idempotency key, backend profile, apply result. |
| API/export output | owner state labels, authorization/redaction policy, page-token policy, derived view state. |
| Package activation | package-set manifest, trust evidence, compatibility, validation, approval, rollback refs. |

### VersionManifestSchema

| Field | Required | Rule |
| --- | ---: | --- |
| `version_manifest_id` | Yes | Deterministic ID computed from `manifest_kind` and canonical included refs. |
| `manifest_kind` | Yes | Closed token for run, replay, validation, graph rebuild, or package activation. |
| `included_refs` | Yes | Canonically sorted refs for every output-affecting artifact. |
| `schema_registry_checksum` | Yes | Active `040` core schema registry checksum. |
| `package_set_checksum` | Yes when packages affect output | Immutable `ProductionPackageSetManifest` checksum. |
| `stage_state_refs` | Yes | Stage state refs and checksums sorted by state kind then ID. |
| `snapshot_refs` | Yes when lakehouse reads affect output | `LakehouseSnapshotRef` and `DatasetVersionRef` refs. |
| `commit_refs` | Yes when writes occur | `LakehouseCommitRef` refs for attempted and successful writes. |
| `acceptance_report_refs` | Required for promotion | Acceptance report refs and checksums. |
| `activation_artifact_refs` | Required when activation-controlled artifacts affect output | Canonically sorted `ActivationControlledArtifactRef` rows and checksums. |
| `created_at` | Yes | RFC3339 UTC manifest creation time; excluded from manifest ID and included in checksum. |
| `manifest_checksum` | Yes | SHA-256 over `040.CanonicalJSON` excluding only `manifest_checksum`. |

`version_manifest_id` must be `vm_` plus SHA-256 lowercase hex over `040.CanonicalJSON` of `manifest_kind` and `included_refs`. `created_at` must not affect `version_manifest_id`.

### RequiredLifecycleMachineBindings

| Production lifecycle | Required owner | Transition table status | Validation row |
| --- | --- | --- | --- |
| package/profile activation | `100` plus package/profile owner | TODO: total transition table required before authoritative promotion | `val-030-lifecycle-package-activation` |
| production run execution | `030` | TODO: total transition table required before authoritative promotion | `val-030-lifecycle-production-run` |
| feed stage execution | `020`, `030` | TODO: total transition table required before authoritative promotion | `val-030-lifecycle-feed-stage` |
| graph apply | `090`, `030` | TODO: total transition table required before authoritative promotion | `val-030-lifecycle-graph-apply` |
| validation acceptance | `120`, `030` | TODO: total transition table required before authoritative promotion | `val-030-lifecycle-validation-acceptance` |

### RunLockKeyDerivation

```text
RunLockKeyDerivation(scope):
1. Build an ordered tuple of source/feed scope, completeness scope, coverage scope, projection scope, graph apply target, package set, and requested output class.
2. Serialize the tuple with `040.CanonicalJSON`.
3. Hash with SHA-256 and prefix with `runlock_`.
4. If two non-identical tuples produce the same lock key, fail with `RUN_LOCK_COLLISION`.
```

### RunLockSet acquisition semantics

| Condition | Required behavior |
| --- | --- |
| All lock keys available | Acquire the complete lock set before any production output write. |
| One or more lock keys unavailable | Acquire no locks or release every acquired lock before returning `RUN_LOCK_ACQUISITION_FAILED`. |
| Partial acquisition failure | Emit `RUN_LOCK_ACQUISITION_FAILED`; write no production output and advance no watermark. |
| Lock lost during run | Stop before next output commit and emit `RUN_LOCK_LOST`; downstream stages must not run. |
| Stale lock | TODO: product governance must define default lease TTL, heartbeat interval, and stale-lock recovery condition before authoritative promotion. |

### DeclaredDAGSubsetProfile matrix

| Subset mode | Allowed outputs | Forbidden outputs | Absence behavior | Cleanup behavior | Projection behavior | Watermark behavior | Validation requirement |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `validation_only` | validation reports, diagnostics | production records | forbidden | forbidden | forbidden | forbidden | negative authority rows |
| `feed_replay` | raw records and receipts | silver/gold/graph/API | no absence | no cleanup | no projection | no advancement | feed replay rows |
| `graph_rebuild` | graph rebuild manifest and graph apply result | raw/silver/gold mutation | no absence | no source cleanup | allowed from persisted deltas | projection only if `060` permits | rebuild equivalence rows |
| `package_canary` | canary output and activation evidence | current production output | no absence | no cleanup | canary only | no production advancement | package canary rows |

### Activation artifact errors

| Error code | Emitted when |
| --- | --- |
| `ACTIVATION_ARTIFACT_INCOMPLETE` | A required activation-controlled artifact ref is missing required fields, invalid, unvalidated, out of scope, superseded, or otherwise incomplete after more specific owner codes are unavailable. |

### Acceptance Criteria

| ID | Criterion |
| --- | --- |
| `030-CLEANUP-AC-001` | No banned reference class remains. |
| `030-CLEANUP-AC-002` | Stage output permissions remain total for declared stage classes. |
| `030-CLEANUP-AC-003` | Any stage emitting a forbidden record class still fails before commit. |
| `030-CLEANUP-AC-004` | `VersionManifest` remains the output-affecting reproducibility record. |
| `030-SCHEMA-PATCH-AC-001` | Every production output class owned by `040` is schema-validated before commit. |
| `030-SCHEMA-PATCH-AC-002` | `VersionManifest` records the active core schema registry checksum for every output-affecting run. |
| `030-SCHEMA-PATCH-AC-003` | A forbidden or malformed core record fails before downstream domain algorithms execute. |

| `030-DAG-ORDER-AC-001` | Duplicate stage IDs fail with `DAG_STAGE_ID_DUPLICATE`, cycles fail with `DAG_CYCLE_DETECTED`, and ready-stage ties sort lexically by `stage_id`. |
| `030-RUNLOCK-AC-001` | Run lock acquisition is all-or-nothing and partial acquisition failure releases acquired locks before returning `RUN_LOCK_ACQUISITION_FAILED`. |
| `030-VERSION-MANIFEST-AC-001` | Same included refs and manifest kind produce the same `version_manifest_id`; checksum includes `created_at` but ID does not. |
| `030-ACTIVATION-AC-001` | Every output-affecting activation-controlled artifact appears in `VersionManifest`. |
| `030-ACTIVATION-AC-002` | Inactive, checksum-mismatched, scope-mismatched, or unvalidated artifacts fail with deterministic precedence. |
| `030-ACTIVATION-AC-003` | Two implementations given the same refs produce byte-identical `version_manifest_id`. |

## Definition of Done

| ID | Criterion |
| --- | --- |
| `030-AC-001` | The same DAG, package set, inputs, stage state, and requested subset produce the same stage order and output classes. |
| `030-AC-002` | A forbidden output class is rejected before commit and before downstream stage execution. |
| `030-AC-003` | Every production-affecting lifecycle has closed states, closed events, total transitions, illegal-transition behavior, and validation criteria. |
| `030-AC-004` | Every output-affecting state record is hashable, replayable, and included in `VersionManifest`. |
| `030-AC-005` | Production replay rejects missing, mutable-only, checksum-mismatched, or retention-ineligible manifest refs before writing output. |

## Open Questions

Open questions marked `TODO:` block authoritative status for the affected contract. A downstream implementation must not resolve a `TODO:` by inference.
