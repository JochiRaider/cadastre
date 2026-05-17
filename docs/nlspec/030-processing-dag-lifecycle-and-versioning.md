---
doc_id: CADASTRE-NLSPEC-030
title: Processing DAG, Lifecycle, and Versioning
doc_type: authoritative-nlspec
status: migration_active
generated_on: 2026-05-17
source_prd: PRD-Cadastre.md
source_prd_sha256: b17ac5d44618c43a57efe8ebd9b6c6e0bd8debc949b513368383c515c07f9748
---

# Processing DAG, Lifecycle, and Versioning

## Authority

This document is a generated Cadastre NLSpec candidate. It is `migration_active` until the migration ledger marks its source rows complete and `docs/nlspec/120-validation-fixtures-and-acceptance.md` records passing acceptance evidence.

This document owns the contracts listed in `Exports`. Other active Cadastre NLSpecs may import those contracts by exact name and must not restate them.

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

## Definition of Done

| ID | Criterion |
| --- | --- |
| `030-AC-001` | The same DAG, package set, inputs, stage state, and requested subset produce the same stage order and output classes. |
| `030-AC-002` | A forbidden output class is rejected before commit and before downstream stage execution. |
| `030-AC-003` | Every production-affecting lifecycle has closed states, closed events, total transitions, illegal-transition behavior, and validation criteria. |
| `030-AC-004` | Every output-affecting state record is hashable, replayable, and included in `VersionManifest`. |
| `030-AC-005` | Production replay rejects missing, mutable-only, checksum-mismatched, or retention-ineligible manifest refs before writing output. |

## Source Traceability

| Source | Section or artifact | Location |
| --- | --- | --- |
| PRD-Cadastre.md | `ProcessingStageDAG` | lines 4265-4306 |
| PRD-Cadastre.md | `StageStateRecord` | lines 7083-7181 |
| PRD-Cadastre.md | `LifecycleStateMachineDefinition` | lines 7182-7315 |
| PRD-Cadastre.md | `Required MVP Lifecycle Machines` | lines 7316-7337 |
| PRD-Cadastre.md | `ProcessingStageLifecycleResult` | lines 7338-7374 |
| PRD-Cadastre.md | `RunLockSet` | lines 7668-7704 |
| PRD-Cadastre.md | `DeclaredDAGSubsetProfile` | lines 7705-7737 |
| PRD-Cadastre.md | `VersionManifest` | lines 2978-3108 |
| PRD-Cadastre.md | `Feed Stage DAG Execution Interface` | lines 9884-9948 |
| PRD-Cadastre.md | `Lifecycle State Machine Validation Interface` | lines 10056-10113 |
| PRD-Cadastre.md | `Required Product Decisions Before Final NLSpec` | lines 13635-13681 |
| Decomposition plan | Current user prompt | Domain decomposition, disposition matrix, dependency model, gap ledger, and migration acceptance criteria. |

## Open Questions

Open questions marked `TODO:` block authoritative status for the affected contract. A downstream implementation must not resolve a `TODO:` by inference.
