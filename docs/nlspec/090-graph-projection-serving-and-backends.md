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
- Market/vendor evaluation outside active `GraphBackendProfile` rows.
- Provider-specific implementation internals except where they affect observable graph apply, query, rebuild, schema, configuration, package activation, error, or replay behavior.

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
- `GraphBackendSelectionPolicy`
- `GraphProviderCapabilityMatrix`
- `GraphProviderAdapterContract`
- `GraphProviderErrorMappingProfile`
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
| `identity_split_handoff` | Consume `070.GraphCorrectionHandoff`, validate retained and new canonical partition refs, sort affected fact refs lexically, and route affected facts through the active `GraphProjectionProfile` only. | `GraphCorrectionHandoff` ref/checksum, affected fact refs or affected fact-selection checksum, split partition refs, retained and new canonical refs, projection profile ref, resolver explanation checksum. | `GRAPH_HANDOFF_EFFECT_UNSUPPORTED` when the active projection profile does not support the handoff; missing metadata rejects before delta persistence. |

Every projected delta from correction must include the source `GoldFactChangeSet` ref, graph handoff effect, projection profile ref, authority or absence result refs when applicable, and validation refs.

### DerivedGraphEdgeRule graph handoff

`090` owns the only graph projection path for `130.DerivedGraphEdgeRule.allowed_output_effect = graph_delta_via_090`. `130` owns the rule schema and rule activation. `090` owns graph projection validation, graph delta identity, graph edge semantics, output eligibility, apply profile, backend preflight, and derived-view lag behavior.

| Condition | `090` behavior |
| --- | --- |
| `allowed_output_effect = graph_delta_via_090` and all refs present | Project only through active `GraphProjectionProfile`, `GraphEdgeSemantics`, `GraphObjectOutputEligibilityRow`, `GraphPropertyEvidencePolicy`, `GraphApplyProfile`, and `DerivedViewLagPolicy`. |
| Missing supporting fact refs | Reject before delta persistence with `DERIVED_GRAPH_EDGE_SUPPORTING_FACTS_REQUIRED`. |
| Missing graph projection profile ref | Reject before delta persistence with `DERIVED_GRAPH_EDGE_PROJECTION_FORBIDDEN`. |
| Direct backend write or graph mutation attempt | Reject before backend execution; no provider-native mutation text may execute. |
| Unsupported rule output | Emit explicit no-op when `130.unsupported_behavior = explicit_no_op`. |
| Replay mismatch | Reject production replay before output with `DERIVED_GRAPH_EDGE_REPLAY_MISMATCH` or imported `080` replay error when replay preflight owns the failure. |

This handoff does not add MVP edge types. The MVP active edge set remains exactly `observed_connection`; any additional active edge type requires profile, semantics, output eligibility, query translation, and fixture closure.

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
| `GraphBackendSelectionPolicy` | stable core selection contract plus activation-controlled default rows where applicable | Required when graph serving is enabled and backend profile input is omitted. |
| `GraphProviderCapabilityMatrix` | `activation_controlled_artifact` | Required before provider profile activation. |
| `GraphProviderErrorMappingProfile` | `activation_controlled_artifact` | Required before provider-native errors are exposed as Cadastre errors. |
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

### GraphBackendSelectionPolicy

Graph backend selection is active only when graph serving, graph apply, graph query, graph rebuild, graph drift check, or graph-serving promotion is in implementation scope. Backend selection is not fact authority, identity authority, source-completeness authority, or graph truth.

| Input condition | Required behavior |
| --- | --- |
| graph serving disabled | No backend profile is selected; graph-serving endpoints remain unavailable or validation-only according to `110`. |
| graph serving enabled and `graph_backend.profile_id` omitted | Materialize `graph_backend.profile_id = mvp-janusgraph.v1` and record a defaulting decision ref. |
| graph serving enabled and `graph_backend.profile_id` supplied | Resolve exactly one active `GraphBackendProfile` with that ID. |
| selected profile missing | Fail with `GRAPH_BACKEND_PROFILE_MISSING` before backend mutation or query. |
| selected profile inactive | Fail with `GRAPH_ARTIFACT_INACTIVE` before backend mutation or query. |
| selected profile checksum mismatch | Fail with `GRAPH_ARTIFACT_CHECKSUM_MISMATCH` before backend mutation or query. |
| required profile field omitted | Fail with `GRAPH_BACKEND_CONFIG_INCOMPLETE`; do not inherit provider runtime defaults. |
| selected default profile contains unresolved production fields | Fail with `GRAPH_BACKEND_DEFAULT_UNRESOLVED` before backend mutation, query, rebuild promotion, or drift check. |

### GraphBackendProfile schema

A `GraphBackendProfile` is provider-neutral. Provider-specific fields may appear only inside declared provider subprofiles and must not alter Cadastre graph IDs, output ordering, page-token identity, error categories, evidence refs, or replay semantics.

| Field | Required | Default or omission behavior | Rule |
| --- | ---: | --- | --- |
| `backend_profile_id` | Yes | none | Stable ID. MVP default is `mvp-janusgraph.v1`. |
| `provider` | Yes | none | Closed token. MVP value: `janusgraph`. |
| `provider_version_ref` | Yes | none | Immutable version or package ref; snapshot-only refs fail production activation unless package policy permits validation-only use. |
| `provider_adapter_ref` | Yes | none | Ref to adapter contract and package. |
| `query_language` | Yes | none | MVP JanusGraph value: `gremlin`. |
| `driver_ref` | Yes | none | Driver artifact, version, and package ref. |
| `storage_backend_kind` | Yes | none | Provider-specific storage backend token mapped by this profile. |
| `storage_backend_version_ref` | Required for external storage | none | Required for CQL, Scylla, HBase, Bigtable, and other external storage. |
| `index_backend_kind` | Yes | none | `none` allowed only when query translation rows declare affected query classes unsupported. |
| `index_backend_version_ref` | Required when index backend is not `none` | none | Required for Elasticsearch, Solr, Lucene, or future index backends. |
| `schema_policy_ref` | Yes | none | Active `GraphReadModelSchemaProfile`. |
| `taxonomy_mapping_ref` | Yes | none | Active `GraphBackendTaxonomyMappingProfile`. |
| `query_translation_profile_ref` | Yes | none | Active `GraphQueryTranslationProfile`. |
| `apply_profile_ref` | Yes | none | Active `GraphApplyProfile`. |
| `capability_matrix_ref` | Yes | none | Active `GraphProviderCapabilityMatrix`. |
| `raw_write_bypass_policy_ref` | Yes | none | Must prove raw Gremlin/admin/import writes are blocked, detected, or excluded from production. |
| `package_set_manifest_ref` | Yes | none | Active `100.ProductionPackageSetManifest`. |
| `validation_refs` | Yes | none | Non-empty refs to `120` backend-profile validation rows. |
| `activation_scope` | Yes | none | Graph serving scope. |
| `lifecycle_status` | Yes | none | Production use requires `active`. |

### MVP JanusGraph default backend profile

The default profile row is a selection default, not a domain decision. It must not become production-active while any `TODO:` value in this table is unresolved.

| Field | Required MVP value |
| --- | --- |
| `backend_profile_id` | `mvp-janusgraph.v1` |
| `provider` | `janusgraph` |
| `query_language` | `gremlin` |
| `provider_adapter_ref` | `janusgraph.gremlin_remote.v1` unless a later active row selects embedded mode |
| `server_mode` | `janusgraph_server_gremlin_remote` |
| `gRPC` | disabled unless a separate active profile row enables it |
| `implicit_schema_creation` | forbidden in production |
| `backend_internal_ids` | forbidden as Cadastre IDs, selectors, cursors, evidence refs, replay keys, drillback keys, response IDs, or page-token identity |
| `storage_backend_kind` | TODO: product governance must select a durable storage backend or require explicit deployment configuration before active use |
| `index_backend_kind` | TODO: product governance must select an index backend or declare affected query classes unsupported before active use |

### GraphProviderCapabilityMatrix

Each active provider profile must satisfy this matrix for every enabled graph-serving scope. Unsupported provider behavior must fail closed or mark the affected query/apply/rebuild class unsupported before backend execution.

