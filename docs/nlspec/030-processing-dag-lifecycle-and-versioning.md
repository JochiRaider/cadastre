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
- `LifecycleTransitionEvidence`
- `ApplyLifecycleEvent`
- `ActivationControlledArtifactLifecycleMachine`
- `ProductionRunExecutionLifecycleMachine`
- `StageExecutionLifecycleMachine`
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

### LifecycleTransitionEvidence schema

`LifecycleTransitionEvidence` is the 030-owned runtime-state record for every production-affecting lifecycle evaluation. Owner-specific lifecycle machines may add owner payload refs, but they must not omit or rename the fields in this table.

| Field | Required rule |
| --- | --- |
| `transition_id` | Deterministic ID over lifecycle machine ID, machine version, subject ref, event idempotency key, prior state, event, selected transition row, next state, and output-affecting artifact refs. |
| `lifecycle_machine_id` | Closed machine identifier. |
| `machine_version` | Immutable owner-defined version. |
| `machine_checksum` | SHA-256 over states, events, guard definitions, transition matrix, no-op rules, validation refs, and owner guard row refs. |
| `subject_ref` | Ref to the artifact, package set, run, stage, graph apply, validation report, review case, or owner-specific lifecycle subject. |
| `subject_checksum` | Checksum of subject state at evaluation time. |
| `prior_state` | State derived before event evaluation from owner-declared artifacts and prior persisted transition evidence. |
| `event` | Closed event token from the selected machine. |
| `event_idempotency_key` | Required for every production transition. Validation-only fixtures may omit it only when the validation row declares omission behavior. |
| `guard_results` | Canonically ordered guard outcomes, sorted by declared guard order and then guard ID. |
| `selected_transition_row_id` | Exact transition row that selected the result. |
| `next_state` | Result state, including same-state no-op and illegal-transition cases. |
| `transition_kind` | Closed enum: `state_change`, `idempotent_no_op`, `guard_refusal`, `illegal_transition`, `terminal_no_op`. |
| `side_effect_refs` | Canonically sorted refs to writes produced by the transition. Empty array means no writes. |
| `error_code` | Null only for successful state changes or idempotent no-op. Otherwise the value must be the most specific owner code available. |
| `version_manifest_id` | Required when the transition can affect output, replay, activation, graph apply, watermark eligibility, validation acceptance, or CI gating. |
| `created_at` | RFC3339 UTC. Excluded from `transition_id`; included in the evidence checksum. |

### ApplyLifecycleEvent algorithm

```text
ApplyLifecycleEvent(machine, subject_ref, event, event_context):
1. Validate machine owner, machine version, machine checksum, and validation refs.
2. Load subject by immutable ref and checksum.
3. Derive prior_state only from owner-declared artifacts and persisted transition evidence.
4. Validate event belongs to machine.events.
5. Validate event_idempotency_key for production transitions.
6. Evaluate guards in declared order.
7. Select exactly one transition matrix row.
8. If selected row is illegal, emit LifecycleTransitionEvidence with no side effects.
9. If selected row is a no-op, emit no-op evidence unless byte-identical evidence already exists.
10. If selected row changes state, write only side effects named by that row.
11. Persist LifecycleTransitionEvidence before any externally visible output, active pointer, derived-view advancement, watermark effect, or last-known-good update.
12. Include machine checksum and transition evidence refs in VersionManifest when output-affecting.
```

### Lifecycle guard precedence

Every lifecycle machine must evaluate guard families in this order unless an owner spec adds a more specific guard inside one family.

| Order | Guard family |
| ---: | --- |
| 1 | Authorization and private-binding exposure. |
| 2 | Subject existence and checksum. |
| 3 | Machine ownership, version, checksum, and validation refs. |
| 4 | Run lock or activation lock. |
| 5 | Idempotency replay or idempotency conflict. |
| 6 | Parent/child persisted state. |
| 7 | Quarantine and emergency override. |
| 8 | Owner artifact gates. |
| 9 | Transition matrix row selection. |
| 10 | Side-effect preflight. |

### Lifecycle totality and illegal transition rule

Every lifecycle machine must cover every state/event pair by explicit transition rows or by exactly one `all_other_state_event_pairs` row. An `all_other_state_event_pairs` row is valid only when it emits `LIFECYCLE_ILLEGAL_TRANSITION`, writes no production output, mutates no active pointer, advances no watermark, and preserves prior state.

A repeated event with the same subject, event idempotency key, machine version, prior state, and input checksums must produce byte-identical transition evidence or an explicit idempotent no-op. The same event idempotency key with different input checksums must fail with `LIFECYCLE_IDEMPOTENCY_CONFLICT` before side effects.

### ActivationControlledArtifactLifecycleMachine

Machine ID: `030.ActivationControlledArtifactLifecycleMachine.v1`.

States:

```text
draft
validated
canary
active
deprecated
retired
quarantined
```

Events:

```text
validation_passed
validation_failed
start_canary
canary_passed
canary_failed
activate
deprecate
expire_deprecation
retire
quarantine
clear_quarantine
rollback_select
replay_same_event
```

Required transition rows:

