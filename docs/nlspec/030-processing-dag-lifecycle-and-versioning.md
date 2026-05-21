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
- `ObservabilityInstrumentationProfile`
- `TelemetrySignalPolicy`
- `TraceContextPolicy`
- `TelemetryCorrelationPolicy`
- `TelemetryRuntimeState`
- `TelemetryReplayExclusionPolicy`
- `CoreRecordSchema`
- `CoreRecordValidationAlgorithm`
- `CoreRecordChecksumPolicy`

## Exports

- `ProcessingStageDAG`
- `StageInstrumentationHandoff`
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
- `RunLockLeasePolicy`
- `RunLockRecord`
- `RunLockOperationIdempotencyKey`
- `RunLockOperationEvidence`
- `RunLockRecoveryEvidence`
- `RunLockCommitGuard`
- `AcquireRunLockSet`
- `HeartbeatRunLockSet`
- `AssertRunLockHeldBeforeCommit`
- `RecoverStaleRunLock`
- `ReleaseRunLockSet`
- `MarkRunLockLost`
- `DeclaredDAGSubsetProfile`
- `VersionManifest`
- `ExecuteProcessingStageDAG`
- `ValidateLifecycleStateMachine`
- `ActivationControlledArtifactRef`
- `StructuredInputRepositoryProfile`
- `StructuredInputRepositorySnapshot`
- `StructuredInputChangeProposal`
- `StructuredInputChangeProposalLifecycleMachine`
- `ResolveStructuredInputRepositorySnapshot`
- `ValidateStructuredInputRepositorySnapshot`

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
| `analysis` | `AnalysisFinding`, `AnalysisMetric`, `RiskAcceptanceRecord` only. |
| `enrichment` | `ThreatIntelEnrichmentRecord`, enrichment diagnostics, and required `StageStateRecord` only. |
| `lineage` | `RunDatasetIOContract`, lineage diagnostics, and required `StageStateRecord` only. |
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

### StageInstrumentationHandoff

Every production `ProcessingStageDAG` stage execution must create a telemetry correlation context when an active `140.ObservabilityInstrumentationProfile` enables traces or structured logs for the stage class.

Telemetry context must not affect stage ordering, output permission, failure-isolation precedence, lifecycle transition selection, run lock acquisition, output record IDs, output checksums, replay equivalence, watermarks, graph apply, or package activation.

A stage span or structured log may reference only fields allowed by `140.TelemetryAttributePolicy`. The default allowed stage correlation attributes are `stage_id`, `stage_class`, `run_id`, `version_manifest_ref`, `package_set_ref`, selected activation artifact refs, owner error code, lifecycle transition evidence ref, and validation row ref.

A stage span, metric, or structured log must not contain raw payload bytes, private source bindings, credentials, backend-generated IDs, source-native identifiers, canonical entity IDs, graph backend IDs, package payload bytes, provider-native query text, or unredacted user/source identifiers unless `140.TelemetryRedactionPolicy` and `110.RedactionPolicy` both permit the exact data class.