| Capability | Required for MVP graph serving | Default when unsupported |
| --- | --- | --- |
| directed property graph | yes | profile activation fails |
| user-supplied Cadastre graph IDs as properties or keys | yes | profile activation fails |
| backend internal ID non-exposure | yes | `GRAPH_BACKEND_ID_FORBIDDEN` |
| explicit schema creation and validation | yes | `GRAPH_SCHEMA_IMPLICIT_CREATION_FORBIDDEN` or `GRAPH_SCHEMA_FINGERPRINT_STALE` |
| deterministic query candidate materialization | yes | query class unsupported |
| index readiness/freshness evidence | required for indexed query classes | affected query classes blocked |
| transaction evidence | yes for graph apply | apply blocked |
| partial-apply resume evidence | yes | `GRAPH_APPLY_RESUME_UNSAFE` |
| rebuild from Cadastre deltas | yes | promotion blocked |
| raw-write bypass control | yes | `GRAPH_RAW_WRITE_BYPASS` |

### JanusGraph mapping tables

The following mappings are exhaustive for the active MVP graph scope. Any Cadastre graph object not listed here is unsupported for `mvp-janusgraph.v1` until a later active mapping row adds it.

| Cadastre concept | JanusGraph concept | Required mapping | Forbidden mapping |
| --- | --- | --- | --- |
| `projected_canonical_node` | vertex | vertex label `cadastre_projected_canonical_node`; Cadastre `graph_node_id` stored as property `cadastre_graph_node_id` | JanusGraph vertex ID as Cadastre ID |
| `observed_connection` | edge | edge label `cadastre_observed_connection`; Cadastre `graph_edge_id` stored as property `cadastre_graph_edge_id` | JanusGraph edge ID as Cadastre ID |
| graph object evidence refs | vertex or edge properties | canonical array property `cadastre_evidence_refs` after redaction and bounds checks | raw payload bytes or backend-native evidence IDs |
| assertion and temporal fields | vertex or edge properties | Cadastre-owned assertion state, valid time, known time, confidence, and source refs | JanusGraph transaction time, index time, or storage time as fact time |

| Cadastre node or edge type | JanusGraph label | Active MVP status |
| --- | --- | --- |
| `projected_canonical_node` | vertex label `cadastre_projected_canonical_node` | active when projection row emits a resolved canonical node |
| `observed_connection` | edge label `cadastre_observed_connection` | active when projection row emits a qualifying flow edge |
| all other node types | none | unsupported; fail with `GRAPH_EDGE_SEMANTICS_ROW_MISSING` or mapped owner error before output |
| all other edge types | none | unsupported; fail with `GRAPH_EDGE_SEMANTICS_ROW_MISSING` before projection/query |

| Cadastre property class | JanusGraph property key | Cardinality or multiplicity rule |
| --- | --- | --- |
| stable graph IDs | `cadastre_graph_node_id`, `cadastre_graph_edge_id` | single cardinality; unique by Cadastre preflight and validation rows |
| scalar graph fields | mapped lower snake-case property keys with `cadastre_` prefix | single cardinality unless the property policy declares array serialization |
| evidence ref arrays | `cadastre_evidence_refs` | single property containing canonical array bytes, not repeated provider values |
| edge uniqueness | `cadastre_graph_edge_id` plus endpoint IDs and edge type | simple JanusGraph edge multiplicity is not authority; Cadastre idempotency key decides reapply behavior |

| Query or index purpose | JanusGraph index family | Required behavior |
| --- | --- | --- |
| Cadastre graph ID lookup | composite index or provider-equivalent exact lookup | required for `node_detail` and evidence drillback |
| endpoint adjacency | vertex-centric index when high-degree adjacency fixtures require it | required before affected neighbor/path classes serve |
| text, range, or mixed predicates | mixed index | required only when query translation row enables those predicates |
| no declared index | none | affected query class unsupported; full scan is forbidden in production |

| JanusGraph configuration surface | Cadastre profile field | Required behavior |
| --- | --- | --- |
| storage backend | `storage_backend_kind`, `storage_backend_version_ref`, storage config checksum | required before active use |
| mixed index backend | `index_backend_kind`, `index_backend_version_ref`, index config checksum | required when indexed query classes are enabled |
| schema initialization | `schema_policy_ref`, schema-init config checksum | implicit schema and destructive drop forbidden in production |
| server mode | `provider_adapter_ref`, `server_mode`, driver refs | default is Gremlin remote server mode |
| TinkerPop/Gremlin versions | `driver_ref`, `provider_version_ref`, `tinkerpop_version_ref` | included in `VersionManifest` |

### JanusGraph schema strategy and operational assumptions

A production JanusGraph profile must use explicit schema. It must fail when implicit schema creation is enabled, when required schema constraints are disabled without an explicit provider exception row and validation fixture, or when schema/index readiness cannot be proven.

`schema.init.drop-before-startup` is forbidden in production apply, query, rebuild promotion, and normal startup. `schema.init.force-close-other-instances` is forbidden unless a controlled rebuild or maintenance profile permits it.

`BackendSchemaFingerprint` for JanusGraph must include vertex labels, edge labels, property keys, cardinality, multiplicity, composite indexes, mixed indexes, vertex-centric indexes, schema-init strategy, provider version, TinkerPop version, storage backend config checksum, index backend config checksum, and provider capability rows.

| Assumption | Required spec behavior |
| --- | --- |
| server mode | default adapter uses JanusGraph Server over Gremlin remote |
| embedded JVM mode | allowed only through a separate active profile row |
| gRPC management | disabled by default; enabling requires separate profile, package, API, and validation rows |
| in-memory storage | validation-only unless a future active profile proves durable snapshot/restore semantics |
| destructive drop | forbidden in production apply and query paths |
| Bigtable | unsupported for MVP until unresolved adapter behavior closes |
| Docker/container entrypoint | unsupported as a production default until entrypoint behavior is inspected and package-gated |

### MVPActiveGraphProjectionProfile

The only active MVP graph projection profile is `mvp-observed-flow-graph.v1`. A production graph projection profile with any other profile ID must remain inactive unless a later authoritative spec-set version adds its profile row, semantics rows, output eligibility rows, query translation rows, validation rows, and promotion refs.

| Profile property | Required MVP value |
| --- | --- |
| `profile_id` | `mvp-observed-flow-graph.v1` |
| Active edge type set | Exactly `observed_connection`. |
| Active node output set | Exactly `projected_canonical_node`. |
| Source fact family | Authorized `GoldFact` records with fact type `observed_network_flow_fact` and qualifying flow-role evidence. |
| Endpoint identity | Both endpoints must resolve to `070.CanonicalEntity` refs through qualifying `070.IdentityDecision` output. |
| Direction source | `FlowRoleEvidence` consumed under `mvp-flow-role-directed-v1`; OCSF endpoint order and backend edge convention are forbidden direction inputs. |
| Structural globals | None emitted. `StructuralGlobalNode` rows are inactive for MVP. |
| Generic external graph payload | Non-emitting by default and never pathfinding-eligible. |
| Theoretical reachability | `has_theoretical_reachability`, modeled reachability facts, boolean reachability properties, and unqualified reachability wording are prohibited. |
| Backend identity | Backend-native IDs must not appear in graph IDs, selectors, evidence refs, page tokens, query cursors, or drillback keys. |
| Unsupported or missing input | Emit deterministic no-op or the most specific graph error; do not infer nodes, edges, traversal, expiry, cleanup, or reachability. |

### GraphProjectionProfileRow schema

A concrete graph projection profile row may affect output only through `030.ActivationControlledArtifactRef` with `artifact_class = graph_projection_profile`. Each row must be validated before any graph delta is persisted.

