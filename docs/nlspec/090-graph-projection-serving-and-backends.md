---
doc_id: CADASTRE-NLSPEC-090
title: Graph Projection, Serving, and Backends
doc_type: nlspec
status: candidate
---

## Authority

This document owns the contracts listed in `Exports`. Other Cadastre NLSpecs may import those contracts by exact name and must not restate them. This document has implementation authority only after the document registry marks it `authoritative` and its acceptance criteria pass.

## Purpose

Define graph read-model construction, graph apply, graph backend profiles, graph query behavior, rebuild, lag, and drift without making the backend authoritative.

## Explicit Non-Scope

- Gold fact authority.
- Identity decisions.
- Source completeness.
- Graph backend product selection.

## Imports

- `AuthorityClass`
- `GoldFact`
- `IdentityDecision`
- `SourceAuthorityProfile`
- `GoldFactChangeSet`
- `PackageStageBinding`

- `GraphNodeDeltaShape`
- `GraphEdgeDeltaShape`
- `EvidenceRef`
- `CoreRecordValidationAlgorithm`
- `ActivationControlledArtifactRef`

## Exports

- `GraphProjectionProfile`
- `GraphNodeDelta`
- `GraphEdgeDelta`
- `GraphDeltaSet`
- `GraphDeltaIdempotencyKey`
- `GraphPropertyEvidencePolicy`
- `GraphEdgeSemantics`
- `GraphTraversalClass`
- `GraphTaxonomyTranslationPolicy`
- `StructuralGlobalNode`
- `StructuralGlobalNodeAliasPolicy`
- `GraphBackendProfile`
- `GraphBackendPreflightResult`
- `BackendSchemaFingerprint`
- `GraphBackendTaxonomyMappingProfile`
- `GraphQueryTranslationProfile`
- `GraphReadModelSchemaProfile`
- `GraphApplyProfile`
- `GraphApplyResult`
- `GraphRebuildManifest`
- `GraphIndexConsistencyCheck`
- `DerivedViewLagPolicy`
- `DerivedViewState`
- `QueryGraph`
- `ApplyGraphDelta`
- `ProjectGraphDeltas`
- `RunGraphReadModelDriftCheck`
- `RebuildGraph`
- `GraphApplyLifecycleMachine`

## Projection Authority

Graph state is a derived read model. It must be rebuildable from authoritative lakehouse records, persisted graph deltas, and active projection/apply/schema profiles.

`GraphNodeDelta` and `GraphEdgeDelta` are concrete 090 records that must conform to `040.GraphNodeDeltaShape` and `040.GraphEdgeDeltaShape` before apply. `090` owns projection/apply semantics and runtime graph delta ID input policies.

A graph backend must not create authoritative facts, infer identity, decide source completeness, decide fact retraction, define bitemporal semantics, repair drift by itself, or expose backend internal IDs as Cadastre IDs.

### GraphExpirySourceAuthorityGate

`GraphProjectionProfile` may emit graph expiry or cleanup deltas only when the source `GoldFactChangeSet` or `060.AbsenceDerivationResult` authorizes `graph_expiry` or `cleanup` for the requested effect token.

`GraphApplyResult`, `GraphIndexConsistencyCheck`, `DerivedViewState`, graph backend import success, graph drift check, missing graph object, graph apply time, graph backend cleanup, or derived-view lag must not authorize graph expiry or cleanup.

`source_stale`, `derived_view_stale`, `unknown`, `permission_limited`, `partial_known_gap`, `partial_unknown_gap`, `source_unavailable`, `scope_unavailable`, `not_checked`, `error`, and `ambiguous` must produce no graph expiry unless an exact active `060` row authorizes the requested effect.

Missing flow evidence must not emit an observed-connection absence edge or graph expiry. It maps to `unknown` and produces no graph mutation.

### GoldFactChangeSet graph handoff matrix

`090.ProjectGraphDeltas` consumes `080` graph handoff effects. `080` emits metadata only; graph projection remains a deterministic derived view.