Telemetry emission failure must emit or update `140.TelemetryRuntimeState` according to `140.TelemetryExporterProfile`; it must not change the stage's domain result except through `110.OperationalHealthStatus` when health is requested.

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
lock_acquiring
locks_acquired
running
lock_lost
succeeded
released
failed
aborted
```

Events:

```text
preflight_passed
preflight_failed
run_lock_acquisition_requested
run_lock_acquired
run_lock_conflict
run_lock_acquisition_failed
run_lock_heartbeat_recorded
run_lock_heartbeat_uncertain
run_lock_lost
run_lock_stale_recovered_by_other_run
run_lock_released
start_run
stage_succeeded
stage_failed_isolated
stage_failed_nonisolated
complete
abort
replay_same_event
```

Required transition rows:

| Prior state | Event | Next state | Required behavior |
| --- | --- | --- | --- |
| `planned` | `preflight_passed` | `preflighted` | Persist preflight evidence. |
| `planned` | `preflight_failed` | `failed` | No production output. |
| `preflighted` | `run_lock_acquisition_requested` | `lock_acquiring` | Persist acquisition-start evidence. |
| `lock_acquiring` | `run_lock_acquired` | `locks_acquired` | Persist complete lock set and fencing token set. |
| `lock_acquiring` | `run_lock_conflict` | `failed` | Write no production output and advance no watermark. |
| `lock_acquiring` | `run_lock_acquisition_failed` | `failed` | Release partial keys if any; write no production output. |
| `locks_acquired` | `start_run` | `running` | Persist run-start evidence. |
| `running` | `stage_succeeded` | `running` | Continue eligible downstream stages. |
| `running` | `stage_failed_isolated` | `running` | Continue only for declared isolated outputs. |
| `running` | `stage_failed_nonisolated` | `failed` | Stop downstream stages. |
| `running` | `run_lock_heartbeat_recorded` | `running` | Persist same-state heartbeat evidence. |
| `running` | `run_lock_heartbeat_uncertain` | `running` | Block output commits until assertion succeeds. |
| `running`, `locks_acquired`, or `lock_acquiring` | `run_lock_lost` | `lock_lost` | Stop before next output commit; no downstream stage may start. |
| `running` | `run_lock_stale_recovered_by_other_run` | `lock_lost` | Old holder is fenced; diagnostics only. |
| `running` or `succeeded` | `run_lock_released` | `released` | Persist release evidence after commit refs, lifecycle results, and manifest refs. |
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
blocked_lock_unverified
blocked_lock_lost
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
lock_assertion_uncertain
lock_lost_before_commit
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
| `running` | `lock_assertion_uncertain` | `blocked_lock_unverified` | No production output until assertion succeeds or timeout routes to failure. |
| `running` | `lock_lost_before_commit` | `blocked_lock_lost` | No production output; parent run transitions to `lock_lost`. |
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
| `run_lock_commit_guard_refs` | Canonically sorted refs to `RunLockCommitGuard` records required by production commits; default `[]` only for non-production or no-output stages. |
| `lock_loss_evidence_ref` | Required when the stage terminal state is `blocked_lock_lost`; otherwise null. |
| `output_refs` | Canonically sorted output refs; default `[]`. |
| `blocked_effects` | Array of blocked effects such as `absence`, `cleanup`, `retraction`, `graph_expiry`, or `watermark`; default `[]`. |
| `error_refs` | Owner-specific errors; default `[]`. |
| `version_manifest_id` | Required for production output. |

## Run Locks

`RunLockSet` must be acquired before any production output write. Lock keys must cover source/feed scope, completeness scope, coverage scope, projection scope, graph apply target, package set, and requested output class when concurrent writes could change observable output.

The lock store must provide atomic compare-and-set per lock key, authoritative lock-store time, and per-key monotonic fencing tokens. If any of these properties is unavailable, stale recovery is disabled and concurrent production writes must fail closed before mutation with the most specific run-lock error.

Lock operation evidence must be persisted before any production-visible output that depends on the operation. Lock records, evidence records, commit guards, and lock-loss records must serialize through `040.CanonicalJSON` when their bytes affect IDs, checksums, replay equivalence, validation output, or audit evidence.

### RunLockLeasePolicy

`RunLockLeasePolicy` is the materialized timing contract for a `RunLockSet`. Defaults must materialize before the `RunLockSet` checksum. Unknown fields fail with `RUN_LOCK_TIMING_INVALID` unless the policy schema declares an extension map.

| Field | Default | Bounds | Rule |
| --- | ---: | ---: | --- |
| `lease_ttl_seconds` | `300` | `60..3600` | A lock is stale only after expiry plus grace under lock-store time. |
| `heartbeat_interval_seconds` | `60` | `10..300` | Must be `<= floor(lease_ttl_seconds / 3)`. |
| `stale_recovery_grace_seconds` | `30` | `0..120` | Must be `<= heartbeat_interval_seconds`. |
| `lock_operation_timeout_seconds` | `15` | `1..60` | Applies to acquire, heartbeat, release, assert, and recover. |
| `commit_guard_min_remaining_lease_seconds` | `30` | `5..300` | Before output commit, refresh or fail when remaining lease is below this value. |
| `lock_time_source` | `lock_store_time` | closed enum | Runner-local wall clock is forbidden for stale decisions. |
| `fencing_required` | `true` | boolean | Production output requires fencing token proof. |

`RUN_LOCK_TIMING_INVALID` must be emitted before lock mutation when any bound, ratio, enum, boolean, or unknown-field rule fails.

### RunLockSet schema additions

A `RunLockSet` that can gate production output must include the following fields in addition to the lock-key list derived by `RunLockKeyDerivation`.

| Field | Required rule |
| --- | --- |
| `run_lock_set_id` | Deterministic ID over run, attempt, environment, requested output class, sorted lock keys, lease policy checksum, and version manifest input checksum. |
| `run_id` | Required run identifier. |
| `run_attempt_id` | Required attempt identifier; retry attempts must not reuse another attempt's lock owner identity. |
| `lock_owner_id` | Stable owner identity for the attempt; raw private scheduler IDs are forbidden in public artifacts. |
| `environment_id` | Required environment scope; stale recovery may occur only within the same environment. |
| `lease_policy` | Materialized `RunLockLeasePolicy`. |
| `lock_records[]` | One `RunLockRecord` per lock key, sorted lexically by `lock_key`. |
| `fencing_token_set` | Token set keyed by `lock_key`; every token must come from the lock store and be monotonic per key. |
| `last_heartbeat_at` | Lock-store timestamp for the most recent confirmed heartbeat across the set. |
| `lease_expires_at` | Earliest lock-store expiry across records in the set. |
| `heartbeat_sequence` | Starts at `0` at acquisition and increments by `1` per successful heartbeat operation. |
| `operation_evidence_refs[]` | Canonically sorted refs to acquire, heartbeat, release, assert, and failure evidence. |
| `recovery_evidence_refs[]` | Canonically sorted refs to stale recovery evidence; empty when no recovery occurred. |
| `commit_guard_refs[]` | Canonically sorted refs to commit guards for every production commit. |
| `lock_loss_evidence_ref` | Required when the set status is `lost`, `expired`, or old-holder `stale_recovered`. |
| `status` | One closed `RunLockSet.status` token. |

Closed `RunLockSet.status` values are:

```text
requested
acquiring
acquired
conflict
released
lost
expired
stale_recovered
failed
```

### RunLockRecord

A `RunLockRecord` represents exactly one lock key.

| Field | Required rule |
| --- | --- |
| `lock_key` | Exact key from `RunLockKeyDerivation`. |
| `lock_scope_tuple_checksum` | SHA-256 over the canonical scope tuple. |
| `requested_output_class` | Exact output class guarded by this key. |
| `run_id` | Current owning run. |
| `run_attempt_id` | Current owning run attempt. |
| `lock_owner_id` | Current owner identity. |
| `lock_record_version` | Monotonic mutation version from the lock store or canonical equivalent. |
| `fencing_token` | Monotonic token for the key; new owner token must be greater than prior token. |
| `heartbeat_sequence` | Last successful heartbeat sequence for this record. |
| `acquired_at` | Lock-store acquisition timestamp. |
| `last_heartbeat_at` | Lock-store timestamp for the latest successful heartbeat. |
| `lease_expires_at` | Lock-store timestamp at which the lease expires before grace. |
| `status` | Closed token: `acquired`, `released`, `lost`, `expired`, `stale_recovered`, or `failed`. |
| `record_checksum` | SHA-256 over canonical record bytes after defaults materialize. |

### RunLockOperationIdempotencyKey

`RunLockOperationIdempotencyKey` is computed over the following canonical fields:

```text
operation_kind
environment_id
run_id
run_attempt_id
run_lock_set_id
sorted lock_keys
version_manifest_id
version_manifest_input_checksum
requested_output_class
operation_sequence
```

Idempotency rules:

- Same key plus same input checksum returns byte-identical `RunLockOperationEvidence`.
- Same key plus different input checksum fails with `RUN_LOCK_IDEMPOTENCY_CONFLICT` before mutation.
- New heartbeat sequence requires a new key.
- Retry of the same heartbeat sequence returns byte-identical evidence and must not create a duplicate mutation.

### RunLockOperationEvidence

| Field | Required rule |
| --- | --- |
| `operation_evidence_id` | Deterministic ID over operation kind, idempotency key, input checksum, result, and new lock record checksums. |
| `operation_kind` | Closed token: `acquire`, `heartbeat`, `assert`, `recover`, `release`, or `mark_lost`. |
| `idempotency_key` | `RunLockOperationIdempotencyKey`. |
| `input_checksum` | SHA-256 over canonical request bytes. |
| `run_id` | Current run. |
| `run_attempt_id` | Current attempt. |
| `run_lock_set_id` | Guarded lock set. |
| `lock_keys` | Sorted lock keys considered by the operation. |
| `prior_lock_record_checksums` | Sorted prior record checksums read before decision. |
| `new_lock_record_checksums` | Sorted new record checksums after successful mutation; empty on no-mutation failure. |
| `fencing_token_set` | Token proof after successful acquire, heartbeat, assert, recover, or release. |
| `lock_store_time_at_decision` | Lock-store timestamp used for the decision. |
| `result` | Closed token: `succeeded`, `failed`, `conflict`, `no_op`, or `uncertain`. |
| `error_code` | Null on success; otherwise most specific `030` run-lock code. |
| `version_manifest_id` | Required for production output and validation-visible lock evidence. |
| `evidence_checksum` | SHA-256 over canonical evidence bytes. |

### RunLockRecoveryEvidence

| Field | Required rule |
| --- | --- |
| `recovery_evidence_id` | Deterministic ID over recovered key, prior checksum, new token, CAS precondition checksum, and result. |
| `recovered_lock_key` | Exact recovered lock key. |
| `prior_run_id` | Prior owning run. |
| `prior_run_attempt_id` | Prior owning attempt. |
| `prior_lease_expires_at` | Prior record expiry under lock-store time. |
| `prior_last_heartbeat_at` | Prior heartbeat time. |
| `prior_fencing_token` | Prior fencing token. |
| `prior_lock_record_checksum` | Prior record checksum included in recovery proof. |
| `new_run_id` | New owning run. |
| `new_run_attempt_id` | New owning attempt. |
| `new_fencing_token` | Token strictly greater than prior token. |
| `stale_predicate_inputs` | Canonical object containing every predicate input. |
| `cas_precondition_checksum` | Checksum over the exact compare-and-set precondition. |
| `cas_result` | Closed token: `succeeded`, `failed`, or `not_attempted`. |
| `old_holder_effect` | One closed `old_holder_effect` token. |
| `version_manifest_id` | Required for production output and validation-visible recovery. |
| `evidence_checksum` | SHA-256 over canonical recovery evidence bytes. |

Closed `old_holder_effect` values are:

```text
fenced
already_released
recovery_rejected
```

### Stale recovery predicate

A lock may be recovered only when all predicate rows are true at the atomic mutation boundary:

1. `lock_store_now >= prior.lease_expires_at + stale_recovery_grace_seconds`.
2. The contender's compare-and-set precondition proves no newer heartbeat or mutation occurred after reading the prior record.
3. Recovery is for the exact same `lock_key`.
4. `environment_id` matches.
5. Requested output class and scope are compatible.
6. Prior lock record checksum is included in `RunLockRecoveryEvidence`.
7. New fencing token is strictly greater than prior fencing token.
8. Recovery is one atomic compare-and-set mutation.
9. Recovery evidence is persisted before recovered output writes.

Failure emits `RUN_LOCK_STALE_RECOVERY_NOT_ELIGIBLE` or `RUN_LOCK_STALE_RECOVERY_FAILED`, acquires no partial lock set, writes no production output, and advances no watermark.

### RunLockCommitGuard

`RunLockCommitGuard` is the required pre-commit assertion for production output governed by a run lock.

| Field | Required rule |
| --- | --- |
| `commit_guard_id` | Deterministic ID over lock set, output commit scope, sorted lock keys, fencing token set, assertion time, and result. |
| `run_lock_set_id` | Guarded lock set. |
| `output_commit_scope` | Canonical commit scope for the attempted production write. |
| `lock_keys[]` | Sorted keys asserted before commit. |
| `fencing_token_set` | Token proof used by the output store or commit guard. |
| `assertion_time` | Lock-store time of assertion. |
| `lease_remaining_seconds` | Non-negative integer seconds remaining under lock-store time. |
| `operation_evidence_ref` | Ref to the assertion evidence record. |
| `result` | Closed token: `succeeded`, `failed`, or `blocked`. |
| `error_code` | Null on success; otherwise `RUN_LOCK_LOST`, `RUN_LOCK_COMMIT_GUARD_MISSING`, `RUN_LOCK_FENCING_TOKEN_STALE`, or `RUN_LOCK_ASSERTION_FAILED`. |
| `checksum` | SHA-256 over canonical guard bytes. |

Every production output commit must call `AssertRunLockHeldBeforeCommit`. If ownership or token cannot be proven, the implementation must fail before commit with `RUN_LOCK_LOST`, `RUN_LOCK_COMMIT_GUARD_MISSING`, `RUN_LOCK_FENCING_TOKEN_STALE`, or `RUN_LOCK_ASSERTION_FAILED`.

### Run-lock algorithms

```text
AcquireRunLockSet(request, lease_policy):
1. Materialize `RunLockLeasePolicy` defaults and validate bounds.
2. Compute sorted lock keys and `run_lock_set_id`.
3. Build `RunLockOperationIdempotencyKey` with `operation_kind = acquire`.
4. Read lock-store time; if unavailable, persist failed evidence and return `RUN_LOCK_STORE_TIME_UNAVAILABLE`.
5. Compare-and-set every key as one all-or-nothing acquisition; when the store cannot mutate all keys atomically, release every acquired key before returning.
6. For active unexpired conflicts, persist `RunLockOperationEvidence` with `RUN_LOCK_CONFLICT` and write no output.
7. For eligible stale records, call `RecoverStaleRunLock` and require successful recovery evidence before acquisition succeeds.
8. Persist complete lock records, fencing token set, and operation evidence before production output.
9. Return acquired `RunLockSet` or a failed set with no partial ownership.