| Field | Required | Default or omission behavior | Rule |
| --- | ---: | --- | --- |
| `profile_id` | Yes | none | Stable profile ID. MVP value is `mvp-observed-flow-graph.v1`. |
| `input_fact_type` | Yes | none | Exact `GoldFact` fact type. Wildcards are forbidden. |
| `source_predicate` | Yes | none | Exact predicate or fact-family discriminator consumed by the row. |
| `allowed_assertion_states` | Yes | none | Closed set imported from `040` and `080`; omission rejects projection. |
| `temporal_policy_ref` | Yes | none | Exact `080` temporal or correction policy ref that governs valid/known intervals. |
| `authority_ref` | Yes | none | Exact `060.SourceAuthorityProfileRow` ref or authorized positive-fact source ref. |
| `endpoint_identity_requirement` | Yes | `resolved_canonical_entity_required` | MVP requires resolved canonical endpoint identity for both endpoints. |
| `node_projection_refs` | Yes | none | Refs to node projection rows; MVP allows only `projected_canonical_node`. |
| `edge_projection_ref` | Yes for edge-emitting rows | none | Must name one active edge projection row and its `GraphEdgeSemanticsRegistry` row. |
| `property_mapping_refs` | Yes | empty only when the row emits no object | Refs to `MVPGraphPropertyMapping` rows. |
| `edge_semantics_row_ref` | Required for edges | none | Missing row emits `GRAPH_EDGE_SEMANTICS_ROW_MISSING`. |
| `output_eligibility_refs` | Yes | none | Exact refs to closed `GraphObjectOutputEligibilityRow` rows. |
| `backend_taxonomy_mapping_ref` | Yes | none | Active `GraphBackendTaxonomyMappingProfile` ref. |
| `query_translation_profile_ref` | Yes | none | Active `GraphQueryTranslationProfile` ref for graph-serving output. |
| `no_op_rules` | Yes | none | Must cover missing flow role, ambiguous flow role, unresolved endpoint, unsupported source fact, prohibited reachability, and generic external payload cases. |
| `validation_refs` | Yes | none | Non-empty refs to passing `120.GraphActiveProfileClosureValidationMatrix` rows. |
| `activation_scope` | Yes | none | Scope in which the row may affect graph output. |
| `lifecycle_status` | Yes | none | Production use requires `active`. |

### MVPGraphNodeProjectionRows

| Node projection row | Input | Required identity | Emitted object | Default when unavailable | Validation rows |
| --- | --- | --- | --- | --- | --- |
| `mvp-project-canonical-node-v1` | Resolved endpoint canonical entity ref from `070` output | Qualifying `IdentityDecision` and `CanonicalEntity` ref | `projected_canonical_node` | Emit no node and block dependent edge with `GRAPH_ENDPOINT_IDENTITY_UNRESOLVED`. | `val-090-unresolved-endpoint-no-edge`, `val-070-graph-endpoint-requires-identity-decision` |
| `mvp-structural-global-node-disabled-v1` | Any structural global request | n/a | none | Deterministic no-op; structural global output remains inactive. | `val-090-mvp-active-profile-exact-edge-set` |
| `mvp-generic-external-payload-disabled-v1` | Source-native graph payload or graph key | n/a | none | Deterministic no-op; payload may be retained only outside pathfinding. | `val-090-generic-external-payload-not-pathfinding` |

### MVPGraphEdgeProjectionRows

| Edge projection row | Required inputs | Direction rule | Edge ID inputs | Confidence | Temporal fields | Evidence refs | Traversal class | Non-implication | No-op or error behavior |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `mvp-observed-connection-edge-v1` | `GoldFact` fact type `observed_network_flow_fact`; two resolved endpoint canonical entity refs; qualifying `FlowRoleEvidence`; source `GoldFact` ref | `mvp-flow-role-directed-v1`; source and target endpoint are selected only from qualifying flow-role evidence | projection profile ID, source gold fact key/ref, from canonical entity ref, to canonical entity ref, direction rule ref, valid interval, known interval, edge type | Import from source fact and serialize through `040.DecimalPrecisionPolicy.confidence_0_1`. | Import from source fact or correction handoff under `080` policies. | Source `GoldFact`, silver observation, raw metadata, temporal resolution, authority refs, and flow-role evidence refs. | `observed_connection_path` | Observed traffic only; not theoretical reachability, service access, policy allow, compromise, exploitability, lateral movement, or identity access. | Missing flow-role evidence emits `GRAPH_FLOW_ROLE_EVIDENCE_REQUIRED` or no-op; ambiguous flow-role evidence emits no edge; unresolved endpoint emits `GRAPH_ENDPOINT_IDENTITY_UNRESOLVED`; OCSF endpoint order emits no edge. |

### MVPGraphPropertyMapping

Only the properties in this table may be emitted by the active MVP graph profile. Unknown graph properties fail before graph delta persistence unless a later active graph property policy row declares them.

| Object class | Property | Required source | Serialization | Default or omission behavior |
| --- | --- | --- | --- | --- |
| `projected_canonical_node` | `canonical_entity_id` | `070.CanonicalEntity` ref | Cadastre ID string | Required. Omission blocks node projection. |
| `projected_canonical_node` | `entity_type` | `070.CanonicalEntity` type | enum token | Required. Omission blocks node projection. |
| `projected_canonical_node` | `display_label` | Redaction-approved label source | string | Optional; omitted when redaction blocks label. |
| `observed_connection` | `source_gold_fact_id` | Source `GoldFact` ref | Cadastre ID string | Required. Omission blocks edge projection. |
| `observed_connection` | `assertion_state` | Source `GoldFact.assertion_state` | enum token | Required. |
| `observed_connection` | `confidence` | Source fact confidence | canonical six-fraction decimal string | Required when source confidence exists; null only when source fact has no confidence policy. |
| `observed_connection` | `valid_from`, `valid_to`, `known_from`, `known_to` | Source fact temporal fields | RFC3339 UTC or null where owner permits | Required. |
| `observed_connection` | `last_seen` | Source fact valid interval or observed time selected by `080` | RFC3339 UTC or null | Required for path ordering when present; null sorts last. |
| `observed_connection` | `evidence_ref_ids` | Evidence refs | canonically sorted array | Required non-empty. |

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
7. If handoff effect is `identity_split_handoff`, validate `070.GraphCorrectionHandoff` ref/checksum, split partition refs, retained and new canonical refs, resolver explanation checksum, affected fact refs or selection checksum, and projection profile ref; sort affected fact refs lexically; expire or update stale projected graph objects only through active projection rows and source-authorized correction/handoff refs; emit no delta when the projection profile declares the handoff unsupported.
8. Reject unsupported or metadata-incomplete handoff effects with `GRAPH_HANDOFF_EFFECT_UNSUPPORTED` or the more specific manifest/lineage code before delta persistence.
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

### GraphApplyBackendEvidenceRow JanusGraph fields

`ApplyGraphDelta` must persist provider evidence before derived-view advancement when the selected backend profile uses JanusGraph. JanusGraph commit evidence is not a single cross-system distributed transaction proof; it must expose the storage, composite index, mixed index, log, rollback, and read-after-write components that can affect correctness.

| Evidence field | Required for JanusGraph MVP |
| --- | --- |
| `janusgraph_transaction_id_or_ref` | required when available; redacted if backend-specific |
| `storage_commit_status` | required |
| `composite_index_commit_status` | required |
| `mixed_index_commit_status` | required when mixed indexes are configured |
| `transaction_log_status` | required when logs are configured or recovery depends on logs |
| `rollback_attempt_status` | required on failure |
| `read_after_write_probe_ref` | required before derived-view advancement |
| `committed_batch_ids` | required for partial or resumed apply |
| `resume_boundary` | required when partial apply occurs |
| `index_freshness_ref` | required for affected query classes |

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

### GraphQueryResponseHandoffTo110

`090` owns graph execution, graph result ordering, graph candidate limits, derived-view state, graph object visibility eligibility, and backend cursor rejection. `110` owns public API rendering. `090.QueryGraph` must emit an owner-level result that matches one row below so `110` can render without re-running graph logic or inferring backend semantics.