| `080` graph handoff effect | Required `090` projection behavior | Required authorization refs | Default when unsupported |
| --- | --- | --- | --- |
| `none` | Emit no graph delta. | `GoldFactChangeSet` ref only. | no-op. |
| `reproject_fact_key` | Recompute projected graph objects for the affected fact key using the active `GraphProjectionProfile`. | `GoldFactChangeSet` ref, temporal refs, projection profile ref. | `GRAPH_HANDOFF_EFFECT_UNSUPPORTED`. |
| `expire_projected_object` | Emit expiry delta only when `060.AbsenceDerivationResult.allowed_effects` contains `graph_expiry`. | `GoldFactChangeSet` ref and `060.AbsenceDerivationResult` ref. | no graph delta plus `GRAPH_EXPIRY_SOURCE_AUTHORITY_REQUIRED`. |
| `cleanup_projected_object` | Emit cleanup delta only when `060.AbsenceDerivationResult.allowed_effects` contains `cleanup`. | `GoldFactChangeSet` ref and `060.AbsenceDerivationResult` ref. | no graph delta plus `GRAPH_EXPIRY_SOURCE_AUTHORITY_REQUIRED`. |
| `conflict_visibility_update` | Update graph visibility only when `GraphEdgeSemantics` and `GraphObjectOutputEligibilityRow` permit conflicted or stale assertions. | `GoldFactChangeSet` ref, graph semantics row ref, eligibility row ref. | no pathfinding-visible delta. |
| `identity_split_handoff` | Consume `070.GraphCorrectionHandoff` and route through projection only; resolver and correction engine do not mutate graph. | `GraphCorrectionHandoff` ref, affected fact refs, projection profile ref. | `GRAPH_HANDOFF_EFFECT_UNSUPPORTED`. |

Every projected delta from correction must include the source `GoldFactChangeSet` ref, graph handoff effect, projection profile ref, authority or absence result refs when applicable, and validation refs.

## Graph artifact activation boundary

Graph authority boundaries, deterministic graph IDs, backend-ID prohibition, query contract, and projection non-authority are stable core contracts. Projection rows, backend profiles, schema profiles, query translation rows, apply profiles, and lag policies are activation-controlled artifacts.

| Artifact or contract | Volatility class | Required handling |
| --- | --- | --- |
| `GraphProjectionProfile` | `activation_controlled_artifact` | Required before graph deltas are projected. |
| `GraphEdgeSemantics` row set | `activation_controlled_artifact` | Must instantiate stable edge semantics contract. |
| `GraphTraversalClass` | stable closed semantics plus activation-controlled inclusion rows where applicable | Parallel traversal eligibility fields are forbidden. |
| `GraphTaxonomyTranslationPolicy` | `activation_controlled_artifact` | Required before external graph taxonomy labels are used. |
| `StructuralGlobalNode` rows | `activation_controlled_artifact` | Disabled by default. |
| `StructuralGlobalNodeAliasPolicy` | `activation_controlled_artifact` | Required before aliases or wildcard-like globals. |
| `GraphBackendProfile` | `activation_controlled_artifact` | Required before mutation, query, rebuild, drift check, or serving promotion. |
| `GraphBackendPreflightResult` | `runtime_state_record` | Runtime validation state. |
| `BackendSchemaFingerprint` | `runtime_state_record` | Runtime schema state. |
| `GraphBackendTaxonomyMappingProfile` | `activation_controlled_artifact` | Required before backend labels/properties are accepted. |
| `GraphQueryTranslationProfile` | `activation_controlled_artifact` | Required before query execution. |
| `GraphReadModelSchemaProfile` | `activation_controlled_artifact` | Required before apply or rebuild promotion. |
| `GraphApplyProfile` | `activation_controlled_artifact` | Required before graph apply. |
| `GraphApplyResult` | `runtime_state_record` | Runtime apply state. |
| `GraphRebuildManifest` | `runtime_state_record` | Rebuild evidence, not authority by itself. |
| `GraphIndexConsistencyCheck` | `runtime_state_record` | Runtime validation state. |
| `DerivedViewLagPolicy` | `activation_controlled_artifact` | Required before serving stale or current graph state. |
| `DerivedViewState` | `runtime_state_record` | Runtime serving state. |

`ProjectGraphDeltas`, `ApplyGraphDelta`, `QueryGraph`, and `RebuildGraph` must validate required activation artifact refs through `030.ActivationControlledArtifactRef` and include every output-affecting ref in `VersionManifest`.

