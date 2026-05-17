---
doc_id: CADASTRE-NLSPEC-090
title: Graph Projection, Serving, and Backends
doc_type: authoritative-nlspec
status: migration_active
generated_on: 2026-05-17
source_prd: PRD-Cadastre.md
source_prd_sha256: b17ac5d44618c43a57efe8ebd9b6c6e0bd8debc949b513368383c515c07f9748
---

# Graph Projection, Serving, and Backends

## Authority

This document is a generated Cadastre NLSpec candidate. It is `migration_active` until the migration ledger marks its source rows complete and `docs/nlspec/120-validation-fixtures-and-acceptance.md` records passing acceptance evidence.

This document owns the contracts listed in `Exports`. Other active Cadastre NLSpecs may import those contracts by exact name and must not restate them.

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

## Definition of Done

| ID | Criterion |
| --- | --- |
| `090-AC-001` | Graph state is rebuildable from authoritative lakehouse records and persisted graph deltas. |
| `090-AC-002` | Query results are deterministically ordered and never depend on backend natural order or internal IDs. |
| `090-AC-003` | Graph apply rejects stale schema fingerprints and missing constraints before mutation. |
| `090-AC-004` | Graph drift checks cannot repair or mutate state. |
| `090-AC-005` | MVP graph profiles cannot emit `has_theoretical_reachability`, modeled reachability facts, or equivalent graph properties. |

## Source Traceability

| Source | Section or artifact | Location |
| --- | --- | --- |
| PRD-Cadastre.md | `GraphNodeDelta` | lines 2928-2947 |
| PRD-Cadastre.md | `GraphEdgeDelta` | lines 2948-2977 |
| PRD-Cadastre.md | `GraphProjectionProfile` | lines 4728-4921 |
| PRD-Cadastre.md | `GraphDeltaSet` | lines 4922-4999 |
| PRD-Cadastre.md | `GraphDeltaIdempotencyKey` | lines 4977-4999 |
| PRD-Cadastre.md | `StructuralGlobalNode` | lines 5000-5031 |
| PRD-Cadastre.md | `GraphPropertyEvidencePolicy` | lines 5651-5678 |
| PRD-Cadastre.md | `GraphEdgeSemantics` | lines 6638-6762 |
| PRD-Cadastre.md | `GraphTraversalClass` | lines 6708-6762 |
| PRD-Cadastre.md | `GraphTaxonomyTranslationPolicy` | lines 6763-6826 |
| PRD-Cadastre.md | `StructuralGlobalNodeAliasPolicy` | lines 6827-6866 |
| PRD-Cadastre.md | `GraphBackendProfile and GraphBackendPreflightResult` | lines 7738-7818 |
| PRD-Cadastre.md | `BackendSchemaFingerprint` | lines 7819-7886 |
| PRD-Cadastre.md | `GraphBackendTaxonomyMappingProfile` | lines 7887-7931 |
| PRD-Cadastre.md | `GraphQueryTranslationProfile` | lines 7932-7991 |
| PRD-Cadastre.md | `GraphReadModelSchemaProfile` | lines 7992-8043 |
| PRD-Cadastre.md | `GraphApplyProfile and GraphApplyResult` | lines 8044-8203 |
| PRD-Cadastre.md | `GraphReadModelDriftCheck` | lines 8390-8433 |
| PRD-Cadastre.md | `Projection Interface` | lines 9730-9760 |
| PRD-Cadastre.md | `Graph Apply Interface` | lines 9761-9778 |
| PRD-Cadastre.md | `Graph Backend Adapter Interface` | lines 9779-9803 |
| PRD-Cadastre.md | `Query API Interface` | lines 9845-9883 |
| PRD-Cadastre.md | `Graph Rebuild Interface` | lines 10153-10173 |
| PRD-Cadastre.md | `Incremental Projection Requirements` | lines 11318-11582 |
| PRD-Cadastre.md | `Graph Query` | lines 1108-1261 |
| PRD-Cadastre.md | `Future Non-MVP Reachability Analysis Domain` | lines 8752-9160 |
| Decomposition plan | Current user prompt | Domain decomposition, disposition matrix, dependency model, gap ledger, and migration acceptance criteria. |

## Open Questions

Open questions marked `TODO:` block authoritative status for the affected contract. A downstream implementation must not resolve a `TODO:` by inference.