| query_class | Required graph payload shape | Empty-result semantics | Partial-result permission | Derived-view stale behavior | Source-stale display eligibility | Conflicted graph-object visibility | Ambiguous graph-object behavior | Candidate-limit behavior | Required `110` error mapping |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `node_detail` | At most one graph node detail with graph profile context and evidence refs. | No visible node returns empty detail only when authorization permits existence disclosure; otherwise `AUTHORIZATION_ERROR`. | Forbidden. | Reject when current graph state is required; otherwise label `derived_view_stale`. | Allowed only when output eligibility row permits stale detail display. | Display only when eligibility row permits conflict detail. | More than one candidate for one Cadastre ID emits `GRAPH_QUERY_TRANSLATION_ERROR`; no mutation. | Candidate limit fixed at 1. | `110.GraphQueryResponse` with `error` or empty payload. |
| `neighbor_expansion` | Center node plus ordered neighbor nodes and edges. | Empty neighbor set is success with empty arrays. | Allowed only when translation profile marks partial expansion safe. | Label or reject by `DerivedViewLagPolicy`. | Eligible stale neighbors may display with `source_stale`. | Eligible conflicted edges may display as `conflicted`; no pathfinding implication. | Ambiguous endpoint or edge state emits `ambiguous` or owner error; no graph mutation. | Emit `GRAPH_QUERY_CANDIDATE_LIMIT_REACHED`; no partial unless allowed. | `110.EndpointOutcomeMatrix.GraphQuery`. |
| `bounded_path` | Ordered paths with path node ID sequence, edge refs, and traversal classes. | Empty traversal class returns no paths without backend traversal; no matching path returns empty paths. | Forbidden for compliance and audit; otherwise only when profile permits. | Reject stale derived view by default. | Stale edges path-visible only when eligibility permits. | Conflicted path objects excluded unless eligibility permits conflict-visible path detail. | Ambiguous traversal class, endpoint, or object state rejects or returns diagnostic without mutation. | Emit `GRAPH_QUERY_CANDIDATE_LIMIT_REACHED`; do not leak partial paths unless permitted. | `GRAPH_TRAVERSAL_CLASS_REQUIRED`, `DERIVED_VIEW_LAG_ERROR`, or candidate-limit row. |
| `evidence_drillback` | Graph object refs mapped to gold/silver/raw evidence refs, no raw bytes. | Empty chain only when graph object is visible and has no evidence refs; missing required lineage maps to `LINEAGE_ERROR`. | Allowed for paged evidence metadata. | Preserve graph state ref separately from source state. | Display source-stale evidence metadata when permitted. | Preserve conflict context in evidence metadata. | Ambiguous evidence mapping emits `ambiguous` and no raw expansion. | Candidate limit blocks raw expansion. | `110.EvidenceDrillbackResponse`. |
| `analysis_read_only` | Read-only graph result set and compatibility refs for `130`. | Empty analysis result is success. | Only when `130.RuleGraphCompatibilityMatrix` permits partial read-only output. | Reject or label according to `130` handoff and `DerivedViewLagPolicy`. | Preserved as analysis input state, not fact authority. | Preserved as non-authoritative analysis context. | Ambiguous graph result remains non-mutating and must not be promoted to finding authority. | Candidate limit emits owner error and no mutation. | `110.AnalysisReadOnly` endpoint outcome row. |

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

The registry is total for the declared MVP edge scope. `all_other_edge_types` is a closed reject row, not an extension point.

| Edge type selector | Source fact type | Predicate | Direction rule | Evidence | Assertion states | Temporal policy | Confidence policy | Traversal class | Non-implication | No-op or error behavior | Validation rows | Status |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `observed_connection` | `observed_network_flow_fact` | observed flow relation | `FlowRoleEvidence` only; OCSF endpoint order and backend edge convention are not qualifying evidence | gold, silver, raw, temporal, authority, and flow-role refs | `active`; `stale` only when `060` and eligibility rows permit stale display | `080` | Import from source fact and serialize through `040.DecimalPrecisionPolicy.confidence_0_1` | `observed_connection_path` | Observed traffic only; no theoretical reachability, service access, policy allow, compromise, exploitability, lateral movement, or identity-access implication. | No edge when `FlowRoleEvidence` is missing or ambiguous; unresolved endpoint blocks with `GRAPH_ENDPOINT_IDENTITY_UNRESOLVED`; OCSF endpoint order emits no direction. | `val-090-observed-connection-positive`, `val-090-observed-connection-missing-flow-role`, `val-090-observed-connection-ambiguous-flow-role`, `val-090-ocsf-endpoint-order-no-direction` | active_mvp |
| `has_theoretical_reachability` | none in MVP | none | none | none | none | none | none | none | prohibited | Emit `THEORETICAL_REACHABILITY_SCOPE_ERROR`, `REACHABILITY_DEFERRED_OUTPUT_FORBIDDEN`, or deterministic no-op; emit no fact, edge, property, API claim, or traversal class. | `val-090-theoretical-reachability-prohibited`, `val-090-boolean-reachability-property-prohibited` | inactive_deferred |
| `all_other_edge_types` | none | none | none | none | none | none | none | none | prohibited unless later active row set promotes the edge type | Reject activation or projection with `GRAPH_EDGE_SEMANTICS_ROW_MISSING`; emit no delta. | `val-090-mvp-active-profile-exact-edge-set` | inactive_unmapped |

The MVP active graph edge set is exactly `observed_connection`. Any additional active edge type requires a later authoritative profile row, semantics row, output eligibility row, query translation row, and validation fixture set before projection activation.

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

### GraphDeltaApplyCanonicalOrder

`ApplyGraphDelta` must sort graph deltas before batching. The sort key is the tuple below; every value is compared lexically after canonical serialization except `class_rank`, which is compared numerically.

| Class rank | Delta class | Applies to | Tie-break keys |
| ---: | --- | --- | --- |
| 10 | edge `expire` | Existing edges | `edge_type`, `from_graph_node_id`, `to_graph_node_id`, `graph_edge_id`, `graph_edge_delta_id` |
| 20 | edge `cleanup` | Existing edges | `edge_type`, `from_graph_node_id`, `to_graph_node_id`, `graph_edge_id`, `graph_edge_delta_id` |
| 30 | node `expire` | Existing nodes | `node_type`, `graph_node_id`, `graph_node_delta_id` |
| 40 | node `cleanup` | Existing nodes | `node_type`, `graph_node_id`, `graph_node_delta_id` |
| 50 | node `upsert` | New or refreshed nodes | `node_type`, `graph_node_id`, `graph_node_delta_id` |
| 60 | edge `upsert` | New or refreshed edges | `edge_type`, `from_graph_node_id`, `to_graph_node_id`, `graph_edge_id`, `graph_edge_delta_id` |
| 70 | node `update_visibility` | Node visibility only | `node_type`, `graph_node_id`, `graph_node_delta_id` |
| 80 | edge `update_visibility` | Edge visibility only | `edge_type`, `from_graph_node_id`, `to_graph_node_id`, `graph_edge_id`, `graph_edge_delta_id` |
| 90 | `no_op` | No backend mutation | `graph_node_delta_id` or `graph_edge_delta_id`; no backend call is made. |

A delta operation not listed in this table fails with `CORE_FIELD_TYPE_INVALID` before apply. Same input delta set, same apply profile, and same backend schema fingerprint must produce byte-identical ordered batch membership.

### GraphQueryTranslationProfile observable contract

| Field | Required behavior |
| --- | --- |
| stable page token fields | Query checksum, authorization context checksum, ordering keys, page boundary, derived-view state, page-token expiry, and profile refs. |
| backend candidate limit | Resolved from `GraphQueryCandidateLimitTable` by query class before backend execution. |
| timeout behavior | Return `GRAPH_QUERY_TIMEOUT` unless partial results are explicitly permitted for the query class. |
| path traversal-class omission | A bounded path query with omitted `allowed_traversal_classes` fails with `GRAPH_TRAVERSAL_CLASS_REQUIRED` before backend execution. |
| empty traversal behavior | Empty `allowed_traversal_classes` returns no paths and emits no backend path query. |
| authorization and redaction | Applied after backend candidate materialization and before response emission. |
| output eligibility | Every returned object must pass `GraphObjectOutputEligibilityRow` for the response context. |

### JanusGraph Gremlin query translation matrix

JanusGraph Gremlin translation is allowed only as backend candidate materialization. Cadastre ordering, authorization, redaction, page-token identity, and error selection occur after candidate materialization.