HeartbeatRunLockSet(run_lock_set):
1. Use lock-store time and the next heartbeat sequence.
2. Use a new idempotency key for the sequence.
3. Compare-and-set each owned lock record only when owner, attempt, token, and prior record checksum match.
4. Extend `lease_expires_at` by `lease_ttl_seconds` from lock-store time.
5. Persist heartbeat evidence before the next production commit.
6. On timeout or uncertain result, emit `RUN_LOCK_HEARTBEAT_UNCERTAIN` and block output until `AssertRunLockHeldBeforeCommit` succeeds.
7. On failed proof of ownership, call `MarkRunLockLost`.

AssertRunLockHeldBeforeCommit(run_lock_set, output_commit_scope):
1. Read lock-store time and current lock records.
2. Verify owner, attempt, environment, sorted lock keys, record checksums, and fencing tokens.
3. If remaining lease is below `commit_guard_min_remaining_lease_seconds`, refresh with heartbeat or fail before commit.
4. Persist `RunLockCommitGuard` and assertion operation evidence.
5. Return success only when the output store can verify the same fencing token set for the commit scope.
6. Return `RUN_LOCK_COMMIT_GUARD_MISSING`, `RUN_LOCK_FENCING_TOKEN_STALE`, `RUN_LOCK_LOST`, or `RUN_LOCK_ASSERTION_FAILED` before mutation when proof fails.

RecoverStaleRunLock(prior_lock, contender_lock_set):
1. Read lock-store time and validate every stale recovery predicate.
2. If any predicate fails, persist recovery evidence with `recovery_rejected` and return `RUN_LOCK_STALE_RECOVERY_NOT_ELIGIBLE`.
3. Perform one compare-and-set mutation from the prior checksum to the contender record with a strictly greater fencing token.
4. If CAS fails, persist recovery evidence and return `RUN_LOCK_STALE_RECOVERY_FAILED`.
5. Persist recovery evidence before recovered output writes.
6. Return the recovered record and mark old-holder effect as `fenced` or `already_released`.

ReleaseRunLockSet(run_lock_set, release_reason):
1. Use lock-store time and an idempotent release key.
2. Release only records still owned by the same run, attempt, and fencing token set.
3. Persist release evidence after required commit refs, lifecycle results, and manifest refs are persisted.
4. Return byte-identical evidence for identical release retries.

