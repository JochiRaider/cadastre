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

- `GoldFactSchema`
- `ComputeGoldFactKeyId`
- `ComputeGoldFactId`
- `CoreRecordValidationAlgorithm`
- `ActivationControlledArtifactRef`

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

## Temporal Axes

Cadastre must keep source event time, source observation time, supplier collection time, supplier delivery time, lakehouse commit time, table snapshot time, CDC connector time, CDC heartbeat time, schema-history time, graph apply time, replay time, and platform current time distinct.

A `GoldFact` candidate may be created only after `ResolveFactTime` emits exactly one `TemporalObservationTimeResolution` row or a deterministic temporal error.

## Temporal and replay artifact activation boundary

Temporal, gold, correction, late-arrival, and replay algorithms are stable core contracts. Source/fact-specific temporal rows, correction rows, late-arrival rows, and replay output-class rows are activation-controlled artifacts.

| Artifact or contract | Volatility class | Required handling |
| --- | --- | --- |
| `TemporalSemanticsPolicy` | `activation_controlled_artifact` | Required before fact-time resolution. |
| `KnowledgeTimeImportPolicy` | stable mode definitions plus activation-controlled policy rows | Production default remains `current_import`; rows must be active for non-default modes. |
| `TemporalObservationTimeResolution` | `runtime_state_record` | Must be persisted or a deterministic temporal error emitted before gold candidate creation. |
| `LateArrivalPolicy` | `activation_controlled_artifact` | Required before late evidence routing. |
| `BitemporalQueryMode` | `stable_core_contract` | Query mode semantics are stable; owner rows may control visibility. |
| `GoldFactCorrectionPolicy` | `activation_controlled_artifact` | Required before correction class behavior. |
| `GoldFactChangeSet` | `runtime_state_record` | System-of-record change record. |
| `CorrectionSnapshotRefPolicy` | `activation_controlled_artifact` | Required before old/new snapshot role handling. |
| `ReplayEquivalencePolicy` | stable checksum algorithm plus activation-controlled output-class rows | Missing output-class rows block replay. |
| `ReplayInputSufficiencyCheck` | `stable_core_contract` | Preflight algorithm; concrete required rows are owner-controlled. |
| `GraphRebuildEquivalencePolicy` | split ownership with `090`; row instances activation-controlled where output-specific | Must remain consistent with graph rebuild owner. |
| `EventSequenceValidationCorpus` | `activation_controlled_artifact` | Validation artifact with checksummed fixtures. |
| `ResolveFactTime`, `EvaluateLateArrival`, `ApplyGoldCorrection`, `DeriveFacts`, `ComputeReplayEquivalenceChecksum` | `stable_core_contract` | Algorithms validate required activation refs before output. |

Current `TODO:` rows remain owner-decision blockers. Production outputs affected by unresolved temporal, correction, late-arrival, or replay policy rows remain blocked until artifact rows exist, validate, and appear in `VersionManifest`.

## ResolveFactTime Algorithm

```text
ResolveFactTime(observation, temporal_policy, knowledge_time_policy):
1. Validate required temporal and knowledge-time artifact refs through `030.ActivationControlledArtifactRef`.
2. Validate temporal_policy covers source dataset, observation type, and fact type.
3. Select valid-time input using declared precedence.
4. Reject absent, malformed, ambiguous, or non-authoritative time unless policy names an explicit fallback.
5. Select known-time input using KnowledgeTimeImportPolicy.
6. Reject reconstructed historical known time unless policy permits and required source-known-time evidence validates.
7. Emit TemporalObservationTimeResolution with selected fields, quality, source refs, policy refs, and checksum.
8. Include temporal policy checksums in `VersionManifest` before gold output.
```

Implicit current-time fallback is forbidden.

## Gold Derivation

`DeriveFacts` must validate required temporal, authority, correction, late-arrival, and replay artifact refs before output. It must consume silver observations, identity decisions, source authority decisions, coverage assertions, temporal resolutions, and active derivation profiles. It must compute `gold_fact_key_id` through `040.ComputeGoldFactKeyId` from subject, predicate, object/value, and valid interval. It must compute immutable `gold_fact_id` through `040.ComputeGoldFactId` from `gold_fact_key_id`, `known_from`, assertion state, authority row, temporal resolution, evidence set, and correction policy. It must emit `GoldFact`, `GoldFactChangeSet`, no-op, conflict, or deterministic error, and every emitted `GoldFact` must pass `040.GoldFactSchema`. It must not mutate existing `GoldFact` rows in place.