| Query class | Gremlin translation boundary | Required post-processing | Unsupported or unsafe provider behavior |
| --- | --- | --- | --- |
| `node_detail` | bounded lookup by Cadastre graph ID property | enforce single candidate, authorization, redaction, deterministic output | more than one candidate emits `GRAPH_QUERY_TRANSLATION_ERROR` |
| `neighbor_expansion` | bounded adjacent edge traversal using mapped labels/properties | sort by Cadastre ordering; enforce candidate limit | candidate overflow emits `GRAPH_QUERY_CANDIDATE_LIMIT_REACHED` |
| `bounded_path` | bounded repeat traversal over named traversal classes only | sort paths by `090` path ordering | omitted traversal classes emit `GRAPH_TRAVERSAL_CLASS_REQUIRED`; full graph traversal forbidden |
| `evidence_drillback` | lookup by Cadastre graph object refs only | no raw bytes; preserve evidence chain | backend IDs rejected |
| `analysis_read_only` | only via active `130.RuleGraphCompatibilityMatrix` | no mutation; expected checksum | provider-specific Gremlin text forbidden unless imported and validated |

Production JanusGraph profiles must block any graph-centric query translation that would perform an unbounded full scan. Backend natural order must never become response order. When JanusGraph `query.force-index` behavior would allow a scan or throw because no index is available, Cadastre must emit `GRAPH_QUERY_FULL_SCAN_FORBIDDEN` or block the affected query class before backend traversal.

#### GraphQueryCandidateLimitTable

| Query class | Default candidate limit | Minimum | Maximum | Limit reached behavior |
| --- | ---: | ---: | ---: | --- |
| `node_detail` | 1 | 1 | 1 | More than one backend candidate for one Cadastre ID fails with `GRAPH_QUERY_TRANSLATION_ERROR`. |
| `neighbor_expansion` | 5000 | 1 | 20000 | Emit `GRAPH_QUERY_CANDIDATE_LIMIT_REACHED`; return no partial result unless the query class explicitly permits partial output. |
| `bounded_path` | 10000 | 1 | 50000 | Emit `GRAPH_QUERY_CANDIDATE_LIMIT_REACHED`; compliance and audit path queries reject rather than return partial paths. |
| `evidence_drillback` | 1000 | 1 | 5000 | Emit `GRAPH_QUERY_CANDIDATE_LIMIT_REACHED` and return no raw payload expansion. |
| `analysis_read_only` | 10000 | 1 | 50000 | Emit `GRAPH_QUERY_CANDIDATE_LIMIT_REACHED`; `130.RuleGraphCompatibilityMatrix` must name the expected result checksum. |

#### GraphQueryPageTokenPolicy

| Token rule | Required behavior |
| --- | --- |
| `110` handoff | TTL request-field behavior, default `900`, bounds `60..3600`, expired-token non-refresh, authorization mismatch, request checksum mismatch, redaction mismatch, and portability rules are owned by `110.PageTokenPolicy`. |
| Identity inputs owned by `090` | Query checksum, authorization context checksum, derived-view state, ordering keys, page boundary, expiry, graph projection profile ref, query translation profile ref, backend taxonomy mapping profile ref, output eligibility row-set checksum, redaction context checksum, backend schema fingerprint ref, and graph query class. |
| Expired token | Reject with `GRAPH_PAGE_TOKEN_EXPIRED` before backend query execution and before owner result materialization. |
| Mismatched token | Reject with `GRAPH_PAGE_TOKEN_INVALID` before backend query execution when any graph-owned identity input mismatches the current request context. |
| Backend cursor | Reject with `GRAPH_PAGE_TOKEN_INVALID`; backend cursors and backend-native IDs are forbidden token identity. |
| Derived-view mismatch | Reject before backend query execution; use `DERIVED_VIEW_LAG_ERROR` only when current derived-view state is required by query class. |
| Portability | Graph page tokens are not portable across callers, roles, graph profiles, query translation profiles, backend taxonomy mappings, output eligibility row sets, redaction contexts, derived-view states, or backend schema fingerprints. |

### GraphRebuildManifest schema

| Field | Required | Rule |
| --- | ---: | --- |
| `input_snapshot_refs` | Yes | Authoritative lakehouse snapshot or dataset refs. |
| `graph_delta_set_refs` | Yes | Persisted graph delta set refs and checksums. |
| `projection_profile_ref` | Yes | Active graph projection profile. |
| `projection_row_set_checksum` | Yes | SHA-256 over active node, edge, and property projection rows. |
| `edge_semantics_row_set_checksum` | Yes | SHA-256 over active `GraphEdgeSemanticsRegistry` rows. |
| `output_eligibility_row_set_checksum` | Yes | SHA-256 over active `GraphObjectOutputEligibilityRow` rows. |
| `backend_taxonomy_mapping_checksum` | Yes | SHA-256 over active `GraphBackendTaxonomyMappingProfile`. |
| `query_translation_checksum` | Yes | SHA-256 over active `GraphQueryTranslationProfile`. |
| `graph_property_policy_checksum` | Yes | SHA-256 over active `GraphPropertyEvidencePolicy`. |
| `backend_profile_ref` | Yes | Active backend profile. |
| `schema_fingerprint` | Yes | Current backend schema fingerprint. |
| `index_consistency_check_ref` | Yes | Passing `GraphIndexConsistencyCheck` ref for the rebuilt read model. |
| `canonical_output_checksum_inputs` | Yes | Canonical ordered list of included output classes and excluded volatile fields. |
| `output_checksum` | Yes | SHA-256 over canonical rebuilt graph-serving output summary. |
| `status` | Yes | success, failed, blocked, or not_run. |
| `errors` | Yes | Default `[]`; owner-specific graph rebuild errors. |

### GraphObjectOutputEligibilityRow

The active MVP output eligibility rows are closed. Graph object existence alone never grants search, neighbor expansion, pathfinding, finding, metric, or identity influence eligibility.

| Object class | Search | Neighbor expansion | Pathfinding | Analysis finding | Metrics | Identity influence | Default emission |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `projected_canonical_node` | allowed when authorization permits | allowed when query target includes the node type | path endpoint only when path query names an allowed traversal class | read-only when `130.RuleGraphCompatibilityMatrix` permits | allowed only when the active metric rule names this object class | none; must not create or merge identity | emitted only from resolved `CanonicalEntity` refs. |
| `observed_connection` | detail and filter only | allowed when edge filter permits and traversal class is requested where needed | allowed only for `observed_connection_path` and only when explicitly requested | read-only when `130.RuleGraphCompatibilityMatrix` names `observed_connection_path` or an allowed detail query | allowed only when the metric rule names `observed_connection` | none | emitted only by `mvp-observed-connection-edge-v1`. |
| `synthetic_structural_object` | disabled | disabled | disabled | disabled | disabled | none | non-emitting in MVP. |
| `generic_external_graph_payload` | disabled | disabled | disabled | disabled | disabled | none | non-emitting in MVP; may be preserved outside graph serving only when another owner permits. |

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

### JanusGraph GraphIndexConsistencyCheck rows