MarkRunLockLost(run_lock_set, loss_reason):
1. Persist lock-loss evidence with current lock record checksums when available.
2. Transition the run to `lock_lost` before the next output commit.
3. Stop downstream stages.
4. Forbid watermark, graph apply promotion, package activation, absence, cleanup, retraction, graph expiry, table maintenance, last-known-good, and derived-view advancement for the affected run.
```

## Version Manifest Contract

`VersionManifest` must include every output-affecting profile, policy, package artifact, package-set manifest, source/feed config hash, stage state hash, lakehouse ref, completeness receipt, coverage assertion, schema artifact, temporal policy, resolver profile, graph profile, graph apply result, lifecycle machine definition, lifecycle machine checksum, lifecycle transition evidence ref, lifecycle state artifact ref, and acceptance report that can affect output or replay.

A production output without a complete immutable `VersionManifest` must fail with `VERSION_MANIFEST_INCOMPLETE`. `activation_artifact_refs` must be present in `VersionManifest.included_refs` for every output-affecting activation-controlled artifact.

When a run emits any `040` record, `VersionManifest` must include `core_record_schema_registry_checksum`, `core_record_schema_versions`, `core_record_checksum_policy_version`, `core_record_validation_result_refs`, and `core_record_error_refs` for every rejected record. Canary and shadow records may use null `version_manifest_id` only when this spec declares the execution mode and `040.CommonRecordHeader` permits it.

### EvidenceRefArtifactClassManifestHandoff

When any output emits `EvidenceRef`, `VersionManifest` must include the active `040.EvidenceArtifactIdKindRegistry` checksum, every consulted `040.EvidenceArtifactClassRegistry` row-set ref, the referenced artifact checksum, and the owner artifact refs required by the referenced artifact class.

`ActivationControlledArtifactRef` remains the canonical reference shape for activation artifacts. `EvidenceRef.artifact_id.kind = activation_artifact_ref` must not bypass artifact lifecycle, checksum, validation refs, activation scope, package-set membership when required, or artifact-class policy. Missing registry checksum or artifact checksum fails with `VERSION_MANIFEST_INCOMPLETE` or the most specific owner artifact error before evidence ref creation.

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
structured_input_repository_profile
structured_input_repository_access_policy
structured_input_repository_redaction_policy
structured_input_validation_matrix
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
source_authority_closure_matrix_row_set
external_schema_authority_signal_mapping_row_set
source_staleness_policy
coverage_dimension_profile
lakehouse_feed_completeness_profile
supplier_collection_visibility_profile
control_result_mapping_row_set
source_history_retention_profile
absence_derivation_policy
projection_watermark_policy
progress_signal_policy
observability_instrumentation_profile
telemetry_signal_policy
trace_context_policy
telemetry_correlation_policy
metric_instrument_catalog
telemetry_attribute_policy
telemetry_redaction_policy
telemetry_exporter_profile
telemetry_health_mapping_policy
telemetry_replay_exclusion_policy
resolver_profile
identifier_evidence_class_row_set
identifier_scope_row_set
asset_generation_boundary_row_set
identity_hard_blocker_row_set
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
graph_backend_selection_policy
graph_provider_capability_matrix
graph_provider_error_mapping_profile
graph_backend_taxonomy_mapping_profile
graph_read_model_schema_profile
graph_query_translation_profile
graph_apply_profile
derived_view_lag_policy
graph_rebuild_manifest_profile
analysis_rule_bundle
analysis_rule_row_set
rule_graph_compatibility_matrix
derivation_rule_bundle
derived_graph_edge_rule_row_set
threat_intel_enrichment_profile
threat_intel_distribution_mapping_policy
threat_intel_artifact_ref_policy
lineage_facet_mapping_policy
artifact_class_policy_row_set
evidence_artifact_class_row_set
registry_governance_artifact
registry_custom_property_schema
registry_classification_policy
analysis_output_replay_policy_row_set
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

### Source-authority closure artifact class mapping

| Owner contract | Canonical artifact class | Notes |
| --- | --- | --- |
| `060.SourceAuthorityProfileRowSet` | `source_authority_row_set` | Existing token retained. |
| `060.SourceAuthorityClosureMatrixRowSet` | `source_authority_closure_matrix_row_set` | Validation view only; not source authority by itself. |
| `060.ExternalSchemaAuthoritySignalMappingRowSet` | `external_schema_authority_signal_mapping_row_set` | Required only when external schema signals are consulted. |

A `SourceAuthorityClosureMatrix` validation result cannot satisfy `VersionManifest` completeness unless all underlying activation-controlled row refs and checksums are also present. A deterministic block validation ref may satisfy the closure-row-set position only for the exact blocked category, dataset, fact, predicate, scope, and requested effect.

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

## Structured Input Repository Contract

`StructuredInputRepositoryProfile` defines the stable interface for Git-backed authoring repositories that maintain schema, mapping, profile, row-set, validation, policy, resolver, graph, analysis, registry, telemetry, and other activation-controlled structured inputs. Concrete repository profile rows are activation-controlled artifacts. A repository profile grants authoring scope only; it does not grant source authority, graph authority, package activation, production approval, rollback eligibility, or system-of-record status.

### StructuredInputRepositoryProfile schema

| Field | Required | Default or omission behavior | Rule |
| --- | ---: | --- | --- |
| `repository_profile_id` | Yes | none | Stable ID scoped to `030`; must not encode a private route or credential. |
| `repository_role` | Yes | none | Closed enum: `authoring_origin`, `registered_remote`, `mirror`. |
| `allowed_artifact_classes` | Yes | none | Non-empty set of closed `030.ActivationControlledArtifactRef.artifact_class` tokens that may be authored from the repository. |
| `allowed_path_roots` | Yes | none | Non-empty normalized repo-relative path prefixes. Empty, absolute, backslash, NUL, `.`, `..`, and duplicate roots are rejected. |
| `protected_ref_policy` | Yes | none | Names refs that may be selected for review. It must not make any ref an activation target. |
| `merge_policy_ref` | Yes | none | Ref to review/merge policy owned by `110` or governance owner. |
| `review_requirement_ref` | Yes | none | Required before proposal merge. Omission blocks merge and materialization. |
| `public_private_classification` | Yes | none | Closed enum: `public_authoring`, `private_authoring`, `mixed_requires_redaction`. |
| `snapshot_selection_policy` | Yes | none | Defines selected refs and path filters; output must resolve to exact commit SHA and tree hash. |
| `force_push_policy` | Yes | `invalidate_unless_exact_commit_tree_remains_selected` | Prior validation is invalid when a ref rewrite changes selected commit or tree. |
| `path_normalization_policy` | Yes | `strict_repo_relative_regular_files_only` | Applies the rejection list in this contract. |
| `validation_matrix_refs` | Yes | none | Non-empty refs to `120.StructuredInputRepositoryValidationMatrix` rows. |
| `activation_scope` | Yes | none | Vendor-neutral scope in which repository-authored content may be materialized. |
| `lifecycle_status` | Yes | none | Production-affecting use requires `active`. |

### StructuredInputRepositorySnapshot schema

| Field | Required | Default or omission behavior | Rule |
| --- | ---: | --- | --- |
| `repository_snapshot_id` | Yes | none | Deterministic ID over repository profile ref, resolved commit SHA, tree hash, selected paths, file manifest checksum, and snapshot content checksum. |
| `repository_profile_ref` | Yes | none | Active `StructuredInputRepositoryProfile` ref. |
| `repository_identity_ref` | Yes | none | Redacted repository identity or checksum; raw URL exposure is governed by `110`. |
| `resolved_commit_sha` | Yes | none | Exact commit SHA. Branches, tags, PR refs, repository URLs, and default branch names are forbidden substitutes. |
| `tree_hash` | Yes | none | Exact tree hash for the resolved commit. |
| `parent_commit_shas` | Yes | `[]` only for root commit | Canonically sorted by commit graph parent order when available. |
| `selected_ref_name` | No | null | Diagnostic only; excluded from activation authority. |
| `selected_paths` | Yes | none | Canonically sorted normalized repo-relative paths or path roots. |
| `file_manifest` | Yes | none | Non-empty unless the selected artifact class explicitly permits an empty row set. |
| `file_manifest_checksum` | Yes | none | SHA-256 over canonical file manifest bytes. |
| `snapshot_content_checksum` | Yes | none | SHA-256 over normalized path, file mode, file size, SHA-256, text/binary class, and artifact-class owner for every selected file. |
| `public_private_classification` | Yes | none | Must be compatible with profile and `010` public/private rules. |
| `validation_run_refs` | Yes | none | Non-empty refs to exact-snapshot validation runs before materialization. |
| `created_from_change_proposal_ref` | No | null | Review provenance only. |
| `force_push_observation_ref` | No | null | Required when selected ref was rewritten after validation. |
| `lifecycle_status` | Yes | none | `validated` permits validation and materialization; it does not activate production behavior. |

File manifest entries must include normalized repo-relative path, file mode, byte size, SHA-256, canonical text/binary classification, and declared artifact-class owner. File mode is restricted to regular file unless a future accepted owner row explicitly permits another file type.

Path normalization must reject absolute paths, backslashes, NUL, empty segments, `.`, `..`, duplicate normalized paths, symlinks, non-regular files, and path collisions after any flat-share decoding. Rejection must occur before checksum computation, validation output, materialization, package release creation, API response materialization, audit output, or telemetry export.

### StructuredInputChangeProposal lifecycle

`StructuredInputChangeProposal` is review state only. It must not activate production behavior.

| State | Meaning | Production effect |
| --- | --- | --- |
| `draft` | Change is authoring-local or pre-review. | none |
| `proposed` | Change is submitted for review. | none |
| `validated` | Exact proposed snapshot passed validation. | none |
| `merged` | Change reached an allowed repository ref. | none |
| `revalidated` | Exact merge commit and tree passed validation. | none |
| `materialized` | Exact revalidated snapshot produced immutable materialization output. | none until `100` activation gates pass |
| `rejected` | Review or validation rejected the change. | none |
| `abandoned` | Authoring stopped without merge. | none |

A transition to `merged` must not activate production behavior. A transition to `materialized` requires exact merge-commit revalidation and a `100.StructuredInputMaterializationResult`.

### ResolveStructuredInputRepositorySnapshot

```text
ResolveStructuredInputRepositorySnapshot(profile, selected_ref, selected_paths):
1. Validate `StructuredInputRepositoryProfile` lifecycle, activation scope, allowed artifact classes, path roots, public/private classification, and validation matrix refs.
2. Reject repository URL, branch name, tag name, pull request ref, default branch name, hook result, or commit timestamp as an activation target.
3. Resolve the allowed selected ref to exact `resolved_commit_sha` and `tree_hash`.
4. Normalize selected paths and reject invalid paths according to this contract.
5. Enumerate selected regular files in ascending lexical normalized path order.
6. Compute each file SHA-256 over exact bytes and record file mode, byte size, text/binary classification, and declared artifact-class owner.
7. Compute `file_manifest_checksum` and `snapshot_content_checksum` using `040.CanonicalJSON` over materialized manifest rows.
8. Emit `StructuredInputRepositorySnapshot` with diagnostic `selected_ref_name` only when redaction permits.
```

Same repository profile ref, exact commit, tree hash, selected paths, file bytes, and file metadata must produce byte-identical `repository_snapshot_id`, `file_manifest_checksum`, and `snapshot_content_checksum`.

### ValidateStructuredInputRepositorySnapshot

```text
ValidateStructuredInputRepositorySnapshot(snapshot, profile, validation_runs, execution_scope):
1. Validate profile ref, checksum, lifecycle status, and activation scope.
2. Validate path rules, file manifest ordering, regular-file restriction, duplicate path rejection, and artifact-class owner coverage.
3. Validate public/private classification and redaction policy refs before caller-visible or audit-visible output.
4. Validate every required `120.StructuredInputRepositoryValidationMatrix` row for the exact `repository_snapshot_id`, tree hash, selected paths, and file manifest checksum.
5. Reject stale validation when validation refs were computed for a different commit SHA, tree hash, selected path set, file manifest checksum, or profile checksum.
6. On force-push or ref rewrite, invalidate validation unless the exact validated commit SHA and tree hash remain selected.
7. Require materialization refs before package release handoff and package-set refs before production activation when package-supplied artifacts affect output.
```

### Structured input repository manifest completeness

`VersionManifestCompletenessMatrix` requires these refs whenever repository-authored structured inputs affect output: repository profile ref, snapshot ref, file manifest checksum, validation run ref, materialization result ref when packaged, package release ref when created, package-set ref when activated, access policy ref when API-visible, redaction policy ref when any repository value is exposed, and validation matrix refs.

`VersionManifest.structured_input_refs` is required when structured-input snapshots affect output. Entries must also appear in `included_refs`; the field is not a parallel manifest mechanism.

| Error code | Required use |
| --- | --- |
| `STRUCTURED_INPUT_REPOSITORY_PROFILE_MISSING` | Required repository profile ref is absent, inactive, checksum-mismatched, out-of-scope, or not validated. |
| `STRUCTURED_INPUT_SNAPSHOT_UNRESOLVED` | A selected repository ref cannot resolve to one exact commit SHA and tree hash. |
| `STRUCTURED_INPUT_PATH_INVALID` | Selected path fails normalization or regular-file validation. |
| `STRUCTURED_INPUT_MUTABLE_REF_FORBIDDEN` | Mutable branch, tag, PR ref, repository URL, or rebuilt tip is used as activation, rollback, or manifest target. |
| `STRUCTURED_INPUT_SNAPSHOT_MISMATCH` | Validation, materialization, or manifest refs do not match exact commit, tree, file manifest, or snapshot checksum. |
| `STRUCTURED_INPUT_VALIDATION_STALE` | Validation run is not for the exact current snapshot or was invalidated by ref rewrite. |
| `STRUCTURED_INPUT_REVALIDATION_REQUIRED` | Merge commit, force-push, or ref rewrite requires exact snapshot revalidation before materialization or activation. |

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
4. Compute `RunLockSet` and materialized `RunLockLeasePolicy`.
5. Acquire the complete lock set before production output and start the heartbeat schedule. Validation-only implementations may omit the schedule only when they require `AssertRunLockHeldBeforeCommit` before every output-affecting validation write.
6. For each stage in canonical topological order, sorting simultaneously ready stages by lexical `stage_id`:
   a. Validate imported profiles, activation-controlled artifact refs, and state refs before stage execution.
   b. Validate permitted outputs.
   c. Execute package role or pure product algorithm.
   d. Persist StageStateRecord for output-affecting state.
   e. Before each production output commit, call `AssertRunLockHeldBeforeCommit`.
   f. On heartbeat uncertainty, block output until assertion succeeds.
   g. Persist ProcessingStageLifecycleResult.
   h. Stop on non-isolated failure and emit deterministic error.
7. On lock loss, persist `MarkRunLockLost`, stop downstream stages, and forbid watermark, graph apply promotion, package activation, absence, cleanup, retraction, graph expiry, table maintenance, last-known-good, and derived-view advancement.
8. Emit VersionManifest with every output-affecting activation artifact ref before any output is externally visible.
9. Release locks only after all required commit refs, lifecycle results, and manifest refs are persisted.
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
| `EvidenceRef` output | core schema registry checksum, evidence artifact kind registry checksum, consulted evidence artifact class row-set refs, referenced record or artifact checksum, redaction policy refs, owner artifact refs, validation refs, and artifact lifecycle refs when the evidence cites an activation artifact. |
| Silver output | raw refs, parser package, parser profile, mapping bundle, `MappingProjectManifest`, `MappingCompilerPipeline`, `MappingValidationRule` row-set refs, `ObservationToOCSFMappingRowSet`, `ExternalSchemaProfile`, `ExternalSchemaArtifactRef`, `ProfileResolutionManifest`, `ExternalEnumMappingRuleSet`, `OCSFBaseEventFieldPolicySet`, `SourceExtensionFieldRuleSet`, `CanonicalValidationOutput`, observation-type validation matrix refs, lifecycle transition evidence refs for every activation-controlled mapping artifact, package-set manifest refs when any mapping artifact is package-supplied, and `OCSFProfileUpgradeReport` refs when the active external schema profile differs from the prior production profile. |
| Identity output | resolver profile row set, identifier evidence class row set, identifier scope row set, candidate generation profile, identity hard blocker row set, asset generation boundary row set, target selector safety policy, resolver decision matrix row set, identity confidence band row set, identity review routing policy, identity split policy, resolver explanation policy, evidence item refs/checksums, identity decision refs, review case refs when applicable, resolver explanation refs/checksums, graph correction handoff refs when applicable, activation report refs when activation or promotion is in scope, and shadow/canary refs when promotion uses them. |
| Temporal resolution output | `TemporalSemanticsPolicy` row ref, `KnowledgeTimeImportPolicy` row ref, source evidence refs, selected input path/value checksum, `TemporalObservationTimeResolution` ref, and resolution checksum. |
| Gold fact output | temporal resolution ref and checksum, exact source authority row refs, lakehouse feed completeness row refs, coverage dimension/profile refs, coverage assertion refs, staleness policy refs, progress-signal policy refs when consulted, control-result mapping refs for control facts, source-history retention refs for history/no-change facts, supplier visibility refs for permission-sensitive absence, absence derivation policy refs, `060.AbsenceDerivationResult` ref when `absence_outcome != null` or when the candidate consumed negative, stale, delete, cleanup, no-change, fixed-state, DNS, DHCP/IPAM, flow non-observation, cloud disappearance, or CDC tombstone evidence, source refs, derivation refs, and validation refs. |
| Gold correction output | `GoldFactChangeSet` ref, correction policy ref, assertion transition row ref, old `LakehouseSnapshotRef`, new `LakehouseSnapshotRef`, table-set checksum ref, retention-protection ref, temporal resolution refs, authority refs, absence refs when applicable, and graph handoff effect refs. |
| Late-arrival output | late-arrival policy row ref, late-arrival route state, temporal resolution ref, authority/completeness/coverage/staleness refs, absence refs when applicable, quarantine refs when selected, and watermark refs when consulted. |
| Replay output | `ReplayEquivalencePolicy` output-class row ref, `ReplayInputSufficiencyCheck` ref, replay equivalence checksum, required owner-specific included-field refs, excluded volatile-field list, and failure-precedence result. |
| CDC-shaped feed replay output | CDC raw subtype refs, schema-history refs, offset refs, tombstone refs, heartbeat refs, replay sufficiency check, and non-authority diagnostics proving CDC metadata was not used as fact time or absence authority. |
| Graph handoff output | `GoldFactChangeSet` ref, graph handoff effect ref, projection profile ref when known, `070.GraphCorrectionHandoff` refs when identity split projection occurs, resolver explanation checksum when split-driven, `060.AbsenceDerivationResult` refs for expiry or cleanup, and validation refs. |
| Absence, cleanup, retraction, or graph expiry output | requested effect token, `060.AbsenceDerivationResult`, `source_authority_closure_matrix_row_set` ref/checksum or deterministic block validation ref, closure validation ref, and every consulted underlying `060` row-set ref consulted by `DeriveAbsenceOrUnknown`. |
| Watermark output | `ProjectionWatermarkPolicy`, `WatermarkCommitRecord`, closure row-set ref/checksum or deterministic block validation ref, completeness decision refs, blocking/no-op refs when advancement is denied, coverage refs where applicable, staleness refs, and all consulted `060` row-set refs. |
| Graph delta/apply | graph projection profile, graph projection row-set checksum, graph edge semantics row-set checksum, traversal class row refs where applicable, graph object output eligibility row-set checksum, graph property evidence policy refs, backend taxonomy mapping profile ref, graph query translation profile ref, graph read-model schema profile ref, graph apply profile ref, derived-view lag policy ref, graph delta set ref, idempotency key, backend profile ref, backend schema fingerprint, graph apply lifecycle transition evidence, graph apply result, graph rebuild manifest ref when rebuild is involved, graph index consistency check refs, derived-view state refs, graph backend selection policy ref when the backend profile was omitted and defaulted, provider id, provider version ref, provider adapter version ref, driver version ref, TinkerPop version ref when provider is JanusGraph, storage backend kind/version/config checksum, index backend kind/version/config checksum, provider runtime config checksum, schema initialization config checksum, provider capability matrix ref, package release refs for provider runtime, driver, storage adapter, and index adapter, package-set manifest checksum, raw-write bypass evidence ref, transaction semantics evidence refs, read-after-write proof ref, and index freshness refs. |
| Graph backend selection/defaulting | selected profile ref, selection policy ref, provider id, defaulting decision ref, explicit-or-omitted input checksum, provider capability matrix ref, backend package gate refs, and validation refs. |
| Run-lock governed output | `RunLockLeasePolicy` checksum, `RunLockSet` ref/checksum, lock record checksums, fencing token set, operation evidence refs, recovery evidence refs when any, commit guard refs for each production commit, lock-loss evidence ref when any, lock idempotency keys/checksums, lifecycle transition evidence refs, and generated error registry checksum. |
| Lifecycle-affecting output | lifecycle machine ID, version, checksum, transition evidence refs, selected transition row IDs, lifecycle state artifact refs, and owner guard row refs. |
| API, export, health, compliance, audit, or validation-visible output | `110.CommonApiResponseEnvelope` ref when applicable, `api_contract_version`, request checksum, endpoint outcome matrix checksum, state-label mapping checksum, generated `ErrorCodeRegistry` checksum, authorization policy refs, `110.AuthorizationDecision` refs, redaction policy refs, redaction context checksum, page-token policy refs, derived-view state refs when graph-derived output is served, audit event refs, owner state refs, owner error refs, compliance/export checksums, health or validation output checksums, and API/export/health/audit/validation refs. Visibility must fail with `VERSION_MANIFEST_INCOMPLETE` before rendering when `generated_error_registry_checksum` is absent. |
| Observability-visible health, API diagnostics, audit diagnostics, validation diagnostics, or telemetry runtime state output | observability instrumentation profile ref, signal policy ref, trace context policy ref when tracing is enabled, telemetry correlation policy ref when correlation refs are emitted, metric instrument catalog ref when metrics are enabled, telemetry attribute policy ref, telemetry redaction policy ref, telemetry exporter profile ref when export is enabled, telemetry health mapping policy ref when health may be affected, telemetry replay exclusion policy ref, telemetry runtime state refs when health/API/audit/validation output depends on telemetry state, package-set refs when telemetry artifacts are package-supplied, and validation refs. |
| Analysis output | `AnalysisRuleBundle`, `AnalysisRule` row refs, `RuleGraphCompatibilityMatrix`, query target refs, graph derived-view refs when graph-backed, authorization/redaction refs, output checksum, replay policy output-class row, and validation refs. |
| Threat-intel enrichment output | `ThreatIntelEnrichmentProfile`, `ThreatIntelArtifactRef`, distribution mapping policy, object-template/taxonomy/galaxy refs when used, redaction refs, artifact checksum, output checksum, and validation refs. |
| Lineage output | `RunDatasetIOContract`, `LineageFacetMappingPolicy` row refs, schema URL, schema bytes checksum, facet bytes checksum, collision decision, redaction refs, and validation refs. |
| Registry governance artifact activation | `RegistryArtifactGovernance`, `RegistryCustomPropertySchema`, `RegistryClassificationPolicy`, approval refs, lifecycle transition evidence, package-set ref when package-supplied, artifact checksum, activation scope, and validation refs. |
| Derived graph edge output | `DerivedGraphEdgeRule`, supporting fact refs, authority/completeness/temporal refs where applicable, `090` projection/profile/semantics/eligibility refs, deterministic ID inputs, output checksum, and validation refs. |
| Structured-input-derived output | repository profile ref, repository snapshot ref, selected path refs, file manifest checksum, snapshot content checksum, validation run refs, materialization result refs when packaged, package release refs when created, package-set refs when activated, access policy refs when API-visible, redaction policy refs when values are exposed, and structured-input validation matrix refs. |
| Package activation | package-set manifest checksum, `ProductionPackageSetManifest.package_set_checksum`, `PackageReleaseManifest.release_checksum`, package release manifest refs and checksums, selected `PackageTypePolicyRowSet` checksum, selected `PackageTypePolicyRow` refs and checksums, `PackageTypePolicyRowCoverageMatrix` checksum, package repository model row refs, resolved `target_environment`, `activation_mode`, artifact digests, sizes, media types, subject digests, repository metadata refs, repository snapshot refs, repository freshness proof refs, repository anti-rollback state refs, trust policy refs, signature verification result refs, transparency evidence refs, attestation set refs, build provenance refs, SBOM refs, dependency lock refs, compatibility matrix refs, validation refs, package developer contract refs, package stage binding refs, promotion record refs, deployment revision refs, selected `PackageDeprecationWindowPolicyRow` refs and checksums, last-known-good health gate refs, last-known-good package-set refs when applicable, rollback compatibility policy refs, rollback plan/result refs, quarantine record refs, quarantine scope policy refs, emergency override refs, `PackageActivationFailureEvent` ref when candidate activation fails, lifecycle transition evidence refs, and approval refs. |

Package activation refs required by `VersionManifestCompletenessMatrix` must appear in `included_refs`. `PackageReleaseManifest` and `ProductionPackageSetManifest` fields are not substitutes for `VersionManifest.included_refs`; package-specific manifest fields must not create a parallel manifest mechanism unless this spec explicitly adds a field row and defines checksum inclusion behavior.

### VersionManifestSchema

| Field | Required | Rule |
| --- | ---: | --- |
| `version_manifest_id` | Yes | Deterministic ID computed from `manifest_kind` and canonical included refs. |
| `manifest_kind` | Yes | Closed token for run, replay, validation, graph rebuild, or package activation. |
| `included_refs` | Yes | Canonically sorted refs for every output-affecting artifact. API/export output must represent the required `VersionManifestCompletenessMatrix` refs through this field rather than through a parallel manifest mechanism. |
| `schema_registry_checksum` | Yes | Active `040` core schema registry checksum. |
| `package_set_checksum` | Yes when packages affect output | Immutable `ProductionPackageSetManifest` checksum. |
| `package_activation_policy_refs` | Required when package activation affects output | Selected package type policy row-set checksum, selected package type policy row refs, package type coverage matrix checksum, selected deprecation policy row refs, compatibility row refs, and package activation failure event refs when applicable. These refs must also appear in `included_refs`; this field is not a parallel manifest mechanism. |
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
| `telemetry_policy_refs` | Required when telemetry affects health, API diagnostics, audit diagnostics, validation diagnostics, or operator-visible telemetry state | Canonically sorted `140` activation artifact refs and checksums. These refs must also appear in `included_refs`; this field is not a parallel manifest mechanism. |
| `telemetry_runtime_state_refs` | Required when telemetry runtime state affects health, API, audit, validation, or operator-visible diagnostics | Canonically sorted `140.TelemetryRuntimeState` refs and checksums. These refs must also appear in `included_refs`; this field is not a parallel manifest mechanism. |
| `structured_input_refs` | Required when structured-input snapshots affect output | Canonically sorted structured input repository profile refs, snapshot refs, file manifest checksums, validation run refs, materialization result refs, package release refs, package-set refs, access policy refs, and redaction policy refs. Every ref in this field must also appear in `included_refs`; this field is not a parallel manifest mechanism. |

| `created_at` | Yes | RFC3339 UTC manifest creation time; excluded from manifest ID and included in checksum. |
| `manifest_checksum` | Yes | SHA-256 over `040.CanonicalJSON` excluding only `manifest_checksum`. |

`version_manifest_id` must be `vm_` plus SHA-256 lowercase hex over `040.CanonicalJSON` of `manifest_kind` and `included_refs`. `created_at` must not affect `version_manifest_id`.

When identity review output, identity mutation, review expiration, or review terminal decision output can affect production behavior, `VersionManifest` must include `070.IdentityReviewCaseStateMachine.v1` machine refs, machine checksums, transition evidence refs, review-case refs, terminal decision refs, and resolver explanation refs.

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
| identity review case | `070` | `070.IdentityReviewCaseStateMachine.v1` | `070`; generic evidence fields in `030` | `val-070-identity-review-totality`, `val-070-identity-review-idempotency`, `val-070-identity-review-illegal-no-mutation`, `val-070-identity-review-terminal-output`, `val-domain-lifecycle-todo-resolved` |

`ValidateSpecSet` must fail when any row in this table omits a concrete machine ID, transition-table owner, validation row, or owner guard/event-derivation location. Owner-specific errors must be used when available; generic lifecycle errors are fallback only.

If an owner spec defines a production-affecting lifecycle machine and transition matrix, domain-level lifecycle status must be `blocked_validation` until the matching `120.LifecycleValidationMatrix` rows pass. It must not remain `blocked_owner_todo` unless the owner spec lacks the machine, states, events, transition matrix, or illegal-transition rule.

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
| All lock keys available | Acquire the complete lock set before any production output write and persist complete operation evidence, lock records, and fencing token set. |
| One or more lock keys unavailable and unexpired | Acquire no locks or release every acquired lock before returning `RUN_LOCK_CONFLICT` or `RUN_LOCK_ACQUISITION_FAILED`. |
| Partial acquisition failure | Emit `RUN_LOCK_ACQUISITION_FAILED`; write no production output and advance no watermark. |
| Heartbeat uncertain | Emit `RUN_LOCK_HEARTBEAT_UNCERTAIN`; block output commits until `AssertRunLockHeldBeforeCommit` succeeds. |
| Lock lost during run | Stop before next output commit, emit `RUN_LOCK_LOST`, persist `MarkRunLockLost`, and prevent downstream stages from running. |
| Stale lock | Recover only when the `Stale recovery predicate` in `## Run Locks` is true and the compare-and-set mutation assigns a strictly greater fencing token. Otherwise emit `RUN_LOCK_STALE_RECOVERY_NOT_ELIGIBLE` or `RUN_LOCK_STALE_RECOVERY_FAILED`, acquire no partial lock set, write no production output, and advance no watermark. |
| Commit guard missing or stale | Emit `RUN_LOCK_COMMIT_GUARD_MISSING`, `RUN_LOCK_FENCING_TOKEN_STALE`, or `RUN_LOCK_ASSERTION_FAILED` before output commit. |

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