## Graph Delta Identity

Every graph node and edge must have a Cadastre-owned deterministic ID. Backend-generated node, edge, relationship, vertex, document, element, transaction, shard, or native cursor IDs are forbidden as Cadastre IDs, selectors, evidence refs, replay keys, drillback keys, response IDs, or pagination identity and must fail with `GRAPH_BACKEND_ID_FORBIDDEN` before graph apply, query response, evidence ref generation, replay, or pagination.

## Edge Semantics and Traversal

Every active graph edge type must have exactly one `GraphEdgeSemantics` row. The row must define source fact type, source predicate, direction rule, evidence requirements, allowed assertion states, temporal policy, confidence policy, traversal class, non-implication rules, and no-op conditions.

`GraphTraversalClass` is the only traversal eligibility authority. Parallel fields such as `traversal_eligibility`, `reverse_traversal_eligibility`, or `pathfinding_role` are forbidden.

OCSF `src_endpoint`, `dst_endpoint`, `network_endpoint`, DNS endpoint, DHCP endpoint, or any external schema endpoint order is not `FlowRoleEvidence`. A projection row that has OCSF endpoint fields but lacks qualifying `FlowRoleEvidence` must emit no `observed_connection` edge and must emit `GRAPH_FLOW_ROLE_EVIDENCE_REQUIRED` when the projection attempt is observable.

## Backend Preflight

A `GraphBackendProfile` must be active before graph mutation, query, rebuild import, drift check, or graph-serving promotion. `GraphBackendPreflightResult` must validate backend, driver, dialect, topology, storage mode, feature availability, raw-write bypass controls, schema profile, and query translation readiness.

## ProjectGraphDeltas Algorithm

```text
ProjectGraphDeltas(change_set, projection_profile):
1. Validate `GoldFactChangeSet`, graph handoff effect, projection profile, graph semantics rows, and output eligibility rows.
2. If handoff effect is `none`, emit no `GraphDeltaSet` and record a no-op projection diagnostic.
3. If handoff effect is `reproject_fact_key`, recompute projected objects for the affected fact key from authoritative gold inputs and active projection rows.
4. If handoff effect is `expire_projected_object`, require `060.AbsenceDerivationResult.allowed_effects` containing `graph_expiry` before emitting expiry delta.
5. If handoff effect is `cleanup_projected_object`, require `060.AbsenceDerivationResult.allowed_effects` containing `cleanup` before emitting cleanup delta.
6. If handoff effect is `conflict_visibility_update`, emit only visibility deltas permitted by `GraphEdgeSemantics` and `GraphObjectOutputEligibilityRow`.
7. If handoff effect is `identity_split_handoff`, require `070.GraphCorrectionHandoff` and route the affected fact refs through projection.
8. Reject unsupported handoff effects with `GRAPH_HANDOFF_EFFECT_UNSUPPORTED` before delta persistence.
9. Persist graph delta refs, source change-set refs, handoff effect refs, projection profile refs, authority refs, and validation refs in `VersionManifest`.
```

## ApplyGraphDelta Algorithm

```text
ApplyGraphDelta(delta_set, apply_profile):
1. Validate graph projection, backend, schema, taxonomy, query translation, apply, and lag artifact refs through `030.ActivationControlledArtifactRef`.
2. Validate every node and edge delta against `040.GraphNodeDeltaShape`, `040.GraphEdgeDeltaShape`, and `040.ValidateCoreRecord`.
3. Reject backend-generated IDs and raw payload leakage before backend execution.
4. Reject graph expiry or cleanup deltas that lack an exact `060` authorization ref and requested effect token.
5. Verify delta_set checksum and GraphDeltaIdempotencyKey.
6. Verify active GraphBackendProfile, GraphReadModelSchemaProfile, and BackendSchemaFingerprint.
7. Reject when backend schema is missing, stale, or fingerprint-mismatched.
8. Sort deltas by apply_profile canonical order.
9. Apply batches only at declared transaction boundaries.
10. Persist backend evidence rows for transaction semantics, failover, read-after-write, storage mode, index freshness, and writer identity when correctness-affecting.
11. On failure, record committed batch IDs or prove no committed writes.
12. Return GraphApplyResult with status, errors, input checksum, backend evidence, idempotency state, artifact refs, and derived view state update eligibility.
```