| Check | Required behavior |
| --- | --- |
| composite index status | must be query-ready before affected query classes serve |
| mixed index provider availability | must pass before mixed-index query classes serve |
| mixed index freshness | stale index blocks affected query classes |
| vertex-centric index presence | required for high-degree adjacency query classes that depend on it |
| schema fingerprint match | stale or mismatched fingerprint blocks mutation/query |
| full-scan prohibition | unbounded graph scan is rejected unless a validation-only row permits it |

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
| `GRAPH_EDGE_SEMANTICS_ROW_MISSING` | An active profile attempts to emit or query an edge type with no exact active semantics row. |
| `GRAPH_ENDPOINT_IDENTITY_UNRESOLVED` | A graph endpoint lacks a qualifying `070.IdentityDecision` and resolved `CanonicalEntity` ref. |
| `GRAPH_QUERY_CANDIDATE_LIMIT_REACHED` | Backend candidate materialization reaches the active query-class candidate limit. |
| `GRAPH_PAGE_TOKEN_EXPIRED` | A graph page token is past its declared expiry. |
| `GRAPH_PAGE_TOKEN_INVALID` | A graph page token identity input mismatches the request context or contains backend cursor identity. |
| `GRAPH_TRAVERSAL_CLASS_REQUIRED` | A bounded path query omits `allowed_traversal_classes`. |
| `GRAPH_QUERY_TIMEOUT` | Backend candidate materialization or owner post-processing exceeds the active timeout and partial output is not permitted. |
| `GRAPH_QUERY_TRANSLATION_ERROR` | Query translation produces zero required mappings, multiple candidates for a single-object query, unsafe backend plan, or non-deterministic materialization. |
| `GRAPH_DELTA_IDEMPOTENCY_CONFLICT` | A repeated graph apply uses the same idempotency key with different output-affecting inputs. |
| `GRAPH_BACKEND_PROFILE_MISSING` | Graph serving is in scope and no selected backend profile resolves. |
| `GRAPH_SCHEMA_FINGERPRINT_STALE` | Backend schema fingerprint is absent, stale, mismatched, or incomplete. |
| `GRAPH_RAW_WRITE_BYPASS` | Provider console, admin, import, or raw query path can mutate graph state outside `ApplyGraphDelta`, or bypass evidence is missing. |
| `GRAPH_DRIFT_REPAIR_FORBIDDEN` | A drift check attempts repair, mutation, graph delta emission, authoritative record mutation, or watermark advancement. |
| `GRAPH_APPLY_RESUME_UNSAFE` | Partial graph apply cannot prove committed batch boundary and resume safety. |
| `GRAPH_REBUILD_EQUIVALENCE_FAILED` | Graph rebuild output, schema, index, delta input, or derived-view state does not match the declared rebuild equivalence policy. |
| `THEORETICAL_REACHABILITY_SCOPE_ERROR` | MVP graph behavior attempts theoretical reachability output. |
| `GRAPH_BACKEND_CONFIG_INCOMPLETE` | A required `GraphBackendProfile` field is omitted or empty. |
| `GRAPH_BACKEND_DEFAULT_UNRESOLVED` | Omitted backend selection materializes the JanusGraph default but required default fields remain `TODO` or unresolved. |
| `GRAPH_PROVIDER_CAPABILITY_MISSING` | The selected profile lacks an active provider capability matrix. |
| `GRAPH_PROVIDER_CAPABILITY_UNSUPPORTED` | The provider capability matrix declares an enabled query, apply, rebuild, or serving capability unsupported. |
| `GRAPH_BACKEND_VERSION_UNPINNED` | Provider, driver, storage adapter, index adapter, or TinkerPop version ref is mutable, missing, or snapshot-only in production scope. |
| `GRAPH_BACKEND_PACKAGE_GATE_FAILED` | A required provider, driver, storage adapter, index adapter, runtime distribution, package release, or package-set gate from `100` failed. |
| `GRAPH_PROVIDER_ADAPTER_UNSUPPORTED` | The selected provider adapter cannot satisfy the declared provider profile, server mode, query language, or driver contract. |
| `GRAPH_SCHEMA_IMPLICIT_CREATION_FORBIDDEN` | Production provider config enables implicit schema creation or omits explicit schema proof. |
| `GRAPH_STORAGE_MODE_UNSAFE` | Storage mode is ephemeral, destructive, validation-only, or lacks durability evidence for production serving. |
| `GRAPH_INDEX_BACKEND_UNAVAILABLE` | Required index provider or mixed-index integration is unavailable for affected query classes. |
| `GRAPH_INDEX_FRESHNESS_REQUIRED` | Index freshness evidence is missing or stale for affected query classes. |
| `GRAPH_QUERY_FULL_SCAN_FORBIDDEN` | Query translation would require unbounded full graph scan or backend natural-order traversal in production. |

### GraphErrorRegistryFragment

This owner fragment feeds `110.GenerateErrorCodeRegistry`. `110` owns the generated caller-visible registry. This table must not render API output by itself. No row may contain `TODO:` when graph serving, graph apply, graph query, graph rebuild, or graph backend activation is in promotion scope.