| Prior state | Event | Next state | Required behavior |
| --- | --- | --- | --- |
| `draft` | `validation_passed` | `validated` | Persist validation pass evidence. |
| `draft` | `validation_failed` | `draft` | Persist validation failure evidence; no production output. |
| `validated` | `start_canary` | `canary` | Canary or shadow output only. |
| `canary` | `canary_passed` | `validated` | Persist canary pass evidence. |
| `canary` | `canary_failed` | `validated` | Persist failure evidence and block activation until a new pass. |
| `validated` | `activate` | `active` | Owner activation guards and validation refs must pass. |
| `active` | `deprecate` | `deprecated` | Persist deprecation evidence. |
| `deprecated` | `expire_deprecation` | `retired` | Persist expiry evidence. |
| `deprecated` | `retire` | `retired` | Persist retirement evidence. |
| any non-retired state | `quarantine` | `quarantined` | Block execution and dependent activation. |
| `quarantined` | `clear_quarantine` | `validated` | Never clear directly to `active`. |
| `deprecated` | `rollback_select` | `active` | Allowed only through owner-approved immutable rollback target. |
| terminal repeated identical event | `replay_same_event` | same state | Idempotent no-op. |
| `all_other_state_event_pairs` | any other event | same state | Illegal transition evidence; no mutation. |

Canary and shadow states must not write current production output, advance source/projection/graph watermarks, or mark last-known-good.

### ProductionRunExecutionLifecycleMachine

Machine ID: `030.ProductionRunExecutionLifecycleMachine.v1`.

States:

```text
planned
preflighted
locks_acquired
running
succeeded
failed
aborted
```

Events:

```text
preflight_passed
preflight_failed
locks_acquired
locks_failed
start_run
stage_succeeded
stage_failed_isolated
stage_failed_nonisolated
lock_lost
complete
abort
replay_same_event
```

Required transition rows:

| Prior state | Event | Next state | Required behavior |
| --- | --- | --- | --- |
| `planned` | `preflight_passed` | `preflighted` | Persist preflight evidence. |
| `planned` | `preflight_failed` | `failed` | No production output. |
| `preflighted` | `locks_acquired` | `locks_acquired` | Persist complete lock set. |
| `preflighted` | `locks_failed` | `failed` | Release acquired locks and write no production output. |
| `locks_acquired` | `start_run` | `running` | Persist run-start evidence. |
| `running` | `stage_succeeded` | `running` | Continue eligible downstream stages. |
| `running` | `stage_failed_isolated` | `running` | Continue only for declared isolated outputs. |
| `running` | `stage_failed_nonisolated` | `failed` | Stop downstream stages. |
| `running`, `locks_acquired`, or `preflighted` | `lock_lost` | `failed` | Stop before next output commit. |
| `running` | `complete` | `succeeded` | Persist completion evidence and final manifest refs. |
| `planned`, `preflighted`, `locks_acquired`, or `running` | `abort` | `aborted` | Persist abort evidence and write no further output. |
| terminal repeated identical event | `replay_same_event` | same state | Idempotent no-op. |
| `all_other_state_event_pairs` | any other event | same state | Illegal transition evidence; no mutation. |

### StageExecutionLifecycleMachine

Machine ID: `030.StageExecutionLifecycleMachine.v1`.

States:

```text
pending
ready
running
retry_wait
succeeded
failed_isolated
failed_nonisolated
skipped
no_op
```

Events:

```text
dependencies_satisfied
start_stage
core_schema_failed
forbidden_output
retryable_error
retry_timer_elapsed
retry_exhausted
nonretryable_error
stage_success
stage_no_output
upstream_nonisolated_failed
replay_same_event
```

Required transition rows:

| Prior state | Event | Next state | Required behavior |
| --- | --- | --- | --- |
| `pending` | `dependencies_satisfied` | `ready` | Stage may start when ordering and dependency guards pass. |
| `ready` | `start_stage` | `running` | Persist stage-start evidence. |
| `running` | `retryable_error` | `retry_wait` | Allowed only when retry policy permits. |
| `retry_wait` | `retry_timer_elapsed` | `running` | Resume attempt under the same stage context. |
| `retry_wait` | `retry_exhausted` | `failed_nonisolated` or `failed_isolated` | `failed_isolated` is allowed only when stage isolation permits. |
| `running` | `core_schema_failed` | `failed_nonisolated` or `failed_isolated` | `failed_isolated` is allowed only when stage isolation permits. |
| `running` | `forbidden_output` | `failed_nonisolated` | Emit `FORBIDDEN_STAGE_OUTPUT`; no forbidden output commit. |
| `running` | `nonretryable_error` | `failed_nonisolated` or `failed_isolated` | `failed_isolated` is allowed only when stage isolation permits. |
| `running` | `stage_success` | `succeeded` | Persist output refs and transition evidence. |
| `running` | `stage_no_output` | `no_op` | Persist no-op evidence. |
| `pending` or `ready` | `upstream_nonisolated_failed` | `skipped` | Persist skip evidence. |
| terminal repeated identical event | `replay_same_event` | same state | Idempotent no-op. |
| `all_other_state_event_pairs` | any other event | same state | Illegal transition evidence; no mutation. |

### ProcessingStageLifecycleResult schema