### ProcessingErrorRegistryFragment

This owner fragment feeds `110.GenerateErrorCodeRegistry`. `110` owns the generated caller-visible registry shape, redaction, duplicate-code policy, generic-code precedence, and final checksum. `030` owns processing, stage, run-lock, lifecycle, declared-subset, activation-artifact fallback, and version-manifest causes.

| error_code | owner_spec | severity | retry_class | caller_visible_fields | audit_visible_fields | redaction_rule | owner_context_schema_ref | fixture_ref |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `FORBIDDEN_STAGE_OUTPUT` | `030` | `security_error` | `none` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.security_boundary` | `030.ProcessingErrorContext` | `error-registry-030-forbidden-stage-output` |
| `DAG_STAGE_ID_DUPLICATE` | `030` | `error` | `caller_correctable` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `030.ProcessingErrorContext` | `error-registry-030-dag-stage-id-duplicate` |
| `DAG_CYCLE_DETECTED` | `030` | `error` | `caller_correctable` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `030.ProcessingErrorContext` | `error-registry-030-dag-cycle-detected` |
| `RUN_LOCK_COLLISION` | `030` | `error` | `retry_after_owner_repair` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `030.ProcessingErrorContext` | `error-registry-030-run-lock-collision` |
| `RUN_LOCK_ACQUISITION_FAILED` | `030` | `blocked` | `transient_retryable` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `030.ProcessingErrorContext` | `error-registry-030-run-lock-acquisition-failed` |
| `RUN_LOCK_TIMING_INVALID` | `030` | `error` | `caller_correctable` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `030.ProcessingErrorContext` | `error-registry-030-run-lock-timing-invalid` |
| `RUN_LOCK_CONFLICT` | `030` | `blocked` | `transient_retryable` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `030.ProcessingErrorContext` | `error-registry-030-run-lock-conflict` |
| `RUN_LOCK_STORE_TIME_UNAVAILABLE` | `030` | `blocked` | `retry_after_refresh` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `030.ProcessingErrorContext` | `error-registry-030-run-lock-store-time-unavailable` |
| `RUN_LOCK_HEARTBEAT_FAILED` | `030` | `blocked` | `retry_after_refresh` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `030.ProcessingErrorContext` | `error-registry-030-run-lock-heartbeat-failed` |
| `RUN_LOCK_HEARTBEAT_UNCERTAIN` | `030` | `blocked` | `retry_after_refresh` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `030.ProcessingErrorContext` | `error-registry-030-run-lock-heartbeat-uncertain` |
| `RUN_LOCK_STALE_RECOVERY_NOT_ELIGIBLE` | `030` | `blocked` | `transient_retryable` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `030.ProcessingErrorContext` | `error-registry-030-run-lock-stale-recovery-not-eligible` |
| `RUN_LOCK_STALE_RECOVERY_FAILED` | `030` | `blocked` | `transient_retryable` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `030.ProcessingErrorContext` | `error-registry-030-run-lock-stale-recovery-failed` |
| `RUN_LOCK_FENCING_TOKEN_STALE` | `030` | `blocked` | `retry_after_refresh` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `030.ProcessingErrorContext` | `error-registry-030-run-lock-fencing-token-stale` |
| `RUN_LOCK_IDEMPOTENCY_CONFLICT` | `030` | `error` | `none` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `030.ProcessingErrorContext` | `error-registry-030-run-lock-idempotency-conflict` |
| `RUN_LOCK_COMMIT_GUARD_MISSING` | `030` | `security_error` | `none` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.security_boundary` | `030.ProcessingErrorContext` | `error-registry-030-run-lock-commit-guard-missing` |
| `RUN_LOCK_ASSERTION_FAILED` | `030` | `blocked` | `retry_after_refresh` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `030.ProcessingErrorContext` | `error-registry-030-run-lock-assertion-failed` |
| `RUN_LOCK_LOST` | `030` | `blocked` | `retry_after_refresh` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `030.ProcessingErrorContext` | `error-registry-030-run-lock-lost` |
| `ACTIVATION_ARTIFACT_INCOMPLETE` | `030` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `030.ProcessingErrorContext` | `error-registry-030-activation-artifact-incomplete` |
| `ACTIVATION_ARTIFACT_OWNER_MISMATCH` | `030` | `error` | `retry_after_owner_repair` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `030.ProcessingErrorContext` | `error-registry-030-activation-artifact-owner-mismatch` |
| `DECLARED_DAG_SUBSET_PROFILE_MISSING` | `030` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `030.ProcessingErrorContext` | `error-registry-030-declared-dag-subset-profile-missing` |
| `DECLARED_DAG_SUBSET_SCOPE_MISMATCH` | `030` | `error` | `caller_correctable` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `030.ProcessingErrorContext` | `error-registry-030-declared-dag-subset-scope-mismatch` |
| `DECLARED_DAG_SUBSET_OUTPUT_FORBIDDEN` | `030` | `security_error` | `none` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.security_boundary` | `030.ProcessingErrorContext` | `error-registry-030-declared-dag-subset-output-forbidden` |
| `LIFECYCLE_MACHINE_MISSING` | `030` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `030.ProcessingErrorContext` | `error-registry-030-lifecycle-machine-missing` |
| `LIFECYCLE_MACHINE_INACTIVE` | `030` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `030.ProcessingErrorContext` | `error-registry-030-lifecycle-machine-inactive` |
| `LIFECYCLE_MACHINE_CHECKSUM_MISMATCH` | `030` | `error` | `retry_after_owner_repair` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `030.ProcessingErrorContext` | `error-registry-030-lifecycle-machine-checksum-mismatch` |
| `LIFECYCLE_EVENT_INVALID` | `030` | `error` | `caller_correctable` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `030.ProcessingErrorContext` | `error-registry-030-lifecycle-event-invalid` |
| `LIFECYCLE_STATE_DERIVATION_FAILED` | `030` | `error` | `retry_after_owner_repair` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `030.ProcessingErrorContext` | `error-registry-030-lifecycle-state-derivation-failed` |
| `LIFECYCLE_ILLEGAL_TRANSITION` | `030` | `error` | `caller_correctable` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `030.ProcessingErrorContext` | `error-registry-030-lifecycle-illegal-transition` |
| `LIFECYCLE_IDEMPOTENCY_CONFLICT` | `030` | `error` | `retry_after_owner_repair` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `030.ProcessingErrorContext` | `error-registry-030-lifecycle-idempotency-conflict` |
| `LIFECYCLE_TRANSITION_EVIDENCE_INCOMPLETE` | `030` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `030.ProcessingErrorContext` | `error-registry-030-lifecycle-transition-evidence-incomplete` |
| `LIFECYCLE_PARENT_CHILD_STATE_INVALID` | `030` | `error` | `retry_after_owner_repair` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `030.ProcessingErrorContext` | `error-registry-030-lifecycle-parent-child-state-invalid` |
| `LIFECYCLE_GUARD_REFUSED` | `030` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `030.ProcessingErrorContext` | `error-registry-030-lifecycle-guard-refused` |
| `VERSION_MANIFEST_INCOMPLETE` | `030` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `030.ProcessingErrorContext` | `error-registry-030-version-manifest-incomplete` |

### ProcessingErrorContext

`ProcessingErrorContext` is the owner context schema for `030` registry rows.

| Field | Required | Rule |
| --- | ---: | --- |
| `context_schema_version` | Yes | Immutable `030` context schema version. |
| `owner_spec` | Yes | Must be `030`. |
| `error_code` | Yes | Must match the generated registry row. |
| `failure_class` | Yes | Closed token: `dag`, `stage_output`, `run_lock`, `activation_artifact`, `declared_subset`, `lifecycle`, or `version_manifest`. |
| `operation` | Yes | DAG validation, stage execution, lock acquisition, artifact validation, lifecycle transition, subset validation, or manifest validation. |
| `affected_record_type` | Yes | Stage, DAG, lock set, artifact ref, lifecycle evidence, subset profile, or version manifest type. |
| `field_path` | Yes | Exact field path when known; null for artifact-wide failures. |
| `stage_id` | No | Required when the failure is stage-specific. |
| `stage_class` | No | Required when output permission or lifecycle stage behavior is involved. |
| `run_lock_key_ref` | No | Required for run-lock failures; raw lock inputs remain redacted. |
| `run_lock_set_id` | No | Required when the failure affects an entire `RunLockSet`. |
| `run_lock_operation_kind` | No | Required for acquire, heartbeat, assert, recover, release, and mark-lost failures. |
| `run_lock_operation_evidence_ref` | No | Required when operation evidence was persisted. |
| `run_lock_commit_guard_ref` | No | Required for commit-guard failures. |
| `fencing_token_set_ref` | No | Required for fencing-token failures; raw token values may be redacted by policy. |
| `lease_policy_checksum` | No | Required for timing and bounds failures. |
| `idempotency_key_checksum` | No | Required for idempotency conflicts. |
| `activation_artifact_ref` | No | Required for activation-artifact failures. |
| `lifecycle_machine_ref` | No | Required for lifecycle machine failures. |
| `version_manifest_ref` | No | Required when manifest validation was attempted. |
| `validation_refs` | Yes | Exact `120` validation refs proving the failure. |
| `redaction_classes` | Yes | Raw payloads, private bindings, credentials, backend IDs, provider-native query text, package payload bytes, and signer secrets must map to `always_forbidden`. |

### Acceptance Criteria

| ID | Criterion |
| --- | --- |
| `030-ANALYSIS-REGISTRY-ARTIFACT-CLASS-AC-001` | `ValidateActivationControlledArtifactRef` fails with `ACTIVATION_ARTIFACT_INCOMPLETE` when any `130` artifact class token is unknown, omitted, inactive, scope-mismatched, checksum-mismatched, or unvalidated. |
| `030-ANALYSIS-REGISTRY-MANIFEST-AC-001` | Analysis output fails with `VERSION_MANIFEST_INCOMPLETE` when any rule bundle, rule row, compatibility matrix, query target, derived-view, authorization, redaction, replay policy, output checksum, or validation ref is missing. |
| `030-THREAT-INTEL-MANIFEST-AC-001` | Threat-intel enrichment output fails with `VERSION_MANIFEST_INCOMPLETE` when profile, artifact, distribution, taxonomy, galaxy, object-template, redaction, checksum, or validation refs are missing. |
| `030-LINEAGE-FACET-MANIFEST-AC-001` | Lineage output fails with `VERSION_MANIFEST_INCOMPLETE` when lineage policy rows, schema URL, schema bytes checksum, facet checksum, collision decision, redaction refs, or validation refs are missing. |
| `030-DERIVED-GRAPH-EDGE-MANIFEST-AC-001` | Derived graph edge output fails with `VERSION_MANIFEST_INCOMPLETE` when rule, supporting fact, authority, completeness, temporal, projection, semantics, eligibility, deterministic ID input, output checksum, or validation refs are missing. |
| `030-ENRICHMENT-LINEAGE-STAGE-OUTPUT-AC-001` | `ExecuteProcessingStageDAG` permits `enrichment` and `lineage` outputs listed in the stage table and fails with `FORBIDDEN_STAGE_OUTPUT` for any other production output class. |
| `030-API-EXPORT-MANIFEST-AC-001` | API/export output fails with `VERSION_MANIFEST_INCOMPLETE` when request checksum, endpoint outcome matrix checksum, state-label mapping checksum, generated error-registry checksum, authorization decision ref, redaction context checksum, page-token policy ref, audit event ref, owner error ref, or required derived-view state ref is missing. |
| `030-OBSERVABILITY-HANDOFF-AC-001` | Stage telemetry context creation does not alter deterministic stage order, output permissions, lifecycle behavior, run locks, output IDs, output checksums, replay equivalence, watermarks, graph apply, or package activation. |
| `030-OBSERVABILITY-ARTIFACT-CLASS-AC-001` | Every telemetry artifact class used by `140` appears in the closed `ActivationControlledArtifactRef.artifact_class` registry and invalid telemetry artifact classes fail before stage execution. |
| `030-OBSERVABILITY-MANIFEST-AC-001` | Health/API/audit/validation output that depends on telemetry policy or runtime state includes required telemetry refs in `VersionManifest`; missing refs fail with `TELEMETRY_VERSION_MANIFEST_INCOMPLETE` or `VERSION_MANIFEST_INCOMPLETE`. |
| `030-CLEANUP-AC-001` | No banned reference class remains. |
| `030-CLEANUP-AC-002` | Stage output permissions remain total for declared stage classes. |
| `030-CLEANUP-AC-003` | Any stage emitting a forbidden record class still fails before commit. |
| `030-CLEANUP-AC-004` | `VersionManifest` remains the output-affecting reproducibility record. |
| `030-SCHEMA-PATCH-AC-001` | Every production output class owned by `040` is schema-validated before commit. |
| `030-SCHEMA-PATCH-AC-002` | `VersionManifest` records the active core schema registry checksum for every output-affecting run. |
| `030-SCHEMA-PATCH-AC-003` | A forbidden or malformed core record fails before downstream domain algorithms execute. |
| `030-DAG-ORDER-AC-001` | Duplicate stage IDs fail with `DAG_STAGE_ID_DUPLICATE`, cycles fail with `DAG_CYCLE_DETECTED`, and ready-stage ties sort lexically by `stage_id`. |
| `030-RUNLOCK-AC-001` | Run lock acquisition is all-or-nothing and partial acquisition failure releases acquired locks before returning `RUN_LOCK_ACQUISITION_FAILED`. |
| `030-RUNLOCK-CLOSE-DEFAULTS-AC-001` | Omitted lease fields materialize to `300/60/30/15/30`, `lock_store_time`, and `fencing_required = true` before lock checksum computation. |
| `030-RUNLOCK-CLOSE-BOUNDS-AC-001` | Invalid TTL, heartbeat, grace, timeout, or commit-guard bounds fail with `RUN_LOCK_TIMING_INVALID` before lock mutation. |
| `030-RUNLOCK-CLOSE-ACQUIRE-AC-001` | Multi-key acquisition is all-or-nothing; partial acquisition releases acquired keys and writes no production output. |
| `030-RUNLOCK-CLOSE-HEARTBEAT-AC-001` | A valid heartbeat increments the sequence, extends the lease under lock-store time, persists evidence, and is idempotent for the same sequence. |
| `030-RUNLOCK-CLOSE-RECOVERY-AC-001` | Stale recovery succeeds only when every predicate is true, evidence is persisted, and the new fencing token is strictly greater than the prior token. |
| `030-RUNLOCK-CLOSE-RECOVERY-RACE-AC-001` | A newer heartbeat after contender read causes recovery failure and no production output. |
| `030-RUNLOCK-CLOSE-OLD-HOLDER-FENCED-AC-001` | An old holder that wakes after recovery fails the commit guard with `RUN_LOCK_FENCING_TOKEN_STALE` and runs no downstream stage. |
| `030-RUNLOCK-CLOSE-IDEMPOTENCY-AC-001` | Same operation idempotency key and input checksum returns byte-identical evidence; same key with different checksum fails before mutation. |
| `030-RUNLOCK-CLOSE-COMMIT-GUARD-AC-001` | Every production commit has a successful `RunLockCommitGuard`; missing, stale, fenced, or failed guards block mutation. |
| `030-RUNLOCK-CLOSE-MANIFEST-AC-001` | Lock policy, lock set, records, fencing tokens, operation evidence, recovery evidence, commit guards, idempotency keys, and lock-loss evidence appear in `VersionManifest` when output is lock-governed. |
| `030-RUNLOCK-CLOSE-LOCK-LOSS-AC-001` | Lock loss blocks downstream stages and forbids watermark, graph apply promotion, package activation, absence, cleanup, retraction, graph expiry, table maintenance, last-known-good, and derived-view advancement. |
| `030-RUNLOCK-CLOSE-GRAPH-HANDOFF-AC-001` | Graph apply consumes a valid commit guard before backend mutation and cannot advance derived view after lock loss. |
| `030-RUNLOCK-CLOSE-PACKAGE-HANDOFF-AC-001` | Package activation uses the same `RunLockSet` semantics and preserves the current active set on lock loss or fencing failure. |
| `030-RUNLOCK-CLOSE-MAINTENANCE-HANDOFF-AC-001` | Table maintenance uses `RunLockCommitGuard` before destructive or rewrite commits. |
| `030-RUNLOCK-CLOSE-WATERMARK-BLOCK-AC-001` | Lock conflict, lock loss, stale fencing, heartbeat uncertainty, and stale recovery failure do not advance any watermark. |
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
| `030-IDENTITY-HARD-BLOCKER-MANIFEST-AC-001` | Identity output fails with `VERSION_MANIFEST_INCOMPLETE` when the active `identity_hard_blocker_row_set` ref or checksum is absent. |
| `030-OCSF-MAP-ARTIFACT-AC-001` | Silver output fails with `VERSION_MANIFEST_INCOMPLETE` when any output-affecting OCSF mapping row set, `MappingProjectManifest`, `MappingCompilerPipeline`, `MappingValidationRule` row set, external schema profile, external schema artifact ref, profile-resolution manifest, enum rule set, base-event field policy set, source-extension rule set, canonical validation output, observation-type validation matrix, lifecycle transition evidence, package-set ref when package-supplied, or applicable `OCSFProfileUpgradeReport` ref is missing from `VersionManifest`. |
| `030-SOURCE-CLOSURE-MANIFEST-AC-001` | Absence-sensitive output fails with `VERSION_MANIFEST_INCOMPLETE` when the closure row-set ref, closure validation ref, or any underlying consulted `060` row-set ref is omitted from `VersionManifest.included_refs`. |
| `030-SOURCE-CLOSURE-MANIFEST-AC-002` | A `SourceAuthorityClosureMatrix` validation result cannot satisfy manifest completeness unless all underlying activation-controlled row refs and checksums are also present. |
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
| `030-GRAPH-BACKEND-VM-AC-001` | Graph serving, graph apply, graph query, graph rebuild, and graph replay fail with `VERSION_MANIFEST_INCOMPLETE` when any output-affecting graph backend provider, adapter, storage, index, schema-init, package, or capability ref is absent from `VersionManifest.included_refs`. |
| `030-GRAPH-REBUILD-MANIFEST-AC-001` | Graph rebuild promotion fails unless manifest refs include rebuild manifest, index consistency check, schema fingerprint, and canonical output checksum inputs. |
| `030-IDENTITY-SPLIT-GRAPH-HANDOFF-AC-001` | Identity split graph handoff output includes `070.GraphCorrectionHandoff` refs and resolver explanation checksum refs. |
| `030-PACKAGE-ARTIFACT-CLASS-AC-001` | Every package activation-controlled artifact class required by `100` appears in the closed `artifact_class` registry. |
| `030-EVIDENCE-ARTIFACT-MANIFEST-AC-001` | Any run that emits `EvidenceRef` includes evidence artifact kind registry checksum, evidence artifact class row-set refs, referenced artifact checksum, redaction refs, owner artifact refs, and validation refs in `VersionManifest`. |
| `030-EVIDENCE-ARTIFACT-MANIFEST-AC-002` | Activation artifact evidence fails when artifact lifecycle is inactive, checksum mismatches, package-set membership is required but absent, or evidence artifact class row-set refs are omitted. |
| `030-PACKAGE-VM-AC-001` | Package activation output fails with `PACKAGE_VERSION_MANIFEST_INCOMPLETE` or `VERSION_MANIFEST_INCOMPLETE` when any required package-set, release, trust, repository, SBOM, provenance, compatibility, rollback, quarantine, emergency, health, validation, lifecycle, or approval ref is missing from `VersionManifest.included_refs`. |
| `030-PACKAGE-POLICY-MANIFEST-AC-001` | Package activation fails before output when the selected `PackageTypePolicyRowSet`, selected `PackageTypePolicyRow`, selected deprecation policy row, package compatibility row, package type coverage matrix checksum, or release manifest checksum is missing from `VersionManifest.included_refs` or checksum-mismatched. |

### Structured input repository acceptance criteria

| ID | Criterion |
| --- | --- |
| `030-STRUCTURED-INPUT-SNAPSHOT-AC-001` | Same repository profile ref, commit, tree, selected paths, file bytes, file modes, and declared artifact-class owners produce byte-identical snapshot ID and file manifest checksum. |
| `030-STRUCTURED-INPUT-MUTABLE-REF-AC-001` | Branch, tag, pull request ref, repository URL, or default branch cannot satisfy production activation, rollback, or `VersionManifest` completeness. |
| `030-STRUCTURED-INPUT-FORCE-PUSH-AC-001` | Ref rewrite invalidates prior validation unless exact validated commit SHA and tree hash remain selected. |
| `030-STRUCTURED-INPUT-VM-AC-001` | Output derived from repository-authored artifacts fails with `VERSION_MANIFEST_INCOMPLETE` when snapshot, validation, materialization, package, access, redaction, or validation refs required by the matrix are omitted. |

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
| `030-AC-008` | Run-lock lease timing, heartbeat, stale recovery, fencing, idempotency, commit guards, and lock-loss effects are fully specified and contain no owner-local stale-lock `TODO:` row. |
| `030-AC-009` | Two implementations given the same lock records, input checksums, policy defaults, and idempotency key produce byte-identical lock operation evidence. |

## Open Questions

Open questions marked `TODO:` block authoritative status for the affected contract. A downstream implementation must not resolve a `TODO:` by inference.