| error_code | owner_spec | default_severity | default_retry_class | caller_visible_fields | audit_visible_fields | redaction_rule | owner_context_schema_ref | fixture_family |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `GRAPH_ARTIFACT_MISSING` | `090` | blocked | policy_change_required | error_code, owner_spec, artifact_class, artifact_id | artifact_ref, manifest_ref, validation_refs | redact private refs; no raw payloads | `090.GraphErrorContext` | `graph-error-graph-artifact-missing` |
| `GRAPH_ARTIFACT_INACTIVE` | `090` | blocked | policy_change_required | error_code, owner_spec, artifact_class, lifecycle_status | artifact_ref, lifecycle_evidence_ref, validation_refs | redact private refs | `090.GraphErrorContext` | `graph-error-graph-artifact-inactive` |
| `GRAPH_ARTIFACT_CHECKSUM_MISMATCH` | `090` | blocked | policy_change_required | error_code, owner_spec, artifact_class | expected_checksum_ref, actual_checksum_ref, manifest_ref | checksums visible; payload refs redacted | `090.GraphErrorContext` | `graph-error-graph-artifact-checksum-mismatch` |
| `GRAPH_ARTIFACT_SCOPE_MISMATCH` | `090` | blocked | policy_change_required | error_code, owner_spec, artifact_class, requested_scope | artifact_ref, activation_scope, validation_refs | redact private scope values | `090.GraphErrorContext` | `graph-error-graph-artifact-scope-mismatch` |
| `GRAPH_PROJECTION_CORE_OVERRIDE_FORBIDDEN` | `090` | security_error | none | error_code, owner_spec, attempted_contract | artifact_ref, attempted_field, stable_owner_ref | redact artifact payload | `090.GraphErrorContext` | `graph-error-graph-projection-core-override-forbidden` |
| `GRAPH_FLOW_ROLE_EVIDENCE_REQUIRED` | `090` | error | caller_correctable | error_code, owner_spec, graph_edge_type, missing_evidence_class | projection_profile_ref, source_observation_ref, validation_refs | redact private source refs | `090.GraphErrorContext` | `graph-error-graph-flow-role-evidence-required` |
| `GRAPH_EXPIRY_SOURCE_AUTHORITY_REQUIRED` | `090` | blocked | policy_change_required | error_code, owner_spec, requested_effect | graph_delta_ref, missing_absence_ref, source_authority_refs | redact private source refs | `090.GraphErrorContext` | `graph-error-graph-expiry-source-authority-required` |
| `GRAPH_HANDOFF_EFFECT_UNSUPPORTED` | `090` | error | policy_change_required | error_code, owner_spec, graph_handoff_effect | change_set_ref, projection_profile_ref, validation_refs | redact private refs | `090.GraphErrorContext` | `graph-error-graph-handoff-effect-unsupported` |
| `GRAPH_EDGE_SEMANTICS_ROW_MISSING` | `090` | error | policy_change_required | error_code, owner_spec, edge_type | graph_profile_ref, edge_semantics_ref, validation_refs | redact private refs | `090.GraphErrorContext` | `graph-error-graph-edge-semantics-row-missing` |
| `GRAPH_ENDPOINT_IDENTITY_UNRESOLVED` | `090` | error | retry_after_owner_repair | error_code, owner_spec, endpoint_role | unresolved_target_ref, identity_decision_ref, projection_profile_ref | redact identity private refs | `090.GraphErrorContext` | `graph-error-graph-endpoint-identity-unresolved` |
| `GRAPH_QUERY_CANDIDATE_LIMIT_REACHED` | `090` | error | caller_correctable | error_code, owner_spec, query_class, candidate_limit | query_checksum, profile_ref, derived_view_state_ref | redact candidate IDs when unauthorized | `090.GraphErrorContext` | `graph-error-graph-query-candidate-limit-reached` |
| `GRAPH_PAGE_TOKEN_EXPIRED` | `090` | error | caller_correctable | error_code, owner_spec, query_class | token_checksum, expiry, request_checksum | token bytes hidden | `090.GraphErrorContext` | `graph-error-graph-page-token-expired` |
| `GRAPH_PAGE_TOKEN_INVALID` | `090` | security_error | none | error_code, owner_spec, mismatch_class | token_checksum, authorization_checksum, request_checksum | token bytes and backend cursor hidden | `090.GraphErrorContext` | `graph-error-graph-page-token-invalid` |
| `GRAPH_TRAVERSAL_CLASS_REQUIRED` | `090` | error | caller_correctable | error_code, owner_spec, query_class | query_checksum, translation_profile_ref | no raw payloads | `090.GraphErrorContext` | `graph-error-graph-traversal-class-required` |
| `GRAPH_QUERY_TIMEOUT` | `090` | error | transient_retryable | error_code, owner_spec, query_class, timeout_seconds | query_checksum, translation_profile_ref, derived_view_state_ref | redact candidate refs | `090.GraphErrorContext` | `graph-error-graph-query-timeout` |
| `GRAPH_QUERY_TRANSLATION_ERROR` | `090` | error | policy_change_required | error_code, owner_spec, query_class | translation_profile_ref, mapping_row_ref, query_checksum | redact backend query text unless validation permits | `090.GraphErrorContext` | `graph-error-graph-query-translation-error` |
| `GRAPH_DELTA_IDEMPOTENCY_CONFLICT` | `090` | error | none | error_code, owner_spec, idempotency_key_ref | prior_apply_ref, attempted_apply_ref, manifest_ref | no backend IDs | `090.GraphErrorContext` | `graph-error-graph-delta-idempotency-conflict` |
| `GRAPH_BACKEND_PROFILE_MISSING` | `090` | blocked | policy_change_required | error_code, owner_spec, profile_id | selection_policy_ref, validation_refs | redact private backend config | `090.GraphErrorContext` | `graph-error-graph-backend-profile-missing` |
| `GRAPH_SCHEMA_FINGERPRINT_STALE` | `090` | blocked | retry_after_refresh | error_code, owner_spec, backend_profile_id | schema_fingerprint_ref, schema_profile_ref, provider_ref | redact backend internals | `090.GraphErrorContext` | `graph-error-graph-schema-fingerprint-stale` |
| `GRAPH_RAW_WRITE_BYPASS` | `090` | security_error | none | error_code, owner_spec, bypass_class | bypass_evidence_ref, backend_profile_ref, audit_ref | redact command text and private refs | `090.GraphErrorContext` | `graph-error-graph-raw-write-bypass` |
| `GRAPH_DRIFT_REPAIR_FORBIDDEN` | `090` | security_error | none | error_code, owner_spec, attempted_effect | drift_check_ref, attempted_mutation_ref | redact attempted payload | `090.GraphErrorContext` | `graph-error-graph-drift-repair-forbidden` |
| `GRAPH_APPLY_RESUME_UNSAFE` | `090` | blocked | retry_after_owner_repair | error_code, owner_spec, apply_result_ref | committed_batch_refs, resume_boundary_ref, idempotency_key_ref | redact backend transaction refs | `090.GraphErrorContext` | `graph-error-graph-apply-resume-unsafe` |
| `GRAPH_REBUILD_EQUIVALENCE_FAILED` | `090` | blocked | retry_after_owner_repair | error_code, owner_spec, rebuild_manifest_ref | output_checksum_refs, schema_fingerprint_ref, index_check_ref | redact backend internals | `090.GraphErrorContext` | `graph-error-graph-rebuild-equivalence-failed` |
| `THEORETICAL_REACHABILITY_SCOPE_ERROR` | `090` | blocked | policy_change_required | error_code, owner_spec, attempted_output | graph_profile_ref, deferred_doc_ref, validation_refs | no private refs | `090.GraphErrorContext` | `graph-error-theoretical-reachability-scope-error` |
| `GRAPH_BACKEND_CONFIG_INCOMPLETE` | `090` | blocked | policy_change_required | error_code, owner_spec, missing_field_path | backend_profile_ref, validation_refs | redact private config values | `090.GraphErrorContext` | `graph-error-graph-backend-config-incomplete` |
| `GRAPH_BACKEND_DEFAULT_UNRESOLVED` | `090` | blocked | policy_change_required | error_code, owner_spec, backend_profile_id | selection_policy_ref, unresolved_field_paths, validation_refs | redact private refs | `090.GraphErrorContext` | `graph-error-graph-backend-default-unresolved` |
| `GRAPH_PROVIDER_CAPABILITY_MISSING` | `090` | blocked | policy_change_required | error_code, owner_spec, capability_matrix_ref | backend_profile_ref, validation_refs | redact private refs | `090.GraphErrorContext` | `graph-error-graph-provider-capability-missing` |
| `GRAPH_PROVIDER_CAPABILITY_UNSUPPORTED` | `090` | blocked | policy_change_required | error_code, owner_spec, capability, affected_class | capability_matrix_ref, backend_profile_ref, validation_refs | redact private refs | `090.GraphErrorContext` | `graph-error-graph-provider-capability-unsupported` |
| `GRAPH_BACKEND_VERSION_UNPINNED` | `090` | blocked | policy_change_required | error_code, owner_spec, version_field_path | backend_profile_ref, package_release_ref | redact private package refs by policy | `090.GraphErrorContext` | `graph-error-graph-backend-version-unpinned` |
| `GRAPH_BACKEND_PACKAGE_GATE_FAILED` | `090` | blocked | retry_after_owner_repair | error_code, owner_spec, package_gate | package_set_ref, package_release_ref, gate_result_ref | package evidence redacted by `110` | `090.GraphErrorContext` | `graph-error-graph-backend-package-gate-failed` |
| `GRAPH_PROVIDER_ADAPTER_UNSUPPORTED` | `090` | blocked | policy_change_required | error_code, owner_spec, provider, adapter_ref | backend_profile_ref, driver_ref, validation_refs | redact private config | `090.GraphErrorContext` | `graph-error-graph-provider-adapter-unsupported` |
| `GRAPH_SCHEMA_IMPLICIT_CREATION_FORBIDDEN` | `090` | blocked | policy_change_required | error_code, owner_spec, schema_policy_ref | provider_config_checksum, backend_profile_ref, validation_refs | redact config payload | `090.GraphErrorContext` | `graph-error-graph-schema-implicit-creation-forbidden` |
| `GRAPH_STORAGE_MODE_UNSAFE` | `090` | blocked | policy_change_required | error_code, owner_spec, storage_backend_kind | backend_profile_ref, storage_config_checksum, validation_refs | redact storage config | `090.GraphErrorContext` | `graph-error-graph-storage-mode-unsafe` |
| `GRAPH_INDEX_BACKEND_UNAVAILABLE` | `090` | blocked | retry_after_owner_repair | error_code, owner_spec, index_backend_kind, affected_query_classes | index_check_ref, backend_profile_ref, package_refs | redact private refs | `090.GraphErrorContext` | `graph-error-graph-index-backend-unavailable` |
| `GRAPH_INDEX_FRESHNESS_REQUIRED` | `090` | blocked | retry_after_refresh | error_code, owner_spec, affected_query_classes | index_freshness_ref, derived_view_state_ref, backend_profile_ref | redact backend internals | `090.GraphErrorContext` | `graph-error-graph-index-freshness-required` |
| `GRAPH_QUERY_FULL_SCAN_FORBIDDEN` | `090` | error | caller_correctable | error_code, owner_spec, query_class | translation_profile_ref, index_check_ref, query_checksum | redact backend query text | `090.GraphErrorContext` | `graph-error-graph-query-full-scan-forbidden` |

### Acceptance Criteria