| Field | Required rule |
| --- | --- |
| `stage_id` | Exact DAG stage ID. |
| `lifecycle_machine_id` | `030.StageExecutionLifecycleMachine.v1` unless an owner-specific machine is named. |
| `terminal_state` | One `030.StageExecutionLifecycleMachine.v1` state. |
| `transition_evidence_refs` | Non-empty for production stage execution. |
| `output_refs` | Canonically sorted output refs; default `[]`. |
| `blocked_effects` | Array of blocked effects such as `absence`, `cleanup`, `retraction`, `graph_expiry`, or `watermark`; default `[]`. |
| `error_refs` | Owner-specific errors; default `[]`. |
| `version_manifest_id` | Required for production output. |

## Run Locks

`RunLockSet` must be acquired before any production output write. Lock keys must cover source/feed scope, completeness scope, coverage scope, projection scope, graph apply target, package set, and requested output class when concurrent writes could change observable output.

## Version Manifest Contract

`VersionManifest` must include every output-affecting profile, policy, package artifact, package-set manifest, source/feed config hash, stage state hash, lakehouse ref, completeness receipt, coverage assertion, schema artifact, temporal policy, resolver profile, graph profile, graph apply result, lifecycle machine definition, lifecycle machine checksum, lifecycle transition evidence ref, lifecycle state artifact ref, and acceptance report that can affect output or replay.

A production output without a complete immutable `VersionManifest` must fail with `VERSION_MANIFEST_INCOMPLETE`. `activation_artifact_refs` must be present in `VersionManifest.included_refs` for every output-affecting activation-controlled artifact.

When a run emits any `040` record, `VersionManifest` must include `core_record_schema_registry_checksum`, `core_record_schema_versions`, `core_record_checksum_policy_version`, `core_record_validation_result_refs`, and `core_record_error_refs` for every rejected record. Canary and shadow records may use null `version_manifest_id` only when this spec declares the execution mode and `040.CommonRecordHeader` permits it.

A `SourceAuthorityClosureMatrix` validation result may be referenced in `VersionManifest`, but it must not substitute for the underlying activation-controlled artifact refs and checksums. Absence-sensitive output must include every consulted `060` row-set ref even when the closure validation result is also included.

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
lakehouse_feed_category_closure_row_set
raw_supplier_profile
lakehouse_read_policy
declared_dag_subset_profile
lakehouse_table_profile
replay_retention_policy
table_maintenance_policy
external_schema_profile
external_schema_artifact
mapping_bundle
parser_profile
observation_to_ocsf_mapping_row_set
external_enum_mapping_rule_set
ocsf_base_event_field_policy_set
profile_resolution_manifest
ocsf_profile_upgrade_report
source_extension_field_rule_set
observation_type_external_mapping_validation_matrix
source_authority_profile
source_authority_row_set
source_staleness_policy
coverage_dimension_profile
lakehouse_feed_completeness_profile
supplier_collection_visibility_profile
control_result_mapping_row_set
source_history_retention_profile
absence_derivation_policy
projection_watermark_policy
progress_signal_policy
resolver_profile
identifier_evidence_class_row_set
identifier_scope_row_set
asset_generation_boundary_row_set
target_selector_safety_policy
resolver_decision_matrix_row_set
identity_confidence_band_row_set
identity_review_routing_policy
identity_split_policy
resolver_explanation_policy
candidate_generation_profile
temporal_semantics_policy
knowledge_time_policy
late_arrival_policy
gold_correction_policy
correction_snapshot_ref_policy
replay_equivalence_policy
graph_rebuild_equivalence_policy
event_sequence_validation_corpus
bitemporal_query_mode_visibility_policy
graph_projection_profile
graph_projection_row_set
graph_edge_semantics_row_set
graph_traversal_class_row_set
graph_object_output_eligibility_row_set
graph_property_evidence_policy
graph_backend_profile
graph_backend_taxonomy_mapping_profile
graph_read_model_schema_profile
graph_query_translation_profile
graph_apply_profile
derived_view_lag_policy
graph_rebuild_manifest_profile
analysis_rule_bundle
registry_governance_artifact
package_release
production_package_set
package_type_policy_row_set
package_repository_model_row_set
package_trust_policy
package_transparency_evidence_policy
package_provenance_policy
package_sbom_policy
package_dependency_lock_policy
package_compatibility_matrix
package_deprecation_window_policy
last_known_good_health_gate
rollback_compatibility_policy
quarantine_scope_policy
emergency_package_override_policy
package_promotion_gate_policy
package_stage_binding
package_developer_contract
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

A production stage subset may execute only with an active `DeclaredDAGSubsetProfile` referenced by `030.ActivationControlledArtifactRef` with `artifact_class = declared_dag_subset_profile`. The subset profile must make every absence, cleanup, retraction, graph-expiry, projection, and watermark effect explicit. Omission of an effect field is not permission.

### DeclaredDAGSubsetProfile schema