## Graph Apply Lifecycle

Machine ID: `090.GraphApplyLifecycleMachine.v1`.

States:

```text
prepared
preflighted
applying
partial_committed
applied
failed_no_commit
failed_partial
resume_pending
no_op
```

Events:

```text
preflight_passed
preflight_failed
start_apply
batch_committed
batch_failed_before_commit
batch_failed_after_commit
retryable_backend_error
retry_exhausted
resume_requested
resume_safe
resume_unsafe
identical_reapply
idempotency_conflict
complete
replay_same_event
```

| Prior state | Event | Next state | Required behavior |
| --- | --- | --- | --- |
| `prepared` | `preflight_passed` | `preflighted` | Persist preflight evidence. |
| `prepared` | `preflight_failed` | `failed_no_commit` | No backend writes. |
| `preflighted` | `start_apply` | `applying` | Persist apply-start evidence. |
| `applying` | `batch_committed` | `applying` or `partial_committed` | Persist committed-batch evidence. |
| `applying` | `complete` | `applied` | Persist `GraphApplyResult`; derived-view update eligible. |
| `applying` | `batch_failed_before_commit` | `failed_no_commit` | No derived-view advancement. |
| `applying` | `batch_failed_after_commit` | `failed_partial` | Persist committed-batch proof; no derived-view advancement. |
| `failed_partial` | `resume_requested` | `resume_pending` | Same idempotency key required. |
| `resume_pending` | `resume_safe` | `applying` | Resume after last compatible committed batch. |
| `resume_pending` | `resume_unsafe` | `failed_partial` | Emit `GRAPH_APPLY_RESUME_UNSAFE`; no mutation. |
| `applied` | `identical_reapply` | `no_op` | Idempotent no-op. |
| any state | `idempotency_conflict` | same state | Emit `GRAPH_DELTA_IDEMPOTENCY_CONFLICT`; no mutation. |
| terminal repeated identical event | `replay_same_event` | same state | Existing evidence or byte-identical no-op. |
| `all_other_state_event_pairs` | any other event | same state | Illegal transition evidence; no mutation. |

Failed, partial, aborted, or schema-preflight-failed graph apply must not advance derived-view or watermark state. Backend transaction behavior must be represented as persisted evidence; it must not become lifecycle authority.

## QueryGraph Contract

Graph query responses must be sorted by Cadastre-defined ordering, never backend natural order.

Path ordering:

```text
1. shortest path length
2. highest minimum edge confidence, descending
3. most recent maximum edge last_seen, descending
4. path node ID sequence lexical order
```

Graph query bounds:

| Parameter | Default | Minimum | Maximum |
| --- | ---: | ---: | ---: |
| `max_depth` | 3 | 1 | 6 |
| `page_size` | 100 | 1 | 1000 |
| `timeout_seconds` | 30 | 1 | 300 |

A query with an empty `allowed_traversal_classes` set must return no paths rather than traversing all edges.

## Derived View Lag

Every graph-serving response using graph read-model state must include or reference `DerivedViewState`. Compliance, audit, and production rule queries must reject stale graph state by default with `DERIVED_VIEW_LAG_ERROR`.

## Graph Rebuild

`RebuildGraph` must use authoritative lakehouse inputs, persisted graph deltas, active projection/apply/schema profiles, and a declared `GraphRebuildEquivalencePolicy`. Rebuild promotion requires `GraphRebuildManifest` and passing `GraphIndexConsistencyCheck`.

## Drift Check Boundary

`GraphReadModelDriftCheck` is non-authoritative operational health only. A drift check that attempts repair, graph mutation, graph delta emission, authoritative record mutation, or watermark advancement must fail with `GRAPH_DRIFT_REPAIR_FORBIDDEN`.

## Graph Projection Contract Details

### GraphNodeDeltaIdPolicy and GraphEdgeDeltaIdPolicy

`GraphNodeDeltaIdPolicy` and `GraphEdgeDeltaIdPolicy` are owned by `090` because graph runtime identity inputs depend on projection and apply semantics. The policies must use `040.CanonicalJSON`, must not include backend-generated IDs, and must produce byte-identical IDs across replay for the same `GraphDeltaSet`.

