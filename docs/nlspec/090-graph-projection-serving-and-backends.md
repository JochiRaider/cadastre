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

## Projection Authority

Graph state is a derived read model. It must be rebuildable from authoritative lakehouse records, persisted graph deltas, and active projection/apply/schema profiles.

A graph backend must not create authoritative facts, infer identity, decide source completeness, decide fact retraction, define bitemporal semantics, repair drift by itself, or expose backend internal IDs as Cadastre IDs.

## Graph Delta Identity

Every graph node and edge must have a Cadastre-owned deterministic ID. Backend-generated node, edge, relationship, vertex, document, element, transaction, shard, or native cursor IDs are forbidden as Cadastre IDs, selectors, evidence refs, replay keys, drillback keys, response IDs, or pagination identity.

## Edge Semantics and Traversal

Every active graph edge type must have exactly one `GraphEdgeSemantics` row. The row must define source fact type, source predicate, direction rule, evidence requirements, allowed assertion states, temporal policy, confidence policy, traversal class, non-implication rules, and no-op conditions.

`GraphTraversalClass` is the only traversal eligibility authority. Parallel fields such as `traversal_eligibility`, `reverse_traversal_eligibility`, or `pathfinding_role` are forbidden.

## Backend Preflight

A `GraphBackendProfile` must be active before graph mutation, query, rebuild import, drift check, or graph-serving promotion. `GraphBackendPreflightResult` must validate backend, driver, dialect, topology, storage mode, feature availability, raw-write bypass controls, schema profile, and query translation readiness.

## ApplyGraphDelta Algorithm

```text
ApplyGraphDelta(delta_set, apply_profile):
1. Verify delta_set checksum and GraphDeltaIdempotencyKey.
2. Verify active GraphBackendProfile, GraphReadModelSchemaProfile, and BackendSchemaFingerprint.
3. Reject when backend schema is missing, stale, or fingerprint-mismatched.
4. Sort deltas by apply_profile canonical order.
5. Apply batches only at declared transaction boundaries.
6. Persist backend evidence rows for transaction semantics, failover, read-after-write, storage mode, index freshness, and writer identity when correctness-affecting.
7. On failure, record committed batch IDs or prove no committed writes.
8. Return GraphApplyResult with status, errors, input checksum, backend evidence, idempotency state, and derived view state update eligibility.
```

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

### GraphEdgeSemanticsRegistry

| Edge type | Source fact type | Predicate | Direction rule | Evidence | Assertion states | Temporal policy | Confidence policy | Traversal class | Non-implication | No-op | Validation rows | Status |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `observed_connection` | `observed_network_flow_fact` | observed flow relation | `FlowRoleEvidence` only | gold, silver, raw refs | active, stale when allowed | `080` | owner policy | explicit class only | does not imply theoretical reachability | no edge when flow role missing | TODO | blocking until exact row complete |
| `has_theoretical_reachability` | none in MVP | none | none | none | none | none | none | none | prohibited | fail/no-op | reachability negative rows | inactive_deferred |
| TODO | TODO | TODO | TODO | TODO | TODO | TODO | TODO | TODO | TODO | TODO | TODO | blocking |

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

`080` owns checksum rules for replay equivalence. This spec imports `GraphRebuildEquivalencePolicy` by name and defines graph-specific use: rebuild promotion must compare graph-serving output, schema fingerprint, index consistency, delta input set, projection profile, and derived-view state using the imported policy.

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

### Acceptance Criteria

| ID | Criterion |
| --- | --- |
| `090-CLEANUP-AC-001` | No banned reference class remains. |
| `090-CLEANUP-AC-002` | Graph state remains a derived read model. |
| `090-CLEANUP-AC-003` | Backend-generated IDs remain forbidden as Cadastre IDs, selectors, evidence refs, replay keys, drillback keys, response IDs, or pagination identity. |
| `090-CLEANUP-AC-004` | The default MVP graph profile still cannot emit theoretical reachability output. |

## Definition of Done

| ID | Criterion |
| --- | --- |
| `090-AC-001` | Graph state is rebuildable from authoritative lakehouse records and persisted graph deltas. |
| `090-AC-002` | Query results are deterministically ordered and never depend on backend natural order or internal IDs. |
| `090-AC-003` | Graph apply rejects stale schema fingerprints and missing constraints before mutation. |
| `090-AC-004` | Graph drift checks cannot repair or mutate state. |
| `090-AC-005` | MVP graph profiles cannot emit `has_theoretical_reachability`, modeled reachability facts, or equivalent graph properties. |


## Open Questions

Open questions marked `TODO:` block authoritative status for the affected contract. A downstream implementation must not resolve a `TODO:` by inference.