| Field | Required | Default or omission behavior | Rule |
| --- | ---: | --- | --- |
| `subset_profile_id` | Yes | none | Stable artifact ID scoped to `030`. |
| `covered_stage_ids` | Yes | none | Non-empty canonical array of stage IDs the subset may execute. |
| `covered_feed_profile_refs` | No | `[]` | Feed profile refs covered by the subset. Empty means no feed profile is covered. |
| `allowed_output_classes` | Yes | none | Non-empty closed output class set. A stage output outside this set fails with `DECLARED_DAG_SUBSET_OUTPUT_FORBIDDEN`. |
| `forbidden_output_classes` | No | `[]` | Additional closed output classes that must fail even if a stage would otherwise allow them. |
| `absence_behavior` | No | `forbidden` | Closed enum: `forbidden`, `allowed_when_060_effect_allowed`. |
| `cleanup_behavior` | No | `forbidden` | Closed enum: `forbidden`, `allowed_when_060_effect_allowed`. |
| `retraction_behavior` | No | `forbidden` | Closed enum: `forbidden`, `allowed_when_060_effect_allowed`. |
| `graph_expiry_behavior` | No | `forbidden` | Closed enum: `forbidden`, `allowed_when_060_effect_allowed`. |
| `watermark_behavior` | No | `forbidden` | Closed enum: `forbidden`, `allowed_when_060_effect_allowed`. |
| `projection_behavior` | No | `forbidden` | Closed enum: `forbidden`, `canary_only`, `persisted_delta_replay_only`, `allowed_when_profiles_match`. |
| `validation_refs` | Yes | none | Non-empty refs to subset positive and negative validation rows. |
| `activation_scope` | Yes | none | Execution scope covered by the profile. |

A production subset request without a matching active subset profile must fail with `DECLARED_DAG_SUBSET_PROFILE_MISSING` before any stage executes. A subset profile whose scope does not cover the requested feed profile, stage IDs, output class, or activation scope must fail with `DECLARED_DAG_SUBSET_SCOPE_MISMATCH`.

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
| `temporal_resolution_ref` | temporal resolution ref, policy refs, resolution checksum | `080` | Required before gold output and replay. |
| `late_arrival_route_state` | route state ref, late-arrival policy ref, authority and watermark refs | `080`, `060` | Required when late evidence affects output or quarantine. |
| `gold_correction_changeset_ref` | change-set ref, operation list checksum, correction policy ref | `080` | Required for correction output and replay. |
| `correction_snapshot_ref_set` | old snapshot ref, new snapshot ref, table-set checksum, retention proof | `020`, `080` | Required before correction output. |
| `replay_input_sufficiency_check` | output class, required refs, failure precedence result | `080`, output owner | Required before replay output. |
| `replay_equivalence_checksum` | output class, included refs, excluded volatile fields, checksum | `080`, output owner | Required for replay acceptance. |
| `graph_handoff_effect_ref` | `GoldFactChangeSet` ref, handoff effect, authority refs when applicable | `080`, `090` | Required when graph projection depends on correction effects. |
| `graph_apply_checkpoint` | delta-set checksum, applied batch IDs, backend evidence refs | `090` | Required for resume. |
| `watermark_checkpoint` | watermark kind, attempted value, commit record | `060`, `080`, `090` | Required before advancement. |
| `package_activation_ref` | package-set manifest and deployment revision | `100` | Required for production output. |
| `replay_ref` | replay input set, equivalence checksum, result | `080`, `120` | Required for replay output. |
| `lifecycle_transition_evidence` | machine ID, machine version, transition evidence ref, evidence checksum | `030`, lifecycle owners | Required for output-affecting lifecycle transitions. |
| `lifecycle_state_artifact` | subject ref, derived state, machine checksum, transition evidence refs | `030`, lifecycle owners | Required when lifecycle state affects replay, activation, graph apply, watermark eligibility, or CI gating. |

### LifecycleStateMachineDefinition row schema

| Field | Required | Rule |
| --- | ---: | --- |
| `lifecycle_machine_id` | Yes | Stable closed machine identifier. |
| `machine_version` | Yes | Immutable owner-defined machine version. |
| `machine_checksum` | Yes | SHA-256 over states, events, guards, transition table, no-op rules, validation refs, and owner guard row refs. |
| `states` | Yes | Closed enum. |
| `events` | Yes | Closed enum. |
| `guards` | Yes | Deterministic ordered list using `Lifecycle guard precedence`. |
| `transition_table` | Yes | Total for every state/event pair unless pure deterministic algorithm is declared. |
| `all_other_state_event_pairs` | Required when explicit rows do not enumerate every pair | Must be illegal-transition or deterministic no-op as declared by this spec. |
| `illegal_transition_behavior` | Yes | Must emit owner-specific error or `LIFECYCLE_ILLEGAL_TRANSITION` and no state mutation. |
| `idempotency_rule` | Yes | Replayed transition must produce byte-identical result or explicit no-op; changed inputs under the same key fail. |
| `artifact_derived_state_rule` | Yes | Defines which artifacts and persisted transition evidence determine state. |
| `observability_fields` | Yes | Must include every `LifecycleTransitionEvidence` required field. |
| `parent_child_composition` | Yes | Parent must not depend on child in-memory state. |
| `validation_rows` | Yes | Must reference `120` rows and owner-specific fixtures. |

Lifecycle diagrams are representational unless generated from a declared lifecycle table.

### VersionManifestCompletenessMatrix