| Policy | Required inputs | Forbidden inputs | Missing behavior | Collision behavior |
| --- | --- | --- | --- | --- |
| `GraphNodeDeltaIdPolicy` | `projection_profile_id`, `graph_delta_set_id`, `delta_operation`, `graph_node_id`, `node_type`, `source_object_ref`, valid/known interval, assertion state | backend node ID, element ID, storage ID, cursor ID | reject before apply | `GRAPH_NODE_DELTA_ID_COLLISION`; emit no commit before graph apply |
| `GraphEdgeDeltaIdPolicy` | `projection_profile_id`, `graph_delta_set_id`, `delta_operation`, `graph_edge_id`, `edge_type`, endpoint graph node IDs, direction rule, valid/known interval, assertion state | backend relationship ID, edge storage ID, cursor ID | reject before apply | `GRAPH_EDGE_DELTA_ID_COLLISION`; emit no commit before graph apply |

`properties` defaults to `{}`. Every property path must pass `GraphPropertyEvidencePolicy`, redaction checks, and raw payload leakage checks. Non-`no_op` deltas must have evidence refs unless an active graph profile explicitly permits structural synthetic output. `observed_connection` direction must derive from `FlowRoleEvidence`, not OCSF endpoint field order or backend edge convention. `has_theoretical_reachability` remains inactive and fail/no-op in MVP.

### GraphEdgeSemanticsRegistry

| Edge type | Source fact type | Predicate | Direction rule | Evidence | Assertion states | Temporal policy | Confidence policy | Traversal class | Non-implication | No-op | Validation rows | Status |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `observed_connection` | `observed_network_flow_fact` | observed flow relation | `FlowRoleEvidence` only; OCSF endpoint order is not qualifying evidence | gold, silver, raw refs | active, stale when allowed | `080` | `090` imports confidence from source fact and `040.DecimalPrecisionPolicy` | `observed_connection_path` | does not imply theoretical reachability | no edge when `FlowRoleEvidence` is missing or ambiguous, even when OCSF endpoint fields are present | `val-090-observed-connection-positive`, `val-090-observed-connection-missing-flow-role`, `val-090-ocsf-endpoint-order-no-direction` | active_mvp |
| `has_theoretical_reachability` | none in MVP | none | none | none | none | none | none | none | prohibited | fail/no-op | `val-090-theoretical-reachability-prohibited` | inactive_deferred |

The MVP active graph edge set is exactly `observed_connection`. Any additional active edge type requires a new `GraphEdgeSemanticsRegistry` row and validation fixtures before projection activation.

### GraphTraversalClass table

| Traversal class | Pathfinding eligibility | Neighbor expansion eligibility | Reverse traversal behavior | Default inclusion | No-op behavior |
| --- | --- | --- | --- | --- | --- |
| `observed_connection_path` | allowed when query names this class | allowed when edge type filter permits | reverse allowed only as a query traversal over the same observed edge; it must not reverse source direction semantics | excluded unless explicitly requested for path queries | empty allowed traversal classes return no paths |
| `non_traversable` | forbidden | profile-defined display only | none | excluded | edge may be returned as property/detail but not traversed |
| `analysis_only` | forbidden unless `130.RuleGraphCompatibilityMatrix` permits read-only analysis query | read-only | none unless analysis rule permits | excluded | no production graph mutation |

### GraphApplyProfile defaults

| Setting | Default | Minimum | Maximum | Rule |
| --- | ---: | ---: | ---: | --- |
| `max_batch_deltas` | 1000 | 1 | 5000 | May be lowered per backend; may be raised above 1000 only by a validated profile row. |
| `retry_max_attempts` | 0 | 0 | 3 | Retry is disabled unless retryable error classes are explicitly declared. |
| `retryable_error_classes` | `[]` | n/a | closed owner set | No driver or backend implicit retry-class inheritance. |
| `retry_delay_schedule_seconds` | `[]` | n/a | `[1, 2, 4]` | Used only when retries are enabled. |
| `transaction_boundary` | one declared batch | n/a | n/a | Commit only at the declared boundary. |
| `partial_apply_behavior` | fail closed unless resume evidence is complete | n/a | n/a | Unsafe resume emits `GRAPH_APPLY_RESUME_UNSAFE`. |
| `read_after_write_proof` | required | n/a | n/a | Missing proof blocks derived-view advancement. |
| `schema_stale_behavior` | reject | n/a | n/a | Emit `GRAPH_SCHEMA_FINGERPRINT_STALE`. |