## Correction Contract

Gold corrections are append-only knowledge transitions. `ApplyGoldCorrection` must emit `GoldFactChangeSet` operations and must not mutate original `GoldFact` bytes. Effective `known_to` closure must be materialized from `GoldFactChangeSet` at read or replay time; base `GoldFact.known_to` remains the value written in the immutable record.

`CorrectionSnapshotRefPolicy` must require old and new lakehouse snapshot refs for every correction class. The contract table below defines default correction classes and records remaining source-specific blocker rows. Missing temporal resolution fails before gold ID computation. Missing authority row fails before fact creation. Schema validation fails before graph projection.

## Assertion-State Transition Matrix

Every correction policy must define a total transition matrix across assertion states. An omitted transition is illegal and must emit `GOLD_CORRECTION_TRANSITION_UNDEFINED`.

## Late Arrival

Late evidence must be routed through `EvaluateLateArrival` using an active late-arrival policy artifact, temporal resolution, source authority, completeness, and watermark state. Silently discarding late authoritative evidence in production is forbidden.

## Replay Contract

Production replay must validate required replay policy artifact refs, then run `ReplayInputSufficiencyCheck`, `ReplayEquivalencePolicy`, `DecideReplayMode`, and `ComputeReplayEquivalenceChecksum` before any replay output is written.

`ReplayEquivalencePolicy` must define included fields, excluded volatile fields, hash algorithms, canonical ordering, failure precedence, and shadow-output behavior by output class. The contract table below defines active draft output classes and explicit remaining blocker rows.

## Deterministic Side Effects

Runtime randomness, wall-clock reads, generated IDs, unordered iteration, external calls, or backend-discovered values must not affect production output unless a declared `DeterministicSideEffectRecord` captures the value and replay behavior.

## Temporal and Replay Contract Details

### TemporalSemanticsPolicy catalog

| Source dataset | Observation type | Fact type | Valid-time inputs | Fallback allowance | Malformed behavior | Ambiguous behavior | Error code | Status |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| TODO | TODO | TODO | TODO | forbidden by default | deterministic temporal error | deterministic temporal error | `TEMPORAL_POLICY_UNRESOLVED` | blocking |

### KnowledgeTimeImportPolicy

| Mode | Required behavior |
| --- | --- |
| `current_import` | `known_from` is Cadastre import knowledge time from persisted evidence. |
| `historical_import` | Historical valid time may be imported, but historical known time is not reconstructed. |
| `reconstructed_source_known_time` | Allowed only when policy names source-known-time evidence and validation rows pass. |
| `rejected_reconstruction` | Default when source-known-time evidence is missing, stale, ambiguous, or untrusted. |

Production default is `current_import`. Historical known-time reconstruction is rejected unless explicit persisted source-known-time evidence exists and an active policy permits `reconstructed_source_known_time`. Current platform time may not be used for known time or valid time unless the value is captured in persisted evidence before derivation.

### BitemporalQueryMode catalog

| Query mode | Default valid time | Default known time | Assertion-state visibility | Page-token scope | Ordering | Audit behavior |
| --- | --- | --- | --- | --- | --- | --- |
| `current` | current platform time | current platform time | active and policy-visible stale | query checksum plus auth context | owner-defined canonical sort | audit optional by API class |
| `valid_at` | supplied | current platform time | active/stale/conflicted per policy | includes valid time | canonical sort | audit required for auditor |
| `known_at` | current platform time | supplied | as-known facts only | includes known time | canonical sort | audit required |
| `corrected_history` | supplied or interval | supplied or current | includes superseded/retracted when requested | includes both times and assertion filter | canonical sort | audit required |

### GoldFactCorrectionPolicy

| Fact type/predicate | Comparable keys | Overlap behavior | Interval split | Confidence-only change | Delete evidence | Conflict behavior | Stale behavior | No-op behavior | Sort keys | Status |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| TODO | `gold_fact_key_id` plus fact-type-specific semantic fields behind it | TODO | TODO | TODO | TODO | TODO | TODO | emit explicit no-op | TODO | blocking |

### CorrectionSnapshotRefPolicy