| Output class | Required refs |
| --- | --- |
| Raw import output | feed profile, feed category closure row set, read policy, raw feed manifest, raw import package, target refs, state records, and feed feasibility assessment ref when activation-sensitive. |
| Any `040` core record output | core record schema registry checksum, core record schema versions, core record checksum policy version, validation result refs, rejected-record error refs. |
| Silver output | raw refs, parser package, parser profile, mapping bundle, `ObservationToOCSFMappingRowSet`, `ExternalSchemaProfile`, `ExternalSchemaArtifactRef`, `ProfileResolutionManifest`, `ExternalEnumMappingRuleSet`, `OCSFBaseEventFieldPolicySet`, `SourceExtensionFieldRuleSet`, `CanonicalValidationOutput`, and observation-type validation matrix refs. |
| Identity output | resolver profile row set, identifier evidence class row set, identifier scope row set, candidate generation profile, asset generation boundary row set, target selector safety policy, resolver decision matrix row set, identity confidence band row set, identity review routing policy, identity split policy, resolver explanation policy, evidence item refs/checksums, identity decision refs, review case refs when applicable, resolver explanation refs/checksums, graph correction handoff refs when applicable, activation report refs when activation or promotion is in scope, and shadow/canary refs when promotion uses them. |
| Temporal resolution output | `TemporalSemanticsPolicy` row ref, `KnowledgeTimeImportPolicy` row ref, source evidence refs, selected input path/value checksum, `TemporalObservationTimeResolution` ref, and resolution checksum. |
| Gold fact output | temporal resolution ref and checksum, exact source authority row refs, lakehouse feed completeness row refs, coverage dimension/profile refs, coverage assertion refs, staleness policy refs, progress-signal policy refs when consulted, control-result mapping refs for control facts, source-history retention refs for history/no-change facts, supplier visibility refs for permission-sensitive absence, absence derivation policy refs, source refs, derivation refs, and validation refs. |
| Gold correction output | `GoldFactChangeSet` ref, correction policy ref, assertion transition row ref, old `LakehouseSnapshotRef`, new `LakehouseSnapshotRef`, table-set checksum ref, retention-protection ref, temporal resolution refs, authority refs, absence refs when applicable, and graph handoff effect refs. |
| Late-arrival output | late-arrival policy row ref, late-arrival route state, temporal resolution ref, authority/completeness/coverage/staleness refs, absence refs when applicable, quarantine refs when selected, and watermark refs when consulted. |
| Replay output | `ReplayEquivalencePolicy` output-class row ref, `ReplayInputSufficiencyCheck` ref, replay equivalence checksum, required owner-specific included-field refs, excluded volatile-field list, and failure-precedence result. |
| CDC-shaped feed replay output | CDC raw subtype refs, schema-history refs, offset refs, tombstone refs, heartbeat refs, replay sufficiency check, and non-authority diagnostics proving CDC metadata was not used as fact time or absence authority. |
| Graph handoff output | `GoldFactChangeSet` ref, graph handoff effect ref, projection profile ref when known, `070.GraphCorrectionHandoff` refs when identity split projection occurs, resolver explanation checksum when split-driven, `060.AbsenceDerivationResult` refs for expiry or cleanup, and validation refs. |
| Absence, cleanup, retraction, or graph expiry output | requested effect token, `060.AbsenceDerivationResult`, `SourceAuthorityClosureMatrix` validation result ref or validation row ref, and all row-set refs consulted by `DeriveAbsenceOrUnknown`. |
| Watermark output | `ProjectionWatermarkPolicy`, `WatermarkCommitRecord`, completeness decision refs, source-authority closure refs, blocking/no-op refs when advancement is denied, coverage refs where applicable, staleness refs, and all consulted `060` row-set refs. |
| Graph delta/apply | graph projection profile, graph projection row-set checksum, graph edge semantics row-set checksum, traversal class row refs where applicable, graph object output eligibility row-set checksum, graph property evidence policy refs, backend taxonomy mapping profile ref, graph query translation profile ref, graph read-model schema profile ref, graph apply profile ref, derived-view lag policy ref, graph delta set ref, idempotency key, backend profile ref, backend schema fingerprint, graph apply lifecycle transition evidence, graph apply result, graph rebuild manifest ref when rebuild is involved, graph index consistency check refs, and derived-view state refs. |
| Lifecycle-affecting output | lifecycle machine ID, version, checksum, transition evidence refs, selected transition row IDs, lifecycle state artifact refs, and owner guard row refs. |
| API/export output | `110.CommonApiResponseEnvelope` ref, `api_contract_version`, request checksum, endpoint outcome matrix checksum, state-label mapping checksum, generated `ErrorCodeRegistry` checksum, authorization policy refs, `110.AuthorizationDecision` refs, redaction policy refs, redaction context checksum, page-token policy refs, derived-view state refs when graph-derived output is served, audit event refs, owner state refs, owner error refs, compliance/export checksums, and API/export validation refs. |
| Package activation | package-set manifest checksum, package release manifest refs and checksums, package type policy row refs, package repository model row refs, artifact digests, sizes, media types, subject digests, repository metadata refs, repository snapshot refs, repository freshness proof refs, repository anti-rollback state refs, trust policy refs, signature verification result refs, transparency evidence refs, attestation set refs, build provenance refs, SBOM refs, dependency lock refs, compatibility matrix refs, validation refs, package developer contract refs, package stage binding refs, promotion record refs, deployment revision refs, last-known-good health gate refs, last-known-good package-set refs when applicable, rollback compatibility policy refs, rollback plan/result refs, quarantine record refs, quarantine scope policy refs, emergency override refs, deprecation window policy refs, activation failure event refs when candidate activation fails, lifecycle transition evidence refs, and approval refs. |