### GraphQueryTranslationProfile observable contract

| Field | Required behavior |
| --- | --- |
| stable page token fields | Query checksum, authorization context checksum, ordering keys, page boundary, derived-view state, expiration, and profile refs. |
| backend candidate limit | Must be explicit per query class; TODO owner decision blocks production query activation. |
| timeout behavior | Return `GRAPH_QUERY_TIMEOUT` unless partial results are explicitly permitted for the query class. |
| empty traversal behavior | Empty `allowed_traversal_classes` returns no paths. |
| authorization and redaction | Applied after backend candidate materialization and before response emission. |

### GraphRebuildManifest schema

| Field | Required | Rule |
| --- | ---: | --- |
| `input_snapshot_refs` | Yes | Authoritative lakehouse snapshot or dataset refs. |
| `graph_delta_set_refs` | Yes | Persisted graph delta set refs and checksums. |
| `projection_profile_ref` | Yes | Active graph projection profile. |
| `backend_profile_ref` | Yes | Active backend profile. |
| `schema_fingerprint` | Yes | Current backend schema fingerprint. |
| `output_checksum` | Yes | SHA-256 over canonical rebuilt graph-serving output summary. |
| `status` | Yes | success, failed, blocked, or not_run. |
| `errors` | Yes | Default `[]`; owner-specific graph rebuild errors. |

### GraphObjectOutputEligibilityRow

| Object class | Search | Neighbor expansion | Pathfinding | Analysis finding | Metrics | Identity influence |
| --- | --- | --- | --- | --- | --- | --- |
| projected canonical node | profile-defined | profile-defined | traversal-class-defined | read-only | allowed if profile permits | no direct identity mutation |
| projected edge | profile-defined | profile-defined | traversal-class-defined | read-only | allowed if profile permits | no identity authority |
| synthetic structural object | disabled by default | disabled by default | disabled by default | disabled by default | disabled by default | none |
| generic external graph payload | disabled by default | disabled by default | disabled by default | disabled by default | disabled by default | none |

### GraphDeltaIdempotencyKey inputs

| Input order | Input | Required | Rule |
| ---: | --- | ---: | --- |
| 1 | `graph_delta_set_checksum` | Yes | SHA-256 lowercase hex. |
| 2 | `graph_apply_profile_id` | Yes | Active profile. |
| 3 | `backend_profile_id` | Yes | Active profile. |
| 4 | `backend_schema_fingerprint` | Yes | Current and non-stale. |
| 5 | `package_set_manifest_checksum` | Yes | Immutable package set. |
| 6 | `target_graph_scope` | Yes | Canonical JSON scope. |

### ResumeGraphApply semantics

| Prior state | Required behavior |
| --- | --- |
| no prior apply | apply all batches in canonical order. |
| prior identical success | return idempotent no-op result. |
| prior failed before commit | reapply from first batch. |
| prior partial with committed batch evidence | resume after last committed compatible batch. |
| prior partial without committed-batch proof | fail with `GRAPH_APPLY_RESUME_UNSAFE`. |
| idempotency key conflict | fail with `GRAPH_DELTA_IDEMPOTENCY_CONFLICT`. |

### GraphBackendProfile activation fixture matrix

| Backend fixture area | Required evidence | Activation effect |
| --- | --- | --- |
| backend and driver | exact backend, driver, version, dialect | blocks backend-specific activation if missing |
| topology and storage mode | writer routing, HA/storage mode, failover semantics | blocks mutation if unsupported |
| raw-write bypass | proof backend console/admin/import bypass is blocked or detected | blocks activation if unsafe |
| schema profile | constraints, indexes, selector indexes, evidence indexes | blocks mutation/query if stale |
| query translation | ordering, pagination, timeout, authorization, redaction | blocks query serving if missing |
| rebuild/import | rebuild manifest, import evidence, index consistency | blocks promotion if missing |