| Correction class | Old snapshot role | New snapshot role | Table-set checksum | Retention protection | Mutable-ref behavior | Status |
| --- | --- | --- | --- | --- | --- | --- |
| `fact_replacement` | required prior fact snapshot | required candidate fact snapshot | required | both protected | reject | blocking until fact rows exist |
| `fact_retraction` | required prior fact snapshot | required retraction evidence snapshot | required | both protected | reject | blocking until source-specific rows exist |
| `interval_split` | required prior interval snapshot | required split candidate snapshot | required | both protected | reject | blocking until fact rows exist |
| `confidence_only_change` | required prior confidence snapshot | required new evidence snapshot | required | both protected | reject | blocking until fact rows exist |

### LateArrivalPolicy

| Dataset | Source authority class | Fact type | Allowed lateness | Route | Correction behavior | Quarantine behavior | Watermark effect | Status |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| TODO | TODO | TODO | TODO | evaluate through policy | append-only correction when permitted | quarantine when unresolved | no advancement unless `060` permits | blocking |

### ReplayEquivalencePolicy

| Output class | Included fields | Excluded volatile fields | Hash algorithm | Canonical ordering | Failure precedence | Shadow-output rules | Status |
| --- | --- | --- | --- | --- | --- | --- | --- |
| raw | manifest refs, raw IDs, payload hashes, import profile | run correlation ID | `sha256` | canonical record ID | missing refs before checksum mismatch | compare only | active draft |
| silver | raw refs, mapping bundle, schema profile, normalized checksum | diagnostics correlation ID | `sha256` | observation ID | schema mismatch before output mismatch | compare only | active draft |
| identity | evidence refs, resolver profile, decision, explanation checksum | reviewer display labels | `sha256` | decision ID | blocker mismatch before output mismatch | compare only | active draft |
| gold | temporal resolution, authority, evidence, fact bytes, correction policy | processing timestamp | `sha256` | fact ID and known interval | authority mismatch before output mismatch | compare only | active draft |
| graph delta/apply/rebuild | projection profile, delta set, idempotency, backend evidence, rebuild manifest | backend transient IDs | `sha256` | delta ID and path order | schema mismatch before output mismatch | compare only | active draft |
| projections/API/analysis | owner refs, labels, authorization/redaction policy, output checksum | request correlation ID | `sha256` | owner-defined | authorization mismatch before output mismatch | compare only | TODO rows required |

Replay-equivalence ownership split:

| Output subset | Checksum-rule owner | Required behavior |
| --- | --- | --- |
| graph/API replay | `090` and `110` for included/excluded fields; `080` for checksum algorithm and preflight ordering | Missing owner row blocks replay before output. |
| analysis replay | `130` for included/excluded fields; `080` for checksum algorithm and preflight ordering | Missing owner row blocks replay before output. |
| gold/correction replay | `080` | Missing temporal, authority, evidence, correction, or snapshot refs fails before checksum comparison. |

### DeterministicSideEffectPolicy

| Side-effect kind | Default |
| --- | --- |
| runtime randomness | rejected unless recorded |
| wall-clock read | rejected unless policy names clock field and records value |
| generated ID | rejected unless derived by `040.CanonicalIdPolicy` |
| unordered iteration | rejected |
| external call | rejected in production unless explicitly validation-only |
| backend-discovered value | replay from recorded value only when owner permits |

### Assertion-state transition matrix

| Prior state | Candidate state | Evidence class | Authority requirement | Output | Error/no-op | Graph handoff effect |
| --- | --- | --- | --- | --- | --- | --- |
| `active` | `superseded` | stronger correction evidence | source authority row | close prior known interval and emit new fact | error if policy missing | graph correction handoff |
| `active` | `retracted` | authorized delete/retraction evidence | authority plus completeness | close prior and emit retraction | no-op if evidence duplicate | graph retraction handoff |
| `stale` | `active` | current authoritative evidence | authority plus staleness policy | emit new active fact | no-op if same evidence | graph update handoff |
| `conflicted` | `active` | conflict resolution evidence | authority plus correction policy | emit resolved fact | error if unresolved | graph update handoff |
| any | TODO | TODO | TODO | TODO | `GOLD_CORRECTION_TRANSITION_UNDEFINED` | no mutation |

### Replay input sufficiency matrix