Package activation refs required by `VersionManifestCompletenessMatrix` must appear in `included_refs`. Package-specific manifest fields must not create a parallel manifest mechanism unless this spec explicitly adds a field row and defines checksum inclusion behavior.

### VersionManifestSchema

| Field | Required | Rule |
| --- | ---: | --- |
| `version_manifest_id` | Yes | Deterministic ID computed from `manifest_kind` and canonical included refs. |
| `manifest_kind` | Yes | Closed token for run, replay, validation, graph rebuild, or package activation. |
| `included_refs` | Yes | Canonically sorted refs for every output-affecting artifact. API/export output must represent the required `VersionManifestCompletenessMatrix` refs through this field rather than through a parallel manifest mechanism. |
| `schema_registry_checksum` | Yes | Active `040` core schema registry checksum. |
| `package_set_checksum` | Yes when packages affect output | Immutable `ProductionPackageSetManifest` checksum. |
| `stage_state_refs` | Yes | Stage state refs and checksums sorted by state kind then ID. |
| `snapshot_refs` | Yes when lakehouse reads affect output | `LakehouseSnapshotRef` and `DatasetVersionRef` refs. |
| `commit_refs` | Yes when writes occur | `LakehouseCommitRef` refs for attempted and successful writes. |
| `acceptance_report_refs` | Required for promotion | Acceptance report refs and checksums. |
| `activation_artifact_refs` | Required when activation-controlled artifacts affect output | Canonically sorted `ActivationControlledArtifactRef` rows and checksums. |
| `resolver_runtime_state_refs` | Required when identity output affects output or replay | Canonically sorted `070.IdentityDecision`, `070.IdentityReviewCase`, `070.ResolverExplanation`, `070.GraphCorrectionHandoff`, `070.ResolverActivationReport`, and `070.ResolverShadowRun` refs and checksums when present. These refs must also appear in `included_refs`; this field is not a parallel manifest mechanism. |
| `temporal_resolution_refs` | Required when source observation time affects gold, correction, graph projection, audit, or replay | Canonically sorted `080.TemporalObservationTimeResolution` refs and checksums. |
| `correction_snapshot_ref_sets` | Required for correction output | Old snapshot refs, new snapshot refs, table-set checksums, retention-protection refs, and mutable-ref rejection evidence. |
| `replay_sufficiency_check_refs` | Required for replay output | One `080.ReplayInputSufficiencyCheck` ref per replay output class. |
| `replay_equivalence_policy_refs` | Required for replay output | Output-class row refs and checksums for every replayed output class. |
| `graph_handoff_effect_refs` | Required when graph projection depends on correction effects | Canonically sorted `080` graph handoff effect refs and related authority refs. |
| `graph_profile_closure_refs` | Required when graph projection, graph apply, graph query, or graph rebuild affects output | Projection profile, projection rows, edge semantics, traversal classes, output eligibility, property policy, taxonomy mapping, query translation, schema profile, apply profile, lag policy, and validation refs. |
| `graph_serving_state_refs` | Required when graph query serving or rebuild promotion affects output | Derived-view state, backend schema fingerprint, graph apply result, graph rebuild manifest, graph index consistency check, page-token policy refs, and canonical graph output checksum refs. |
| `lifecycle_machine_refs` | Required when lifecycle behavior affects output, replay, activation, graph apply, watermark eligibility, or CI gating | Machine ID, version, owner, and machine checksum refs. |
| `lifecycle_transition_evidence_refs` | Required when lifecycle behavior affects output, replay, activation, graph apply, watermark eligibility, or CI gating | Canonically sorted transition evidence refs and checksums. |
| `lifecycle_state_artifact_refs` | Required when persisted lifecycle state artifacts affect output, replay, activation, graph apply, watermark eligibility, or CI gating | Subject refs, state artifact refs, and checksums. |
| `created_at` | Yes | RFC3339 UTC manifest creation time; excluded from manifest ID and included in checksum. |
| `manifest_checksum` | Yes | SHA-256 over `040.CanonicalJSON` excluding only `manifest_checksum`. |

`version_manifest_id` must be `vm_` plus SHA-256 lowercase hex over `040.CanonicalJSON` of `manifest_kind` and `included_refs`. `created_at` must not affect `version_manifest_id`.

### RequiredLifecycleMachineBindings

| Production lifecycle | Required owner | Machine ID | Transition-table location | Validation rows |
| --- | --- | --- | --- | --- |
| activation-controlled artifact lifecycle | `030` plus artifact owner | `030.ActivationControlledArtifactLifecycleMachine.v1` | `030`; owner guard rows in owner spec | `val-030-lifecycle-artifact-totality`, owner artifact rows |
| package-set activation | `100` | `100.PackageSetActivationLifecycleMachine.v1` | `100` | `val-100-lifecycle-package-activation` |
| production run execution | `030` | `030.ProductionRunExecutionLifecycleMachine.v1` | `030` | `val-030-lifecycle-production-run` |
| stage execution | `030` plus stage owner | `030.StageExecutionLifecycleMachine.v1` | `030`; owner event-derivation rows in owner spec | `val-030-lifecycle-stage-execution` |
| feed stage execution | `020`, `030` | `030.StageExecutionLifecycleMachine.v1` with `020.FeedStageLifecycleEventDerivation` | `020`, `030` | `val-030-lifecycle-feed-stage` |
| graph apply | `090` | `090.GraphApplyLifecycleMachine.v1` | `090` | `val-090-lifecycle-graph-apply-totality` |
| validation acceptance | `120` | `120.ValidationAcceptanceLifecycleMachine.v1` | `120` | `val-120-lifecycle-validation-acceptance` |