### GraphRebuildEquivalencePolicy boundary

`080` owns checksum rules for replay equivalence. This spec imports `GraphRebuildEquivalencePolicy` by name and defines graph-specific use: rebuild promotion must compare graph-serving output, schema fingerprint, index consistency, delta input set, projection profile, graph handoff refs, and derived-view state using the imported policy. Replay output classes `graph_delta`, `graph_apply`, and `graph_rebuild` must remain distinct and must not share owner-specific included/excluded field rows.

### CurrentRecentGraphServingPolicy

MVP graph serving covers current-state and recent-history graph queries. Full historical reconstruction defaults to lakehouse-backed reconstruction outside the graph backend unless a future authoritative graph historical serving policy exists.

### Backend preflight table

| Check | Required evidence | Failure code | Retryability | Activation effect | Validation fixture |
| --- | --- | --- | --- | --- | --- |
| backend profile active | profile checksum | `GRAPH_BACKEND_PROFILE_MISSING` | no | block | missing profile |
| schema fingerprint current | fingerprint checksum and timestamp | `GRAPH_SCHEMA_FINGERPRINT_STALE` | yes after refresh | block mutation/query | stale fingerprint |
| raw-write bypass controls | audit/control evidence | `GRAPH_RAW_WRITE_BYPASS` | no | block | bypass attempt |
| query translation parity | expected result checksums | `GRAPH_QUERY_TRANSLATION_ERROR` | no | block query serving | parity fixture |
| rebuild equivalence | rebuild checksums and index consistency | `GRAPH_REBUILD_EQUIVALENCE_FAILED` | yes after rebuild | block promotion | rebuild mismatch |

### Graph artifact errors

| Error code | Emitted when |
| --- | --- |
| `GRAPH_ARTIFACT_MISSING` | Required graph projection, backend, schema, query translation, apply, taxonomy, edge semantics, or lag artifact ref is missing. |
| `GRAPH_ARTIFACT_INACTIVE` | Required graph artifact is not active for production execution. |
| `GRAPH_ARTIFACT_CHECKSUM_MISMATCH` | Required graph artifact checksum mismatches the active ref or manifest. |
| `GRAPH_ARTIFACT_SCOPE_MISMATCH` | Required graph artifact does not cover the graph execution, backend, projection, query, or serving scope. |
| `GRAPH_PROJECTION_CORE_OVERRIDE_FORBIDDEN` | A graph projection artifact attempts to redefine core IDs, core schema, identity semantics, temporal semantics, or product authority. |
| `GRAPH_FLOW_ROLE_EVIDENCE_REQUIRED` | A projection attempts to emit an `observed_connection` edge from OCSF endpoint order or external schema endpoint fields without qualifying `FlowRoleEvidence`. |
| `GRAPH_EXPIRY_SOURCE_AUTHORITY_REQUIRED` | A graph expiry or cleanup delta lacks an exact `060` authorization ref and requested effect token. |
| `GRAPH_HANDOFF_EFFECT_UNSUPPORTED` | A `GoldFactChangeSet` graph handoff effect is not covered by the active projection profile. |

### Acceptance Criteria

| ID | Criterion |
| --- | --- |
| `090-CLEANUP-AC-001` | No banned reference class remains. |
| `090-CLEANUP-AC-002` | Graph state remains a derived read model. |
| `090-CLEANUP-AC-003` | Backend-generated IDs remain forbidden as Cadastre IDs, selectors, evidence refs, replay keys, drillback keys, response IDs, or pagination identity. |
| `090-CLEANUP-AC-004` | The default MVP graph profile still cannot emit theoretical reachability output. |
| `090-SCHEMA-PATCH-AC-001` | Every graph node delta and edge delta conforms to the corresponding 040 primitive shape before apply. |
| `090-SCHEMA-PATCH-AC-002` | Backend-generated IDs fail with `GRAPH_BACKEND_ID_FORBIDDEN`. |
| `090-SCHEMA-PATCH-AC-003` | Graph properties with undeclared provenance or raw payload leakage fail before graph apply. |
| `090-SCHEMA-PATCH-AC-004` | MVP graph profiles still cannot emit theoretical reachability output. |