| Output class | Required refs | Failure code | Mutable-ref behavior | Retention-ineligible behavior | Schema mismatch behavior |
| --- | --- | --- | --- | --- | --- |
| activation-controlled policy artifacts | temporal, knowledge-time, late-arrival, correction, snapshot-ref, replay-equivalence, and output-class rows | owner-specific artifact error | reject before replay output | reject before replay output | reject before replay output |
| raw | feed profile, manifest, object/table refs, import profile | `REPLAY_INPUT_MISSING` | reject | reject | reject |
| silver | raw refs, parser/mapping, external schema artifact | `REPLAY_SCHEMA_MISMATCH` | reject | reject | reject |
| identity | resolver profile, evidence refs, version manifest | `REPLAY_AUTHORITY_MISMATCH` | reject | reject | reject |
| gold/correction | temporal, authority, correction, snapshot refs | `REPLAY_INPUT_INSUFFICIENT` | reject | reject | reject |
| graph | projection, delta, apply, rebuild, schema refs | `REPLAY_GRAPH_MISMATCH` | reject | reject | reject |

### Temporal artifact errors

| Error code | Emitted when |
| --- | --- |
| `TEMPORAL_ARTIFACT_MISSING` | Required temporal, knowledge-time, late-arrival, correction, snapshot-ref, or replay artifact ref is missing. |
| `TEMPORAL_ARTIFACT_INACTIVE` | Required temporal artifact is not active for production execution. |
| `TEMPORAL_ARTIFACT_CHECKSUM_MISMATCH` | Required temporal artifact checksum mismatches the active ref or manifest. |
| `REPLAY_POLICY_ARTIFACT_MISSING` | Required replay equivalence or output-class policy artifact is absent before replay. |

### Acceptance Criteria

| ID | Criterion |
| --- | --- |
| `080-CLEANUP-AC-001` | No banned reference class remains. |
| `080-CLEANUP-AC-002` | Source event time, observation time, supplier collection time, supplier delivery time, lakehouse commit time, table snapshot time, CDC time, graph apply time, replay time, and platform current time remain distinct. |
| `080-CLEANUP-AC-003` | `ResolveFactTime` still rejects implicit current-time fallback. |
| `080-CLEANUP-AC-004` | Corrections remain append-only knowledge transitions and must not mutate `GoldFact` rows in place. |
| `080-SCHEMA-PATCH-AC-001` | Every emitted `GoldFact` passes `040.GoldFactSchema`. |
| `080-SCHEMA-PATCH-AC-002` | `gold_fact_key_id` and `gold_fact_id` are both present and computed by the declared algorithms. |
| `080-SCHEMA-PATCH-AC-003` | Corrections never mutate original `GoldFact` bytes. |
| `080-SCHEMA-PATCH-AC-004` | Replay rejects gold output when any 040 schema, temporal, authority, evidence, or correction checksum mismatches. |

| `080-KNOWLEDGE-TIME-AC-001` | Production known time defaults to `current_import`, and historical known-time reconstruction is rejected unless policy and persisted source-known-time evidence permit it. |
| `080-REPLAY-SPLIT-AC-001` | Projection, API, and analysis replay rows import owner-specific included/excluded fields and use `080` only for checksum algorithm and preflight ordering. |
| `080-VOLATILITY-AC-001` | Inactive temporal policy, missing knowledge-time policy, correction policy checksum mismatch, late-arrival row manifest omission, and replay policy row absence fail before gold or replay output. |
| `080-VOLATILITY-AC-002` | Current `TODO:` temporal and replay rows block affected production outputs until active artifact rows and validation refs exist. |

## Definition of Done

| ID | Criterion |
| --- | --- |
| `080-AC-001` | Current, valid-at, known-at, corrected-history, replay, and audit queries are reconstructable from persisted records. |
| `080-AC-002` | Every gold fact candidate has a temporal resolution row or deterministic temporal error. |
| `080-AC-003` | Corrections never mutate existing `GoldFact` rows in place. |
| `080-AC-004` | Replay rejects missing, mutable-only, retention-ineligible, schema-incompatible, authority-mismatched, or checksum-mismatched inputs before writing output. |
| `080-AC-005` | Event-sequence validation covers temporal resolution, late arrival, correction, replay, side effects, and graph rebuild equivalence. |

## Open Questions

Open questions marked `TODO:` block authoritative status for the affected contract. A downstream implementation must not resolve a `TODO:` by inference.