`ValidateSpecSet` must fail when any row in this table omits a concrete machine ID, transition-table owner, validation row, or owner guard/event-derivation location. Owner-specific errors must be used when available; generic lifecycle errors are fallback only.

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

| Subset mode | Allowed outputs | Forbidden outputs | Absence behavior | Cleanup behavior | Retraction behavior | Graph expiry behavior | Projection behavior | Watermark behavior | Validation requirement |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `validation_only` | validation reports, diagnostics | production records | forbidden | forbidden | forbidden | forbidden | forbidden | forbidden | negative authority rows |
| `feed_replay` | raw records and receipts | silver/gold/graph/API | forbidden | forbidden | forbidden | forbidden | forbidden | forbidden | feed replay rows |
| `graph_rebuild` | graph rebuild manifest and graph apply result | raw/silver/gold mutation | forbidden | forbidden | forbidden | allowed only when `060` permits requested effect | persisted delta replay only | forbidden unless explicit watermark behavior permits | rebuild equivalence rows |
| `package_canary` | canary output and activation evidence | current production output | forbidden | forbidden | forbidden | forbidden | canary only | forbidden | package canary rows |

A subset profile that omits watermark behavior must not advance a watermark. A subset profile that allows raw replay must not inherit full-run absence, cleanup, retraction, graph expiry, or watermark behavior.

### Activation artifact errors

| Error code | Emitted when |
| --- | --- |
| `ACTIVATION_ARTIFACT_INCOMPLETE` | A required activation-controlled artifact ref is missing required fields, invalid, unvalidated, out of scope, superseded, or otherwise incomplete after more specific owner codes are unavailable. |
| `DECLARED_DAG_SUBSET_PROFILE_MISSING` | A production subset request has no matching active `DeclaredDAGSubsetProfile`. |
| `DECLARED_DAG_SUBSET_SCOPE_MISMATCH` | The active subset profile does not cover the requested stage IDs, feed profiles, activation scope, or output class. |
| `DECLARED_DAG_SUBSET_OUTPUT_FORBIDDEN` | A subset stage attempts an output, absence effect, cleanup, retraction, graph expiry, projection, or watermark behavior forbidden by the active subset profile. |

### Lifecycle errors

| Error code | Emitted when |
| --- | --- |
| `LIFECYCLE_MACHINE_MISSING` | A required lifecycle machine definition or owner binding is absent. |
| `LIFECYCLE_MACHINE_INACTIVE` | A required lifecycle machine exists but is not active for the execution scope. |
| `LIFECYCLE_MACHINE_CHECKSUM_MISMATCH` | Machine checksum differs from the manifest, registry, or selected transition evidence. |
| `LIFECYCLE_EVENT_INVALID` | Event token is not in the selected machine's closed event set. |
| `LIFECYCLE_STATE_DERIVATION_FAILED` | Prior state cannot be derived from owner-declared artifacts and persisted transition evidence. |
| `LIFECYCLE_ILLEGAL_TRANSITION` | Selected state/event pair is illegal and no owner-specific illegal-transition code exists. |
| `LIFECYCLE_IDEMPOTENCY_CONFLICT` | Same event idempotency key is reused with different input checksums. |
| `LIFECYCLE_TRANSITION_EVIDENCE_INCOMPLETE` | Required transition evidence field, checksum, or manifest ref is missing. |
| `LIFECYCLE_PARENT_CHILD_STATE_INVALID` | Parent transition depends on invalid, missing, or in-memory child state. |
| `LIFECYCLE_GUARD_REFUSED` | A declared guard refuses the transition and no owner-specific refusal code exists. |

### Acceptance Criteria

| ID | Criterion |
| --- | --- |
| `030-API-EXPORT-MANIFEST-AC-001` | API/export output fails with `VERSION_MANIFEST_INCOMPLETE` when request checksum, endpoint outcome matrix checksum, state-label mapping checksum, generated error-registry checksum, authorization decision ref, redaction context checksum, page-token policy ref, audit event ref, owner error ref, or required derived-view state ref is missing. |
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
| `030-FEED-CLOSURE-AC-001` | The closed `artifact_class` registry includes feed category closure row sets, declared subset profiles, lakehouse feed completeness profiles, supplier visibility profiles, control result mappings, source history retention profiles, absence policies, and projection watermark policies. |
| `030-FEED-CLOSURE-AC-002` | A production subset request without a matching active `DeclaredDAGSubsetProfile` fails before stage execution. |
| `030-FEED-CLOSURE-AC-003` | A subset profile that permits raw replay but omits watermark behavior does not advance a watermark. |
| `030-FEED-CLOSURE-AC-004` | Same DAG bytes, same requested subset, same subset profile, and same activation refs produce byte-identical stage order and manifest refs. |
| `030-IDENTITY-MANIFEST-AC-001` | Identity output fails with `VERSION_MANIFEST_INCOMPLETE` when any consulted resolver artifact ref is omitted from `VersionManifest`. |
| `030-IDENTITY-MANIFEST-AC-002` | Resolver artifact checksum mismatch, inactive artifact, or out-of-scope activation ref fails before identity output or replay promotion. |
| `030-IDENTITY-MANIFEST-AC-003` | Resolver explanation checksum refs and split graph correction handoff refs are included for identity replay when applicable. |
| `030-IDENTITY-MANIFEST-AC-004` | `ResolverActivationReport` and `ResolverShadowRun` refs are manifest-included when activation, shadow, canary, or promotion uses them. |