| `090-EDGESET-AC-001` | The active MVP graph edge set is exactly `observed_connection`; theoretical reachability remains inactive and prohibited. |
| `090-GRAPH-ID-COLLISION-AC-001` | Graph node and edge delta ID collisions fail with `GRAPH_NODE_DELTA_ID_COLLISION` or `GRAPH_EDGE_DELTA_ID_COLLISION` before graph apply commits. |
| `090-REBUILD-MANIFEST-AC-001` | Every graph rebuild promotion has a `GraphRebuildManifest` with input refs, profile refs, schema fingerprint, output checksum, status, and errors. |
| `090-VOLATILITY-AC-001` | Inactive graph projection profile, query translation checksum mismatch, apply profile manifest omission, backend ID exposure, and attempted `has_theoretical_reachability` activation fail before mutation or serving. |
| `090-VOLATILITY-AC-002` | Graph apply default values, maximums, retry classes, and retry schedule are explicit and no backend default is inherited. |
| `090-LIFECYCLE-AC-001` | Graph apply lifecycle matrix is total. |
| `090-LIFECYCLE-AC-002` | Identical reapply produces no mutation and no duplicate graph objects. |
| `090-LIFECYCLE-AC-003` | Partial apply without committed-batch proof fails with `GRAPH_APPLY_RESUME_UNSAFE`. |
| `090-LIFECYCLE-AC-004` | Failed, partial, aborted, or schema-preflight-failed apply never advances derived-view or watermark state. |
| `090-GRAPH-APPLY-DEFAULTS-AC-001` | Graph apply defaults and maximums are explicit. |
| `090-OCSF-DIRECTION-AC-001` | OCSF endpoint order alone never determines `observed_connection` edge direction. |
| `090-SOURCE-CLOSURE-GRAPH-AC-001` | Graph expiry delta without `060` requested-effect authorization fails before apply. |
| `090-SOURCE-CLOSURE-GRAPH-AC-002` | Graph derived-view lag never maps to source staleness or source absence. |
| `090-SOURCE-CLOSURE-GRAPH-AC-003` | Missing flow evidence emits no absence edge and no expiry. |
| `090-GRAPH-HANDOFF-AC-001` | `none` handoff emits no graph delta. |
| `090-GRAPH-HANDOFF-AC-002` | `reproject_fact_key` recomputes affected projected graph objects and includes source `GoldFactChangeSet` refs. |
| `090-GRAPH-HANDOFF-AC-003` | `expire_projected_object` emits expiry only when `060` authorizes `graph_expiry`. |
| `090-GRAPH-HANDOFF-AC-004` | `cleanup_projected_object` emits cleanup only when `060` authorizes `cleanup`. |
| `090-GRAPH-HANDOFF-AC-005` | `conflict_visibility_update` is not pathfinding-eligible by default. |
| `090-GRAPH-HANDOFF-AC-006` | `identity_split_handoff` routes through projection only and does not let the resolver or correction engine mutate graph state. |
| `090-OCSF-DIRECTION-AC-002` | A `network_activity_observation` with valid OCSF Network Activity fields but missing or ambiguous `FlowRoleEvidence` emits no `observed_connection` edge. |
| `090-OCSF-DIRECTION-AC-003` | DNS and DHCP observations do not emit `observed_connection` edges unless a graph projection row and `FlowRoleEvidence` explicitly permit the edge. |

## Definition of Done

| ID | Criterion |
| --- | --- |
| `090-AC-001` | Graph state is rebuildable from authoritative lakehouse records and persisted graph deltas. |
| `090-AC-002` | Query results are deterministically ordered and never depend on backend natural order or internal IDs. |
| `090-AC-003` | Graph apply rejects stale schema fingerprints and missing constraints before mutation. |
| `090-AC-004` | Graph drift checks cannot repair or mutate state. |
| `090-AC-005` | MVP graph profiles cannot emit `has_theoretical_reachability`, modeled reachability facts, or equivalent graph properties. |
| `090-AC-006` | Graph direction for observed connections derives from Cadastre `FlowRoleEvidence`, not OCSF endpoint order or backend edge convention. |

## Open Questions

Open questions marked `TODO:` block authoritative status for the affected contract. A downstream implementation must not resolve a `TODO:` by inference.