| ID | Criterion |
| --- | --- |
| `090-DERIVED-GRAPH-EDGE-HANDOFF-AC-001` | Derived graph edge requests succeed only through active `GraphProjectionProfile`, `GraphEdgeSemantics`, `GraphObjectOutputEligibilityRow`, `GraphPropertyEvidencePolicy`, `GraphApplyProfile`, and `DerivedViewLagPolicy`; missing support, missing projection ref, direct mutation, unsupported output, and replay mismatch fail or no-op as specified. |
| `090-GRAPH-QUERY-RESPONSE-HANDOFF-AC-001` | Every `query_class` in `GraphQueryResponseHandoffTo110` emits exactly one owner-level result shape consumed by `110` without backend inference. |
| `090-GRAPH-CONFLICT-VISIBILITY-AC-001` | Conflicted graph objects are visible only when graph object output eligibility permits the query context and never authorize pathfinding, cleanup, or absence by default. |
| `090-GRAPH-AMBIGUOUS-NONMUTATING-AC-001` | Ambiguous graph query states emit a non-mutating empty result, state label, or owner error and never mutate graph, identity, facts, or watermarks. |
| `090-GRAPH-PAGE-TOKEN-TTL-HANDOFF-AC-001` | `090.GraphQueryPageTokenPolicy` imports TTL behavior from `110.PageTokenPolicy` while preserving graph-owned token identity inputs and backend cursor rejection. |
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
| `090-IDENTITY-SPLIT-PROJECTION-AC-001` | Identity split handoff projection consumes `070.GraphCorrectionHandoff` only as projection input and never permits resolver graph mutation. |
| `090-IDENTITY-SPLIT-PROJECTION-AC-002` | The same split handoff, affected fact refs, projection profile, and version manifest produce byte-identical graph deltas across replay. |
| `090-IDENTITY-SPLIT-PROJECTION-AC-003` | Missing split handoff metadata emits no graph delta and rejects before delta persistence. |
| `090-IDENTITY-SPLIT-PROJECTION-AC-004` | `VersionManifest` includes graph correction handoff refs and resolver explanation checksum refs when identity split projection occurs. |
| `090-GRAPH-PROFILE-CLOSURE-AC-001` | `mvp-observed-flow-graph.v1` has exactly one active edge type, `observed_connection`, and rejects all other edge types. |
| `090-GRAPH-PROFILE-CLOSURE-AC-002` | Positive observed connection projection emits nodes and one edge only when both endpoint identities are resolved and qualifying `FlowRoleEvidence` exists. |
| `090-GRAPH-PROFILE-CLOSURE-AC-003` | Missing or ambiguous `FlowRoleEvidence` emits no edge, no absence edge, no expiry, and no watermark. |
| `090-GRAPH-PROFILE-CLOSURE-AC-004` | Generic external graph payloads and synthetic structural objects are non-emitting and not pathfinding-eligible in MVP. |
| `090-GRAPH-QUERY-CLOSURE-AC-001` | Bounded path queries with omitted traversal classes fail, and empty traversal-class arrays return no paths. |
| `090-GRAPH-QUERY-CLOSURE-AC-002` | Candidate-limit, expired-token, and token-mismatch cases fail before backend query execution with the declared graph error codes. |
| `090-GRAPH-APPLY-ORDER-AC-001` | Same graph delta set and apply profile produce byte-identical canonical delta ordering and batch membership. |
| `090-GRAPH-REBUILD-CLOSURE-AC-001` | Rebuild equivalence includes profile, edge semantics, output eligibility, taxonomy, query translation, property policy, schema, index consistency, and canonical output checksums. |
| `090-GRAPH-BACKEND-SELECTION-AC-001` | Graph serving enabled with omitted backend profile materializes `mvp-janusgraph.v1`; graph serving disabled materializes no backend; unresolved default fields block production serving with `GRAPH_BACKEND_DEFAULT_UNRESOLVED`. |
| `090-GRAPH-BACKEND-PREFLIGHT-AC-001` | Backend preflight rejects missing profile, inactive profile, checksum mismatch, omitted required profile fields, unpinned provider version, unsafe storage mode, implicit schema creation, and missing package gates before mutation or query. |
| `090-GRAPH-PROVIDER-CAPABILITY-AC-001` | Every active graph provider profile satisfies `GraphProviderCapabilityMatrix` or declares deterministic unsupported behavior for each query, apply, rebuild, and serving class. |
| `090-GRAPH-JANUSGRAPH-SCHEMA-AC-001` | JanusGraph implicit schema creation, destructive schema drop, stale fingerprint, missing schema constraints without exception, and missing index readiness fail closed. |
| `090-GRAPH-JANUSGRAPH-QUERY-AC-001` | JanusGraph Gremlin translation fixtures produce provider-neutral `QueryGraph` outputs, reject full scans, enforce candidate limits, reject backend IDs, and sort by Cadastre ordering. |
| `090-GRAPH-JANUSGRAPH-APPLY-AC-001` | JanusGraph partial-commit fixtures require storage, composite-index, mixed-index, rollback, committed-batch, read-after-write, and resume-boundary evidence. |
| `090-GRAPH-PROVIDER-PORTABILITY-AC-001` | A future provider can satisfy the same capability, taxonomy, query translation, schema, apply, rebuild, ordering, ID, page-token, and error contracts without JanusGraph-specific fields leaking into Cadastre outputs. |

## Definition of Done

| ID | Criterion |
| --- | --- |
| `090-AC-001` | Graph state is rebuildable from authoritative lakehouse records and persisted graph deltas. |
| `090-AC-002` | Query results are deterministically ordered and never depend on backend natural order or internal IDs. |
| `090-AC-003` | Graph apply rejects stale schema fingerprints and missing constraints before mutation. |
| `090-AC-004` | Graph drift checks cannot repair or mutate state. |
| `090-AC-005` | MVP graph profiles cannot emit `has_theoretical_reachability`, modeled reachability facts, or equivalent graph properties. |
| `090-AC-006` | Graph direction for observed connections derives from Cadastre `FlowRoleEvidence`, not OCSF endpoint order or backend edge convention. |
| `090-AC-007` | The active MVP graph profile is `mvp-observed-flow-graph.v1` and emits only resolved canonical nodes plus `observed_connection` edges. |
| `090-AC-008` | Query translation enforces Cadastre-owned candidate limits, page-token TTLs, traversal-class behavior, output eligibility, authorization, and redaction. |
| `090-AC-009` | Graph rebuild promotion compares the complete graph profile, schema, taxonomy, query translation, output eligibility, property policy, index consistency, and canonical output checksum inputs. |

## Open Questions

Open questions marked `TODO:` block authoritative status for the affected contract. A downstream implementation must not resolve a `TODO:` by inference.

| `090-TODO-JANUSGRAPH-RELEASE-PIN` | TODO: Pin JanusGraph provider release/package, provider adapter package, driver package, and TinkerPop version refs before active production profile. | P0 before active production profile. | Product governance plus `100` package-set evidence. | Validation-only profile may remain selected but blocked for production. |
| `090-TODO-JANUSGRAPH-STORAGE-BACKEND` | TODO: Select durable storage backend or require explicit deployment-supplied storage backend configuration and validation refs. | P0 unless deployment config explicitly supplies and validates it. | Product governance plus `100` package gates and `120` fixtures. | `mvp-janusgraph.v1` fails with `GRAPH_BACKEND_DEFAULT_UNRESOLVED`. |
| `090-TODO-JANUSGRAPH-INDEX-BACKEND` | TODO: Select index backend or declare query classes unsupported. | P0 for query classes requiring mixed or text/range index support. | Product governance plus `120-GRAPH-JANUSGRAPH-INDEX-*`. | Affected query classes are blocked. |
| `090-TODO-JANUSGRAPH-BIGTABLE` | TODO: Resolve Bigtable adapter behavior if Bigtable is selected. | P2 unless selected, then P0. | JanusGraph source inspection and provider fixtures. | Bigtable unsupported for MVP. |
| `090-TODO-JANUSGRAPH-GRPC` | TODO: Resolve gRPC management surface before enabling it. | P2 unless enabled, then P0. | `090`, `100`, `110`, and `120`. | gRPC disabled by default. |
| `090-TODO-JANUSGRAPH-CONFIG-PRECEDENCE` | TODO: Confirm complete selected-field configuration precedence. | P1 for selected profile fields. | Provider source inspection and config fixtures. | Omitted or ambiguous selected config fields fail closed. |
| `090-TODO-JANUSGRAPH-DOCKER-ENTRYPOINT` | TODO: Inspect Docker/container entrypoint before containerized mode becomes MVP default. | P1 if containerized mode is MVP default. | Package and deployment validation. | Container entrypoint is not a production default. |
| `090-TODO-JANUSGRAPH-ID-BINARY-ENCODING` | TODO: Resolve full ID binary encoding only if migration/export depends on it. | P2 unless migration/export depends on binary encoding. | Provider source inspection. | Backend IDs remain forbidden. |
| `090-TODO-JANUSGRAPH-QUERY-OPTIMIZER` | TODO: Trace query optimizer strategies for performance-sensitive portability. | P1 for performance-sensitive query portability. | Provider source inspection and query fixtures. | Full-scan and natural-order behavior remain forbidden. |
| `090-TODO-JANUSGRAPH-EXAMPLES` | TODO: Treat examples as non-blocking rationale only unless adopted by owner validation rows. | Non-blocking. | Owner spec review. | Examples do not define behavior. |
