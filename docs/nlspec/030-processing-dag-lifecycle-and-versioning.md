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

A production output without a complete immutable `VersionManifest` must fail with `VERSION_MANIFEST_INCOMPLETE`.

## Declared Subset Contract

A production stage subset may execute only with an active `DeclaredDAGSubsetProfile`. The subset profile must state allowed outputs, forbidden outputs, cleanup behavior, absence behavior, projection behavior, watermark behavior, and required validation rows.

## Execution Algorithm

```text
ExecuteProcessingStageDAG(dag, requested_scope, package_set):
1. Validate dag acyclicity and active lifecycle state.
2. Resolve active package set and stage bindings.
3. Validate requested subset or full DAG mode.
4. Acquire RunLockSet.
5. For each stage in canonical topological order:
   a. Validate imported profiles and state refs.
   b. Validate permitted outputs.
   c. Execute package role or pure product algorithm.
   d. Persist StageStateRecord for output-affecting state.
   e. Persist ProcessingStageLifecycleResult.
   f. Stop on non-isolated failure and emit deterministic error.
6. Emit VersionManifest before any output is externally visible.
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
| Silver output | raw refs, parser package, mapping bundle, external schema profile, validation output. |
| Identity output | resolver profile, evidence scope, identity decision refs, review case refs when applicable. |
| Gold output | temporal policy, source authority, coverage, correction policy, source refs, snapshot refs. |
| Graph delta/apply | projection profile, graph delta set, idempotency key, backend profile, apply result. |
| API/export output | owner state labels, authorization/redaction policy, page-token policy, derived view state. |
| Package activation | package-set manifest, trust evidence, compatibility, validation, approval, rollback refs. |

### RunLockKeyDerivation

```text
RunLockKeyDerivation(scope):
1. Build an ordered tuple of source/feed scope, completeness scope, coverage scope, projection scope, graph apply target, package set, and requested output class.
2. Serialize the tuple with `040.CanonicalJSON`.
3. Hash with SHA-256 and prefix with `runlock_`.
4. If two non-identical tuples produce the same lock key, fail with `RUN_LOCK_COLLISION`.
```

### DeclaredDAGSubsetProfile matrix

| Subset mode | Allowed outputs | Forbidden outputs | Absence behavior | Cleanup behavior | Projection behavior | Watermark behavior | Validation requirement |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `validation_only` | validation reports, diagnostics | production records | forbidden | forbidden | forbidden | forbidden | negative authority rows |
| `feed_replay` | raw records and receipts | silver/gold/graph/API | no absence | no cleanup | no projection | no advancement | feed replay rows |
| `graph_rebuild` | graph rebuild manifest and graph apply result | raw/silver/gold mutation | no absence | no source cleanup | allowed from persisted deltas | projection only if `060` permits | rebuild equivalence rows |
| `package_canary` | canary output and activation evidence | current production output | no absence | no cleanup | canary only | no production advancement | package canary rows |

### Acceptance Criteria

| ID | Criterion |
| --- | --- |
| `030-CLEANUP-AC-001` | No banned reference class remains. |
| `030-CLEANUP-AC-002` | Stage output permissions remain total for declared stage classes. |
| `030-CLEANUP-AC-003` | Any stage emitting a forbidden record class still fails before commit. |
| `030-CLEANUP-AC-004` | `VersionManifest` remains the output-affecting reproducibility record. |

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