| `030-OCSF-MAP-ARTIFACT-AC-001` | Silver output fails with `VERSION_MANIFEST_INCOMPLETE` when any output-affecting OCSF mapping row set, enum rule set, base-event field policy set, profile-resolution manifest, source-extension rule set, or observation-type validation matrix ref is missing from `VersionManifest`. |
| `030-SOURCE-CLOSURE-MANIFEST-AC-001` | Absence-sensitive output fails with `VERSION_MANIFEST_INCOMPLETE` when any consulted `060` row-set ref is omitted. |
| `030-SOURCE-CLOSURE-MANIFEST-AC-002` | Same refs and same row bytes produce the same `version_manifest_id`; changing one closure row checksum changes the manifest checksum or blocks output. |
| `030-LIFECYCLE-AC-001` | All required lifecycle machines have closed states and closed events. |
| `030-LIFECYCLE-AC-002` | Every state/event pair selects exactly one transition row. |
| `030-LIFECYCLE-AC-003` | Illegal transitions emit evidence, mutate no state, and write no production output. |
| `030-LIFECYCLE-AC-004` | Idempotent replay is byte-identical or fails with a deterministic idempotency conflict. |
| `030-LIFECYCLE-AC-005` | Canary and shadow outputs never advance current production, last-known-good, or watermark state. |
| `030-LIFECYCLE-AC-006` | Lifecycle transition evidence appears in `VersionManifest` whenever lifecycle behavior is output-affecting. |

| `030-TEMPORAL-MANIFEST-AC-001` | Missing temporal resolution ref, temporal policy ref, or temporal resolution checksum fails before gold output. |
| `030-CORRECTION-MANIFEST-AC-001` | Missing old snapshot ref, new snapshot ref, table-set checksum, or retention-protection ref fails before correction output. |
| `030-REPLAY-MANIFEST-AC-001` | Missing replay equivalence row ref or replay sufficiency check ref fails before replay output. |
| `030-GRAPH-HANDOFF-MANIFEST-AC-001` | Graph handoff refs appear in `VersionManifest` whenever graph projection depends on correction effects. |
| `030-GRAPH-PROFILE-MANIFEST-AC-001` | Graph projection, apply, query, rebuild, or replay fails with `VERSION_MANIFEST_INCOMPLETE` when any graph profile closure artifact ref is omitted. |
| `030-GRAPH-PROFILE-MANIFEST-AC-002` | Edge semantics, output eligibility, query translation, backend taxonomy mapping, property policy, and derived-view lag refs are manifest-included whenever they affect graph output or serving. |
| `030-GRAPH-REBUILD-MANIFEST-AC-001` | Graph rebuild promotion fails unless manifest refs include rebuild manifest, index consistency check, schema fingerprint, and canonical output checksum inputs. |
| `030-IDENTITY-SPLIT-GRAPH-HANDOFF-AC-001` | Identity split graph handoff output includes `070.GraphCorrectionHandoff` refs and resolver explanation checksum refs. |
| `030-PACKAGE-ARTIFACT-CLASS-AC-001` | Every package activation-controlled artifact class required by `100` appears in the closed `artifact_class` registry. |
| `030-PACKAGE-VM-AC-001` | Package activation output fails with `PACKAGE_VERSION_MANIFEST_INCOMPLETE` or `VERSION_MANIFEST_INCOMPLETE` when any required package-set, release, trust, repository, SBOM, provenance, compatibility, rollback, quarantine, emergency, health, validation, lifecycle, or approval ref is missing from `VersionManifest.included_refs`. |

## Definition of Done

| ID | Criterion |
| --- | --- |
| `030-AC-001` | The same DAG, package set, inputs, stage state, and requested subset produce the same stage order and output classes. |
| `030-AC-002` | A forbidden output class is rejected before commit and before downstream stage execution. |
| `030-AC-003` | Every production-affecting lifecycle has closed states, closed events, total transitions, illegal-transition behavior, and validation criteria. |
| `030-AC-004` | Every output-affecting state record is hashable, replayable, and included in `VersionManifest`. |
| `030-AC-005` | Production replay rejects missing, mutable-only, checksum-mismatched, or retention-ineligible manifest refs before writing output. |
| `030-AC-006` | Declared subset execution fails closed for absence, cleanup, retraction, graph expiry, projection, and watermark effects unless the active subset profile and `060` effect gates both permit the requested behavior. |
| `030-AC-007` | Identity output replay is incomplete unless every output-affecting resolver artifact, runtime state ref, explanation checksum, and split handoff ref is manifest-included. |

## Open Questions

Open questions marked `TODO:` block authoritative status for the affected contract. A downstream implementation must not resolve a `TODO:` by inference.
