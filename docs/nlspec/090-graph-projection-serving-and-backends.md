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
- `ObservabilityInstrumentationProfile`
- `TelemetryAttributePolicy`
- `TelemetryRedactionPolicy`
- `TelemetryRuntimeState`

- `GraphNodeDeltaShape`
- `GraphEdgeDeltaShape`
- `EvidenceRef`
- `CoreRecordValidationAlgorithm`
- `ActivationControlledArtifactRef`
- `GoldFactPredicateContractRow`
- `GoldFactSubjectRefKindRegistry`
- `GoldFactObjectValueKindRegistry`
- `030.ScopeSelector`
- `030.ActivationScope`
- `030.ScopeSelectorContext`
- `030.ResolveScopedRow`
- `030.ActivationControlledRowSchema`
- `030.ActivationControlledRowField`
- `030.ActivationControlledRowRef`
- `030.ActivationControlledRowSetSchema`
- `020.SourceDatasetCatalogRow`

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
- `GraphEdgeSemanticsRegistry`
- `GraphObjectOutputEligibilityRow`
- `GraphObjectOutputEligibilityRowSet`
- `GraphActiveProfileClosure`
- `GraphBackendSelectionDefaultDecision`
- `GraphBackendActivationBlockerSet`

## Projection Authority

Graph state is a derived read model. It must be rebuildable from authoritative lakehouse records, persisted graph deltas, and active projection/apply/schema profiles.

`GraphNodeDelta` and `GraphEdgeDelta` are concrete 090 records that must conform to `040.GraphNodeDeltaShape` and `040.GraphEdgeDeltaShape` before apply. `090` owns projection/apply semantics and runtime graph delta ID input policies.

A graph backend must not create authoritative facts, infer identity, decide source completeness, decide fact retraction, define bitemporal semantics, repair drift by itself, or expose backend internal IDs as Cadastre IDs.

### GraphTelemetryHandoff

Graph projection, graph apply, graph query, graph rebuild, graph backend preflight, graph drift check, and derived-view lag operations may emit telemetry only through `140.EmitTelemetry`.

Graph telemetry must not expose backend-generated node IDs, edge IDs, relationship IDs, vertex IDs, document IDs, element IDs, provider-native query text, raw graph property values, raw payload bytes, private source bindings, or graph backend credentials.

`diagnostic_correlation_ref`, trace ID, span ID, `server_correlation_nonce`, exporter runtime ID, Collector runtime ID, telemetry runtime state ID, telemetry backend IDs, and exporter profile runtime ID must not appear in graph node IDs, graph edge IDs, graph properties, graph selectors, page tokens, path queries, drift checks, apply idempotency keys, rebuild manifests, derived-view-state identifiers, graph evidence output, graph profile context, or backend schema fingerprints.

`diagnostic_correlation_ref` may appear only in `110.CommonApiResponseEnvelope` when `110` and `140` permit it. It must never appear in `GraphQueryResponse.nodes`, `GraphQueryResponse.edges`, `GraphQueryResponse.paths`, `GraphQueryResponse.evidence`, `derived_view_state_ref`, or `graph_profile_context`.

Graph telemetry, graph backend metrics, graph backend health, graph apply span success, graph query span success, graph rebuild span success, and graph drift-check telemetry must not create facts, infer identity, decide source completeness, authorize graph expiry or cleanup, repair drift, advance derived-view state, or satisfy graph rebuild equivalence.

Provider-native telemetry attributes must be translated through `140.TelemetryAttributePolicy` and redacted through `140.TelemetryRedactionPolicy` before export.

### GraphExpirySourceAuthorityGate

`GraphProjectionProfile` may emit graph expiry or cleanup deltas only when the source `GoldFactChangeSet` or `060.AbsenceDerivationResult` authorizes `graph_expiry` or `cleanup` for the requested effect token.

`GraphApplyResult`, `GraphIndexConsistencyCheck`, `DerivedViewState`, graph backend import success, graph drift check, missing graph object, graph apply time, graph backend cleanup, or derived-view lag must not authorize graph expiry or cleanup.

`source_stale`, `derived_view_stale`, `unknown`, `permission_limited`, `partial_known_gap`, `partial_unknown_gap`, `source_unavailable`, `scope_unavailable`, `not_checked`, `error`, and `ambiguous` must produce no graph expiry unless an exact active `060` row authorizes the requested effect.

Missing flow evidence must not emit an observed-connection absence edge or graph expiry. It maps to `unknown` and produces no graph mutation.

### SourceAuthorityClosureGraphEffectGate

Graph expiry and cleanup deltas require all of the following:

1. `060.AbsenceDerivationResult.requested_effect` equals the graph handoff effect.
2. `060.AbsenceDerivationResult.allowed_effects` contains the requested effect.
3. The selected `060.SourceAuthorityClosureMatrixRow.closure_outcome` maps to `000.closed_active` through `000.SourceAuthorityClosureStateCrosswalk`.
4. The producing `VersionManifest` includes the selected `020.SourceDatasetCatalogRow` ref/checksum and every underlying `060` row ref/checksum consulted by the requested effect.

`closure_outcome = deterministically_blocked` and `closure_outcome = blocked_validation` must emit no graph delta. A deterministic block row may be recorded as a graph no-op diagnostic only; it must not become graph expiry, cleanup, retraction, absence, or watermark permission. Missing source-dataset catalog rows or missing `VersionManifest` refs reject before delta persistence.

#### SourceEffectGraphOutcomeMatrix

| Upstream closure state or blocker | Required `090` behavior |
| --- | --- |
| `closed` for `graph_expiry` | May emit expiry delta only through an active graph profile and only when the producing `VersionManifest` includes source-dataset, selected closure row, `AbsenceDerivationResult`, and all consulted underlying `060` refs. |
| `closed` for `cleanup` | May emit cleanup delta only through an active graph profile and only when the producing `VersionManifest` includes source-dataset, selected closure row, `AbsenceDerivationResult`, and all consulted underlying `060` refs. |
| `closed_active` from `000.SourceAuthorityClosureStateCrosswalk` | Same as matching `060.closure_outcome = closed`; no graph output is allowed unless the requested effect is `graph_expiry` or `cleanup`. |
| `deterministically_blocked` or `closed_deterministically_blocked` | Emit graph no-op diagnostic only; no `GraphDeltaSet`, no backend mutation, no expiry, no cleanup, no retraction, no absence edge, and no watermark. |
| `blocked_validation` | Reject before delta persistence with owner validation error. |
| `blocked_missing_ref` | Reject before delta persistence with missing-ref owner error. |
| `blocked_checksum` | Reject before delta persistence with checksum error. |
| `blocked_todo` | Reject before delta persistence and promotion with TODO blocker. |
| `blocked_package_set` | Reject before delta persistence and candidate package output visibility. |
| `blocked_manifest` or missing `VersionManifest` refs | Reject with manifest error before delta persistence. |
| missing source-dataset catalog row | Reject before delta persistence. |
| ambiguous source-dataset catalog row | Reject before delta persistence; no row-order tie break. |
| inactive, unvalidated, private-leaking, or package-set-mismatched source-dataset row | Reject before delta persistence. |

Source-history no-change proof, graph read-model drift checks, missing backend objects, derived-view stale state, graph apply success, graph backend cleanup, and backend query results must never authorize graph expiry or cleanup. They may affect graph output only when the producing `GoldFactChangeSet` or `060.AbsenceDerivationResult` authorizes `graph_expiry` or `cleanup` for the exact requested effect and the selected closure refs are manifest-included.

### SourceDatasetGraphClosureHandoff

Graph expiry, cleanup, and any graph delta whose source-effect dependency is absence-sensitive must include the selected `020.SourceDatasetCatalogRow` ref/checksum, the selected `060.SourceAuthorityClosureMatrixRow` ref/checksum or deterministic block row ref/checksum, every underlying consulted `060` row ref/checksum, the `060.AbsenceDerivationResult` ref/checksum when absence or effect authorization is evaluated, and the producing `030.VersionManifest` ref before delta persistence and before backend mutation.

`network_flow` may project only positive `gfp-mvp-flow-observed-connection-v1` facts in MVP. Flow non-observation, missing flow-role evidence, ambiguous flow-role evidence, missing source-dataset catalog rows, missing source-authority closure refs, OCSF endpoint order, and unresolved endpoint identity must emit no edge, no absence edge, no expiry, no cleanup, no graph property, no pathfinding input, and no backend mutation. The allowed positive edge remains subject to `050.FlowRoleEvidence`, `070` endpoint identity handoff, active `GraphEdgeSemantics`, active `GraphObjectOutputEligibilityRow`, and graph profile closure.

`future_reachability` deterministic block rows must map to inactive `has_theoretical_reachability` behavior. While `200` remains `inactive_deferred`, a graph profile, backend profile, analysis rule, package release, query profile, or validation row must not activate `has_theoretical_reachability`, modeled reachability facts, boolean reachability properties, or unqualified reachability wording.

### GoldFactChangeSet graph handoff matrix

`090.ProjectGraphDeltas` consumes `080` graph handoff effects. `080` emits metadata only; graph projection remains a deterministic derived view.

| `080` graph handoff effect | Required `090` projection behavior | Required authorization refs | Default when unsupported |
| --- | --- | --- | --- |
| `none` | Emit no graph delta. | `GoldFactChangeSet` ref only. | no-op. |
| `reproject_fact_key` | Recompute projected graph objects for the affected fact key using the active `GraphProjectionProfile`. | `GoldFactChangeSet` ref, temporal refs, projection profile ref. | `GRAPH_HANDOFF_EFFECT_UNSUPPORTED`. |
| `expire_projected_object` | Emit expiry delta only when `060.AbsenceDerivationResult.allowed_effects` contains `graph_expiry` and the selected source-dataset catalog row is manifest-included. | `GoldFactChangeSet` ref, `020.SourceDatasetCatalogRow` ref/checksum, and `060.AbsenceDerivationResult` ref. | no graph delta plus `GRAPH_EXPIRY_SOURCE_AUTHORITY_REQUIRED`. |
| `cleanup_projected_object` | Emit cleanup delta only when `060.AbsenceDerivationResult.allowed_effects` contains `cleanup` and the selected source-dataset catalog row is manifest-included. | `GoldFactChangeSet` ref, `020.SourceDatasetCatalogRow` ref/checksum, and `060.AbsenceDerivationResult` ref. | no graph delta plus `GRAPH_EXPIRY_SOURCE_AUTHORITY_REQUIRED`. |
| `source_authority_blocked` | Emit no graph delta and record graph no-op diagnostic. | deterministic source-dataset or source-authority block row ref | no-op; no backend mutation |
| `conflict_visibility_update` | Update graph visibility only when `GraphEdgeSemantics` and `GraphObjectOutputEligibilityRow` permit conflicted or stale assertions. | `GoldFactChangeSet` ref, graph semantics row ref, eligibility row ref. | no pathfinding-visible delta. |
| `identity_split_handoff` | Consume `070.GraphCorrectionHandoff`, validate `retained_canonical_entity_ref`, `new_canonical_entity_refs`, `split_partition_refs`, `affected_fact_refs` or `affected_fact_selection_checksum`, `resolver_explanation_checksum`, and `version_manifest_ref`, sort affected fact refs lexically when enumerated, and route affected facts through the active `GraphProjectionProfile` only. | `GraphCorrectionHandoff` ref/checksum, `split_identity_decision_ref`, `split_policy_ref`, `retained_canonical_entity_ref`, `new_canonical_entity_refs`, `split_partition_refs`, `affected_fact_refs` or `affected_fact_selection_checksum`, projection profile ref, resolver explanation checksum. | `GRAPH_HANDOFF_EFFECT_UNSUPPORTED` when the active projection profile does not support the handoff; missing or invalid metadata rejects before delta persistence. |

Every projected delta from correction must include the source `GoldFactChangeSet` ref, graph handoff effect, projection profile ref, authority or absence result refs when applicable, and validation refs.

Identity split handoff input validation must fail before delta persistence according to this table:

| Missing or invalid input | Required graph behavior |
| --- | --- |
| missing handoff ref | reject before delta persistence |
| checksum-mismatched handoff | reject before delta persistence |
| missing retained or new canonical refs | reject before delta persistence |
| missing affected fact refs and selection checksum | reject before delta persistence |
| unresolved endpoint identity | emit `GRAPH_ENDPOINT_IDENTITY_UNRESOLVED`; no node and no edge |

`090` must not re-evaluate `070` evidence classes, candidate pairs, review cases, blockers, confidence bands, split partitions, or decision rows. Graph projection consumes resolver outputs only through persisted identity decisions, resolver explanations, and `GraphCorrectionHandoff` records.

#### Assertion transition graph validation cross-reference

`090` must validate the graph side of every `080.AssertionStateTransitionRow` output. This table is exhaustive for `080` transition output classes that can affect graph projection.

| `080` transition output | Required `090` validation |
| --- | --- |
| `insert_fact` or `reproject_fact_key` | Recompute projected objects through the active graph profile only. |
| `mark_retracted` with graph expiry allowed | Require `060.AbsenceDerivationResult.allowed_effects` containing `graph_expiry`. |
| cleanup handoff | Require `060.AbsenceDerivationResult.allowed_effects` containing `cleanup`. |
| conflict visibility update | Require active `GraphEdgeSemantics` and `GraphObjectOutputEligibilityRow`. |
| unsafe negative, no-op, or source-authority-blocked | Emit no graph delta, no backend mutation, and a mutation-prohibition proof. |
| identity split handoff | Validate `070.GraphCorrectionHandoff` refs exactly and do not re-evaluate resolver evidence. |

`GraphRebuildEquivalencePolicy` imports `080.ReplayEquivalencePolicyOutputClassRow` and must use exact imported field-selection refs for `graph_delta`, `graph_apply`, and `graph_rebuild`. This document does not add new MVP edge types; `observed_connection` remains the only active MVP edge, and theoretical reachability remains prohibited.

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

### MVP Graph Active Profile Closure Requirements

The MVP active graph profile must close graph semantic selection before projection, query serving, graph apply, rebuild promotion, or analysis compatibility can execute.

| Closure item | Required behavior |
| --- | --- |
| active edge set | Exactly one active `GraphEdgeSemantics` row for `observed_connection`. |
| all other edge types | Inactive rows or exact deterministic block rows, including `has_theoretical_reachability`. |
| output eligibility | Exactly one active `GraphObjectOutputEligibilityRowSet` controlling search, neighbor expansion, pathfinding, analysis findings, metrics, and identity influence. |
| missing `FlowRoleEvidence` | Emit no edge and no mutation. |
| ambiguous `FlowRoleEvidence` | Emit no edge and no mutation. |
| unresolved endpoint identity | Emit no edge and no node. |
| OCSF endpoint order | Never supplies graph direction. |
| predicate contract | `observed_connection` projection requires `required_gold_fact_predicate_contract_ref = gfp-mvp-flow-observed-connection-v1`; missing, mismatched, or unmanifested refs emit no edge. |
| predicate row manifest | The selected predicate row ref and checksum must appear in `VersionManifest` with graph projection refs before edge persistence. |

### MVP Graph Backend Selection Closure Requirements

`mvp-postgresql-relational-graph.v1` is the default selection decision only when graph serving is enabled and no backend profile is supplied. Default selection emits `GraphBackendSelectionDefaultDecision` and must not mutate, query, rebuild, promote, drift-check, or serve graph state. `mvp-postgresql-age.v1` may be selected only by explicit input or explicit validation scope; it is not the omitted default.

#### PostgreSQLRelationalDefaultBackendProfileClosure

Omitted backend selection materializes only `mvp-postgresql-relational-graph.v1`. The selection result is a `GraphBackendSelectionDefaultDecision`; it is not a preflight result, package activation result, schema proof, query proof, apply proof, rebuild proof, health success, drift check, or promotion proof.

Selection alone must not run backend preflight, open a backend connection, execute a graph query, apply a graph delta, rebuild graph state, perform a drift check, mark health healthy, advance `DerivedViewState`, or promote graph serving. Any unresolved default blocker emits `GRAPH_BACKEND_DEFAULT_UNRESOLVED` unless a more specific owner error applies.

`GraphBackendActivationBlockerSet` must contain one row, resolved or deterministically blocked, for each blocker below before backend mutation, query serving, rebuild promotion, drift check, health success, or graph-serving output:

| Blocker family | Required row | Default when missing, expired, mismatched, or `TODO:` |
| --- | --- | --- |
| `postgresql_runtime_distribution_ref` | Exact `100.PackageReleaseManifest` and package-set refs for the PostgreSQL runtime distribution. | `GRAPH_BACKEND_DEFAULT_UNRESOLVED` or `GRAPH_BACKEND_PACKAGE_GATE_FAILED`. |
| `postgresql_provider_version_ref` | Exact provider engine version ref and compatibility row. | `GRAPH_BACKEND_VERSION_UNPINNED`. |
| `postgresql_provider_adapter_package_ref` | Exact `graph_provider_adapter_package` release ref and checksum. | `GRAPH_BACKEND_PACKAGE_GATE_FAILED`. |
| `postgresql_driver_package_ref` | Exact `graph_backend_driver_package` release ref, driver protocol compatibility row, and checksum. | `GRAPH_BACKEND_PACKAGE_GATE_FAILED`. |
| `postgresql_deployment_profile_ref` | Exact `graph_backend_deployment_profile` package ref for provider, region or redacted region ref, service tier, role model, backup/restore, and upgrade scope. | `GRAPH_BACKEND_CONFIG_INCOMPLETE`. |
| `provider_support_evidence_ref` | Active `GraphProviderSupportEvidenceRow`; expired evidence behaves as missing. | `GRAPH_PROVIDER_SUPPORT_EVIDENCE_MISSING` or `GRAPH_PROVIDER_SUPPORT_EVIDENCE_EXPIRED`. |
| `graph_backend_activation_blocker_set_ref` | Selected blocker set ref and checksum. | `GRAPH_BACKEND_DEFAULT_UNRESOLVED`. |
| `graph_provider_capability_matrix_ref` | Selected `GraphProviderCapabilityMatrix` row-set ref and checksum. | `GRAPH_PROVIDER_CAPABILITY_MISSING`. |
| `graph_read_model_schema_profile_ref` | Selected relational schema profile ref for `mvp-postgresql-relational-schema.v1`. | `GRAPH_BACKEND_CONFIG_INCOMPLETE`. |
| `backend_schema_fingerprint_ref` | Current `BackendSchemaFingerprint` for the selected profile and deployment scope. | `GRAPH_SCHEMA_FINGERPRINT_STALE`. |
| `role_rls_search_path_preflight_ref` | Selected `RoleRlsSearchPathPreflightRow` ref and checksum. | `GRAPH_POSTGRES_SECURITY_PREFLIGHT_UNSAFE`. |
| `graph_backend_taxonomy_mapping_profile_ref` | Selected backend taxonomy mapping ref and checksum. | `GRAPH_BACKEND_CONFIG_INCOMPLETE`. |
| `graph_query_translation_profile_ref` | Selected query translation row-set ref and checksum. | `GRAPH_QUERY_TRANSLATION_ERROR`. |
| `graph_apply_profile_ref` | Selected apply profile ref and checksum. | `GRAPH_BACKEND_CONFIG_INCOMPLETE`. |
| `restore_rehearsal_evidence_ref` | Passing `GraphRestoreRehearsalEvidenceRow`. | `GRAPH_BACKEND_RESTORE_UNVERIFIED`. |
| `upgrade_rehearsal_evidence_ref` | Passing `GraphUpgradeRehearsalEvidenceRow` or rebuild migration proof. | `GRAPH_BACKEND_UPGRADE_UNVERIFIED`. |
| `benchmark_threshold_profile_refs` | `GraphBenchmarkThresholdProfile` rows for every benchmark tier in promotion scope. | `GRAPH_BENCHMARK_THRESHOLD_MISSING`. |
| `error_registry_fragment_refs` | Generated `110.ErrorCodeRegistryRow` refs for every backend-visible owner error. | `ERROR_REGISTRY_OWNER_FRAGMENT_INCOMPLETE`. |
| `telemetry_redaction_validation_refs` | `140` telemetry redaction and cardinality validation refs for backend diagnostics. | `TELEMETRY_ATTRIBUTE_FORBIDDEN` or `GRAPH_BACKEND_DEFAULT_UNRESOLVED`. |
| `validation_refs` | Concrete `120` rows with fixture checksums, expected output or error checksums, mutation-prohibition proofs, and acceptance criterion IDs. | blocked validation result; no production effect. |
| `package_set_manifest_ref` | Active `100.ProductionPackageSetManifest` ref and checksum containing every selected backend package release. | `GRAPH_BACKEND_PACKAGE_COHORT_MISMATCH` or `GRAPH_BACKEND_PACKAGE_GATE_FAILED`. |
| `version_manifest_requirement` | `030.VersionManifest` refs include every selected row, checksum, package ref, evidence ref, validation ref, generated error registry checksum, and telemetry ref. | `VERSION_MANIFEST_INCOMPLETE`. |

PostgreSQL relational rows are the current omitted-backend default candidate. AGE rows are optional and validation-only unless every AGE production gate passes.

#### PackageActivationHandoff

Graph backend package blockers consume package eligibility only through `100`. When a graph backend runtime, provider adapter, driver, storage backend adapter, index backend adapter, runtime distribution, deployment profile, schema profile, query translation artifact, apply profile, restore/upgrade artifact, benchmark artifact, or provider support artifact is package-supplied, backend preflight must require all of the following refs before mutation, query serving, rebuild promotion, drift check, health success, or graph-serving output:

| Required package evidence | Required behavior |
| --- | --- |
| `100.PackageReleaseManifest` | Each backend package release must be immutable, checksum-valid, trust-verified, compatible, validated, and selected by package type. |
| `100.ProductionPackageSetManifest` | The active package set must include every selected backend package release and checksum. |
| selected `100.PackageTypePolicyRow` refs | Each package type token must resolve exactly one active row for the target environment. |
| selected deprecation rows | Missing or expired rows block backend preflight; no default window may be invented. |
| selected compatibility rows | Runtime, adapter, driver, schema, query, apply, restore, upgrade, benchmark, trust, SBOM, provenance, dependency, and deployment axes must pass when required. |
| trust and repository evidence | Signatures, trust roots, signer authorization, transparency, repository metadata, freshness, snapshot, and anti-rollback refs must pass before backend use. |
| validation refs and package error parity | `120` package rows and `110` package error parity must pass with concrete checksums. |
| `030.VersionManifest` refs | Every selected package ref class must appear in `VersionManifest.included_refs`. |

`090` must not independently decide package eligibility. A graph backend package gate that relies on backend defaulting, provider labels, package names, broad package labels, module names, version strings, repository refs, or validation summaries without the `100` refs above fails with `GRAPH_BACKEND_PACKAGE_GATE_FAILED` or the most specific package owner error before backend preflight succeeds.
PostgreSQL relational rows are the current omitted-backend default candidate. AGE rows are optional and validation-only unless every AGE production gate passes.

### GraphScopeSelectorContextSet

`GraphScopeSelectorContextSet` is the owner context family for graph serving, graph projection, graph backend, backend taxonomy, query translation, apply profile, and schema profile activation. Each context instantiates `030.ScopeSelectorContext`; graph identity, graph truth, graph edge semantics, traversal eligibility, backend query translation, and graph apply behavior remain owned by `090`.

| Context name | Row families covered | Required dimensions | Optional dimensions | Subset behavior |
| --- | --- | --- | --- | --- |
| `graph_serving_scope` | graph serving profiles and derived-view state visibility | `graph_profile_id`, `query_class` | `consumer_scope`, `dataset_version_ref` | none by default. |
| `graph_projection_scope` | `GraphProjectionProfile` rows | `graph_profile_id`, `fact_type`, `predicate` | `edge_type`, `node_type` | none by default. |
| `backend_profile_scope` | `GraphBackendProfile` rows | `provider_id`, `graph_profile_id` | `storage_backend_kind`, `index_backend_kind`, `deployment_scope` | none by default. |
| `backend_taxonomy_scope` | `GraphBackendTaxonomyMappingProfile` rows | `provider_id`, `graph_profile_id` | `node_type`, `edge_type`, `property_path` | none by default. |
| `query_translation_scope` | `GraphQueryTranslationProfile` rows | `query_class`, `provider_id`, `graph_profile_id` | `node_type`, `edge_type`, `operation` | none by default. |
| `apply_profile_scope` | `GraphApplyProfile` rows | `graph_profile_id`, `provider_id`, `operation` | `deployment_scope` | none by default. |
| `schema_profile_scope` | `GraphReadModelSchemaProfile` rows | `graph_profile_id`, `provider_id` | `node_type`, `edge_type`, `property_path` | none by default. |

Graph profile and backend selection algorithms must call `030.ResolveScopedRow` after non-scope predicates such as graph profile ID, provider ID, query class, edge type, node type, or requested operation match. Ambiguous graph profile or backend profile scope produces no graph delta and no backend mutation.

Selector-only endpoint strings, graph backend IDs, backend labels, provider-native query text, backend-generated IDs, and backend internal cursor values are forbidden as identity selectors, evidence selectors, replay keys, drillback keys, response IDs, or page-token identity. This guard is independent of scope matching.

### GraphBackendSelectionPolicy

Graph backend selection is active only when graph serving, graph apply, graph query, graph rebuild, graph drift check, or graph-serving promotion is in implementation scope. Backend selection is not fact authority, identity authority, source-completeness authority, or graph truth.

| Input condition | Required behavior |
| --- | --- |
| graph serving disabled | No backend profile is selected; graph-serving endpoints remain unavailable or validation-only according to `110`. |
| graph serving enabled and `graph_backend.profile_id` omitted | Materialize `graph_backend.profile_id = mvp-postgresql-relational-graph.v1` and record a defaulting decision ref. |
| graph serving enabled and `graph_backend.profile_id` supplied | Resolve exactly one active `GraphBackendProfile` with that ID. |
| selected profile missing | Fail with `GRAPH_BACKEND_PROFILE_MISSING` before backend mutation or query. |
| selected profile inactive | Fail with `GRAPH_ARTIFACT_INACTIVE` before backend mutation or query. |
| selected profile checksum mismatch | Fail with `GRAPH_ARTIFACT_CHECKSUM_MISMATCH` before backend mutation or query. |
| required profile field omitted | Fail with `GRAPH_BACKEND_CONFIG_INCOMPLETE`; do not inherit provider runtime defaults. |
| selected default profile contains unresolved production fields | Fail with `GRAPH_BACKEND_DEFAULT_UNRESOLVED` before backend mutation, query, rebuild promotion, or drift check. |

### GraphBackendSelectionClosureStatus

`GraphBackendSelectionPolicy` owns default backend selection. `mvp-postgresql-relational-graph.v1` is the default selection token only. Production mutation, query, rebuild promotion, drift check, and graph-serving output remain blocked until `GraphBackendProfile`, `GraphProviderCapabilityMatrix`, `GraphReadModelSchemaProfile`, `GraphQueryTranslationProfile`, backend taxonomy mapping, package refs, provider support refs, relational schema fingerprint refs, role/security preflight refs, restore/upgrade evidence refs, and `120` graph backend validation rows pass. `mvp-postgresql-age.v1` is explicit and validation-only unless AGE production gates pass.

| Selection state | Required behavior |
| --- | --- |
| graph serving disabled | No backend selection output and no backend activation blocker. |
| default token materialized | Emit `GraphBackendSelectionDefaultDecision`; do not mutate, query, rebuild, or promote graph serving by selection alone. |
| selected backend profile unresolved | Fail with `GRAPH_BACKEND_DEFAULT_UNRESOLVED` or the most specific owner error before backend interaction. |
| selected backend profile validated but activation gates blocked | Preserve selection evidence and block production backend effects. |
| all backend, package, schema, capability, and validation gates pass | Backend profile may be used only through `GraphApplyProfile`, `GraphQueryTranslationProfile`, `GraphReadModelSchemaProfile`, and derived-view gates. |

### GraphBackendProfile schema

A `GraphBackendProfile` is provider-neutral. Provider-specific fields may appear only inside declared provider subprofiles and must not alter Cadastre graph IDs, output ordering, page-token identity, error categories, evidence refs, or replay semantics.

| Field | Required | Default or omission behavior | Rule |
| --- | ---: | --- | --- |
| `backend_profile_id` | Yes | none | Stable ID. MVP omitted default is `mvp-postgresql-relational-graph.v1`; `mvp-postgresql-age.v1` require explicit selection. |
| `provider` | Yes | none | Closed token. Current supported candidate tokens are `postgresql`, `postgresql_age`. |
| `provider_version_ref` | Yes | none | Immutable version or package ref; snapshot-only refs fail production activation unless package policy permits validation-only use. |
| `provider_adapter_ref` | Yes | none | Ref to adapter contract and package. |
| `query_language` | Yes | none | Current candidate values are `sql` for PostgreSQL relational, `sql_cypher` for AGE. |
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
| `raw_write_bypass_policy_ref` | Yes | none | Must prove raw admin/import writes are blocked, detected, or excluded from production. |
| `package_set_manifest_ref` | Yes | none | Active `100.ProductionPackageSetManifest`. |
| `validation_refs` | Yes | none | Non-empty refs to `120` backend-profile validation rows. |
| `activation_scope` | Yes | none | `030.ActivationScope` for graph serving scope. |
| `lifecycle_status` | Yes | none | Production use requires `active`. |

### GraphBackendActivationControlledRowFieldClosure

The backend row families below are the complete activation-controlled closure set for backend selection, PostgreSQL relational production eligibility, optional AGE production eligibility, provider portability, backend health, and backend validation. Each listed row family must be represented as a `030.ActivationControlledRowSetSchema` whose rows are described by `030.ActivationControlledRowField` tables using exactly the `030` column order: `field_path`, `type`, `required`, `default`, `null_allowed`, `omit_allowed`, `bounds`, `array_semantics`, `duplicate_policy`, `canonical_sort_key`, `id_input`, `checksum_input`, `extension_policy`, `redaction_owner`, `version_manifest_requirement`, `missing_error`, and `invalid_error`.

The shared field rows below apply to every backend row family in this closure set unless a row-family table below narrows the bound. They must not be omitted by a concrete backend row schema.

| field_path | type | required | default | null_allowed | omit_allowed | bounds | array_semantics | duplicate_policy | canonical_sort_key | id_input | checksum_input | extension_policy | redaction_owner | version_manifest_requirement | missing_error | invalid_error |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `row_id` | `040.ScalarType.token` | yes | none | no | no | non-empty lower-kebab or lower-snake token, maximum 128 code points | n/a | reject | n/a | ordered:1 | yes | closed | `110` | selected row ref and row checksum | `GRAPH_BACKEND_CONFIG_INCOMPLETE` | `GRAPH_BACKEND_CONFIG_INCOMPLETE` |
| `row_schema_version` | `040.ScalarType.token` | yes | none | no | no | immutable version token, maximum 64 code points | n/a | reject | n/a | ordered:2 | yes | closed | `110` | row schema version | `GRAPH_BACKEND_CONFIG_INCOMPLETE` | `GRAPH_BACKEND_CONFIG_INCOMPLETE` |
| `activation_scope` | `030.ActivationScope` | yes | none | no | no | must cover selected graph serving scope through `GraphScopeSelectorContextSet` | n/a | reject | n/a | ordered:3 | yes | closed | `110` | selector context ref, request selector checksum, selected row selector checksum | `GRAPH_BACKEND_CONFIG_INCOMPLETE` | `GRAPH_ARTIFACT_CHECKSUM_MISMATCH` |
| `lifecycle_status` | owner enum | yes | none | no | no | production selection requires `active`; validation-only rows may use `validated` or `canary` when owner validation declares scope | n/a | reject | n/a | no | yes | closed | `110` | lifecycle transition evidence refs | `GRAPH_ARTIFACT_INACTIVE` | `GRAPH_ARTIFACT_INACTIVE` |
| `validation_refs` | array of `030.ActivationControlledArtifactRef` | yes | none | no | no | non-empty for production or validation effect | canonical_set | reject | `artifact_ref` | no | yes | closed | `110` | every selected validation ref | `GRAPH_BACKEND_CONFIG_INCOMPLETE` | `GRAPH_ARTIFACT_CHECKSUM_MISMATCH` |
| `package_set_ref` | `030.ActivationControlledArtifactRef` | conditional:row_family_is_package_or_runtime_supplied | none | no | no | must reference active `100.ProductionPackageSetManifest` when any row or runtime evidence is package-supplied | n/a | reject | n/a | no | yes | closed | `110` | package-set ref and checksum | `GRAPH_BACKEND_PACKAGE_GATE_FAILED` | `GRAPH_BACKEND_PACKAGE_GATE_FAILED` |
| `row_checksum` | `040.ScalarType.sha256` | yes | derived:`040.CanonicalJSON` over row bytes excluding `row_checksum` | no | no | SHA-256 lowercase hex | n/a | reject | n/a | no | no | closed | `110` | selected row checksum | `GRAPH_ARTIFACT_CHECKSUM_MISMATCH` | `GRAPH_ARTIFACT_CHECKSUM_MISMATCH` |

The row-family field inventory below is exhaustive for backend selection and PostgreSQL/AGE closure. A concrete row schema may add only a typed, bounded, checksummed, redaction-owned extension map that does not redefine stable `090` behavior. Unknown fields fail before backend interaction.

| Row family | Required family-specific field paths | Required defaults and bounds | ID and checksum inputs | Required manifest refs | Missing or invalid error |
| --- | --- | --- | --- | --- | --- |
| `GraphBackendProfile` | `backend_profile_id`, `provider`, `provider_version_ref`, `provider_adapter_ref`, `query_language`, `driver_ref`, `storage_backend_kind`, `index_backend_kind`, `schema_policy_ref`, `taxonomy_mapping_ref`, `query_translation_profile_ref`, `apply_profile_ref`, `capability_matrix_ref`, `raw_write_bypass_policy_ref`, `package_set_manifest_ref`, `provider_support_evidence_ref`, `activation_blocker_set_ref` | `provider` is one of `postgresql`, or `postgresql_age`; omitted backend profile defaults only to `mvp-postgresql-relational-graph.v1`; storage and index provider defaults are forbidden | `backend_profile_id`, `provider`, `row_schema_version`, `activation_scope`; all listed refs are checksum inputs | selected profile row, row-set checksum, defaulting decision ref when omitted, package-set ref, provider support ref, validation refs | `GRAPH_BACKEND_PROFILE_MISSING`, `GRAPH_BACKEND_CONFIG_INCOMPLETE`, `GRAPH_BACKEND_DEFAULT_UNRESOLVED` |
| `GraphBackendActivationBlockerSet` | `backend_profile_ref`, `blocker_rows`, `resolved_blocker_rows`, `deterministic_block_rows`, `production_effect_allowed` | `production_effect_allowed` defaults to `false`; empty `blocker_rows` is invalid for production | `backend_profile_ref`, canonical blocker row IDs; all blocker refs are checksum inputs | every blocker row, deterministic block row, package-set ref, validation ref | `GRAPH_BACKEND_DEFAULT_UNRESOLVED` |
| `GraphProviderCapabilityMatrix` | `provider`, `backend_profile_ref`, `query_classes`, `apply_classes`, `rebuild_classes`, `health_classes`, `unsupported_behavior`, `partial_support_behavior` | Unsupported behavior must be `blocked` or `deterministic_no_op`; silent fallback is forbidden | provider, profile ref, class token; all capability decisions are checksum inputs | capability row refs and unsupported validation refs | `GRAPH_PROVIDER_CAPABILITY_MISSING`, `GRAPH_PROVIDER_CAPABILITY_UNSUPPORTED` |
| `GraphProviderSupportEvidenceRow` | `provider`, `region`, `engine_version_ref`, `service_tier`, `support_status`, `backup_restore_support`, `upgrade_path_status`, `evidence_date`, `evidence_expiry`, `package_set_ref`, `validation_refs`, `redaction_classes` | `support_status` is `supported`, `unsupported`, `preview`, or `unknown`; expired evidence behaves as missing; private provider config is forbidden | provider, region token or redacted region ref, engine version ref, service tier, evidence date | provider support evidence ref, expiry, validation refs, package-set ref | `GRAPH_PROVIDER_SUPPORT_EVIDENCE_MISSING`, `GRAPH_PROVIDER_UNSUPPORTED` |
| `GraphReadModelSchemaProfile` | `schema_profile_ref`, `anchor_node_table`, `anchor_edge_table`, `constraints`, `indexes`, `rls_policy_refs`, `search_path_policy_ref`, `jsonb_allowed_paths`, `schema_fingerprint_algorithm_ref` | PostgreSQL relational anchors are required for PostgreSQL and AGE; implicit schema creation is forbidden | schema profile ref and fingerprint algorithm ref; all table/constraint/index refs checksum | schema profile ref, fingerprint ref, role/RLS/search-path preflight refs | `GRAPH_SCHEMA_FINGERPRINT_STALE`, `GRAPH_BACKEND_CONFIG_INCOMPLETE` |
| `BackendSchemaFingerprint` | `backend_profile_ref`, `schema_profile_ref`, `tables`, `columns`, `constraints`, `indexes`, `triggers`, `rls_policies`, `functions`, `extension_versions`, `package_refs`, `fingerprint_checksum` | Missing inventory item invalidates the fingerprint; backend native IDs are forbidden | profile ref, schema profile ref, canonical inventory checksum | fingerprint ref before apply, query, rebuild, restore, upgrade, and replay | `GRAPH_SCHEMA_FINGERPRINT_STALE` |
| `GraphQueryTranslationProfile` | `query_class`, `provider`, `backend_profile_ref`, `candidate_limit_table_ref`, `translation_template_ref`, `expected_query_checksum`, `authorization_ref`, `redaction_ref`, `timeout_policy_ref`, `ordering_policy_ref`, `page_token_policy_ref`, `mutation_proof_ref` | Candidate limits import `GraphQueryCandidateLimitTable`; raw SQL, raw Cypher, backend cursors, and natural row order are forbidden | query class, provider, profile ref, template checksum; translated query checksum is checksum input | translation row, mutation proof, validation refs, generated error registry checksum | `GRAPH_QUERY_PLAN_UNSUPPORTED`, `GRAPH_BACKEND_CONFIG_INCOMPLETE` |
| `GraphApplyProfile` | `apply_profile_ref`, `provider`, `max_batch_deltas`, `min_batch_deltas`, `batch_transaction_boundary`, `retry_policy`, `read_after_write_proof_required`, `resume_policy_ref`, `stale_schema_behavior`, `lock_loss_behavior` | PostgreSQL default `max_batch_deltas = 1000`, minimum `1`, maximum `5000`; retry disabled by default; one declared batch per transaction boundary; stale schema and lock loss block advancement | apply profile ref, provider, batch parameters, resume policy; read-after-write proof checksum | apply profile row, idempotency key, read-after-write proof, resume refs | `GRAPH_BACKEND_CONFIG_INCOMPLETE`, `GRAPH_APPLY_RESUME_BLOCKED` |
| `GraphRestoreRehearsalEvidenceRow` | `backend_profile_ref`, `clean_environment_restore_ref`, `restored_schema_fingerprint_ref`, `index_consistency_check_ref`, `query_parity_ref`, `derived_view_validation_ref`, `restore_result` | `restore_result` must be `pass` before production promotion; stale or missing proof blocks | backend profile ref, restore ref, fingerprint ref, validation ref | restore row, fingerprint, query parity, derived-view validation | `GRAPH_BACKEND_RESTORE_UNVERIFIED` |
| `GraphUpgradeRehearsalEvidenceRow` | `backend_profile_ref`, `from_runtime_ref`, `to_runtime_ref`, `schema_migration_diff_ref`, `rollback_or_rebuild_plan_ref`, `major_version_rehearsal_ref`, `upgrade_result` | `upgrade_result` must be `pass`; AGE rows must include extension upgrade or rebuild migration proof | backend profile ref, runtime refs, migration diff ref | upgrade row, rollback/rebuild refs, package-set ref | `GRAPH_BACKEND_UPGRADE_UNVERIFIED` |
| `GraphBenchmarkThresholdProfile` | `benchmark_profile_ref`, `candidate_limit_table_ref`, `graph_size_tier`, `degree_distribution_ref`, `run_count`, `warmup_rule`, `p50_threshold_ms`, `p95_threshold_ms`, `p99_threshold_ms`, `timeout_expectation`, `overflow_expectation`, `metric_checksum_refs` | Candidate caps must not be restated; thresholds are required only when performance gates are in scope; `TODO:` threshold values force blocked acceptance | benchmark profile ref, size tier, query class, threshold refs | benchmark row refs, metric checksum refs, validation refs | `GRAPH_QUERY_TIMEOUT`, `GRAPH_BENCHMARK_THRESHOLD_MISSING` |

A package release manifest, provider label, benchmark summary, validation report summary, defaulting decision, or research report must not substitute for any selected row ref or row checksum named in this closure table.

#### RoleRlsSearchPathPreflightRow

`RoleRlsSearchPathPreflightRow` is required for every PostgreSQL-backed profile before query serving, graph apply, rebuild promotion, health success, drift checks, or validation acceptance.

| Field | Required behavior |
| --- | --- |
| `backend_profile_ref` | Structured ref and checksum for the selected backend profile. |
| `runtime_ref` | Exact PostgreSQL runtime distribution ref and checksum. |
| `role_model_ref` | Exact role model ref covering read-only query role and apply writer role. |
| `rls_policy_refs` | Non-empty refs for every RLS policy relied on by graph serving or apply. |
| `search_path_policy_ref` | Exact policy ref; implicit, provider-default, or session-inherited `search_path` is forbidden. |
| `read_only_query_role_ref` | Required for every query-serving path; write privileges fail preflight. |
| `apply_writer_role_ref` | Required for graph apply; must not be reused for read-only query serving. |
| `raw_write_bypass_control_ref` | Required proof that console, admin, import, raw SQL, AGE namespace, or provider-native write bypasses are blocked, detected, or excluded from production. |
| `validation_refs` | Non-empty refs proving safe role, unsafe role, unsafe `search_path`, RLS bypass, and raw-write bypass cases. |
| `package_set_ref` | Required when runtime, role policy, or deployment profile is package-supplied. |
| `activation_scope` | Must cover selected graph serving and apply scope. |
| `lifecycle_status` | Production use requires `active`. |
| `row_checksum` | SHA-256 over canonical row bytes after defaults materialize. |

There are no implicit role, RLS, or `search_path` defaults. Missing or invalid preflight rows emit `GRAPH_POSTGRES_SECURITY_PREFLIGHT_UNSAFE` or the more specific row error.

#### GraphProviderSupportEvidenceRow expansion

`GraphProviderSupportEvidenceRow` must include `provider`, region or redacted region ref, exact engine version ref, service tier, backup/restore support, upgrade support, evidence date, evidence expiry, support status, package-set ref, validation refs, and redaction classes. Expired evidence must behave as missing for preflight, health success, package activation, and promotion.

| Support status | Required behavior |
| --- | --- |
| `supported` | May satisfy provider support only until `evidence_expiry` and only for the exact provider, region or redacted region ref, engine version, service tier, and package set. |
| `preview` | Blocks production unless an explicit production-approval row and validation refs are present. |
| `unsupported` | Blocks activation with `GRAPH_PROVIDER_UNSUPPORTED`. |
| `unknown` | Blocks activation with `GRAPH_PROVIDER_SUPPORT_EVIDENCE_MISSING`. |
| expired evidence | Treated as missing and emits `GRAPH_PROVIDER_SUPPORT_EVIDENCE_EXPIRED`. |

### GraphBackendActivationChecklist

Production use of a graph backend requires every checklist row below to resolve before mutation, query, rebuild promotion, drift check, or graph-serving output.

| Requirement | Required behavior |
| --- | --- |
| provider package refs | Must resolve through `100.PackageReleaseManifest`, exact package type token `graph_backend_provider_package`, and active `ProductionPackageSetManifest`. |
| provider adapter refs | Must resolve through `100`, exact package type token `graph_provider_adapter_package`, and `090.GraphProviderAdapterContract`. |
| driver refs | Must resolve through package trust and compatibility gates using exact package type token `graph_backend_driver_package`. |
| storage backend adapter refs | Must resolve through exact package type token `graph_storage_backend_adapter_package`. |
| index backend adapter refs | Must resolve through exact package type token `graph_index_backend_adapter_package`. |
| runtime distribution refs | Must resolve through exact package type token `graph_backend_runtime_distribution`. |
| deployment profile refs | Must resolve through exact package type token `graph_backend_deployment_profile`; broad `deployment_profile` is not a package type. |
| storage backend | Must be explicitly declared; provider defaults are forbidden. |
| index backend | Must be explicitly declared or unsupported-query behavior must be declared. |
| capability matrix | Must declare supported, unsupported, partial, and fail-closed query/apply behavior. |
| schema profile | Must validate before mutation or query. |
| raw-write bypass policy | Must be explicit and fail-closed. |
| PostgreSQL provider support evidence | Required for the selected provider, region, engine version, service tier, extension support, backup/restore support, and upgrade path when those can affect activation. |
| relational schema fingerprint | Required for `mvp-postgresql-relational-graph.v1` and `mvp-postgresql-age.v1`. |
| role, RLS, and `search_path` preflight | Required for PostgreSQL-backed profiles. |
| AGE extension preflight | Required only when `mvp-postgresql-age.v1` is selected. |
| validation refs | Must include `120-GRAPH-POSTGRES-*` for the default relational profile, `120-GRAPH-AGE-*` when AGE is selected, and backend package gates. |
| lifecycle status | Production use requires `active`. |

Each referenced graph backend release must resolve exactly one active `100.PackageTypePolicyRow`, must be included in the active `100.ProductionPackageSetManifest`, and must appear in `030.VersionManifest.included_refs`. Graph backend preflight must reject broad labels, provider-specific package names, mutable refs, and deployment profile aliases before graph mutation, query serving, rebuild promotion, or drift check.

### MVP PostgreSQL relational default backend profile

The default profile row is a selection default, not a domain decision. It must not become production-active while any `TODO:` value, missing ref, failed validation row, or package-set gate remains unresolved. Selection-only behavior must emit a `GraphBackendSelectionDefaultDecision` and no graph mutation, query, rebuild promotion, drift check, or serving output.

| Field | Required MVP value or rule |
| --- | --- |
| `backend_profile_id` | `mvp-postgresql-relational-graph.v1` |
| `provider` | `postgresql` |
| `query_language` | `sql` |
| `storage_backend_kind` | `postgresql_relational_tables` |
| `index_backend_kind` | `postgresql_indexes` |
| `provider_adapter_ref` | PostgreSQL graph adapter package ref through `100`. |
| `driver_ref` | PostgreSQL driver package ref through `100`. |
| `schema_policy_ref` | `mvp-postgresql-relational-schema.v1` `GraphReadModelSchemaProfile`. |
| `taxonomy_mapping_ref` | PostgreSQL relational `GraphBackendTaxonomyMappingProfile`. |
| `query_translation_profile_ref` | PostgreSQL SQL `GraphQueryTranslationProfile`. |
| `apply_profile_ref` | PostgreSQL relational `GraphApplyProfile`. |
| `capability_matrix_ref` | PostgreSQL relational capability matrix. |
| `raw_write_bypass_policy_ref` | SQL DML/DDL bypass prevention proof. |
| `package_set_manifest_ref` | Active `100.ProductionPackageSetManifest`. |
| `validation_refs` | Non-empty `120-GRAPH-POSTGRES-*` refs. |

### PostgreSQL relational anchor schema contract

`mvp-postgresql-relational-schema.v1` is the required schema profile for the PostgreSQL relational default and for any AGE profile. Relational anchors preserve Cadastre-owned graph identity; they do not make PostgreSQL row identity, tuple identity, sequence identity, OID identity, or AGE identity authoritative.

| Required relation | Required purpose | Required constraints |
| --- | --- | --- |
| `graph_node` | One row per Cadastre graph node ID. | Primary key or unique constraint on `graph_node_id`; canonical source object ref fields; assertion, temporal, confidence, and derived-view fields constrained by Cadastre vocabularies. |
| `graph_edge` | One row per Cadastre graph edge ID. | Primary key or unique constraint on `graph_edge_id`; foreign keys from endpoint node IDs to `graph_node.graph_node_id`; edge type constrained to `observed_connection` for MVP; traversal class constrained by `GraphTraversalClass`; valid and known interval order checks. |
| `graph_edge_evidence_ref` | Bridge from graph edge ID to evidence refs. | Foreign key to `graph_edge`; uniqueness on `(graph_edge_id, evidence_ref_id)`. |
| `graph_apply_state` | Persisted apply batch, idempotency, partial-apply, resume, and derived-view advancement state. | Idempotency key uniqueness; batch status closed enum; no derived-view advancement unless read-after-write and schema fingerprint checks pass. |
| `graph_schema_fingerprint_state` | Persisted schema inventory and checksum evidence. | Exactly one current fingerprint per active backend profile and deployment scope; stale or mismatched fingerprints block apply, query, rebuild promotion, and replay. |
| bounded property storage | Optional declared JSONB or `graph_node_property` representation. | JSONB is allowed only for declared, typed, bounded property paths; undeclared paths fail before persistence or query. |

`BackendSchemaFingerprint` for PostgreSQL-backed profiles must include tables, columns, constraints, indexes, triggers, RLS policies, functions, extension versions when present, package refs, schema profile refs, and checksums. The fingerprint must be computed before graph apply, graph query serving, rebuild promotion, restore acceptance, upgrade acceptance, and replay. Duplicate node IDs, duplicate edge IDs, orphan edges, invalid edge types, invalid traversal classes, invalid intervals, stale fingerprints, and undeclared JSONB paths must fail before backend mutation or caller-visible graph output.

#### PostgreSQLRelationalAnchorSchemaProfile

`PostgreSQLRelationalAnchorSchemaProfile` is the contract-visible schema object set for PostgreSQL-backed graph serving. The profile must define `graph_node`, `graph_edge`, `graph_edge_evidence_ref`, `graph_apply_state`, `graph_schema_fingerprint_state`, and either declared JSONB property paths or a declared bounded property table. Application-owned graph IDs are the only graph IDs that may appear in responses, selectors, cursors, page tokens, drillback refs, replay refs, evidence refs, audit output, telemetry, or validation output.

| Schema object | Required closure |
| --- | --- |
| `graph_node` | Stores one row per application-owned graph node ID. Backend-generated IDs, row IDs, OIDs, tuple IDs, and sequence values are forbidden as graph identity. |
| `graph_edge` | Stores one row per application-owned graph edge ID. `edge_type` must be exactly `observed_connection` for MVP. Endpoint refs must target `graph_node.graph_node_id`. |
| `graph_edge_evidence_ref` | Stores a bounded evidence bridge. It must use evidence refs, not backend row locators. |
| `graph_apply_state` | Stores batch, idempotency, partial-apply, resume, lock-loss, and derived-view advancement evidence. |
| `graph_schema_fingerprint_state` | Stores current and prior schema fingerprint refs for restore, upgrade, replay, query, apply, and rebuild gates. |
| declared JSONB paths | Optional. Every JSONB path must be declared, typed, bounded, indexed when queried, and covered by validation. Undeclared paths fail before persistence or query. |
| declared property table | Optional alternative to JSONB. It must have bounded property names, typed values, and duplicate rejection. |

### PostgreSQL SQL query translation matrix

PostgreSQL SQL translation is allowed only as backend candidate materialization. Cadastre ordering, authorization, redaction, page-token identity, stale-read handling, and error selection occur after candidate materialization. SQL row order, SQL cursor names, prepared statement names, physical row locators, tuple IDs, OIDs, sequence values, and provider-native query handles must not become response order, response identity, drillback identity, replay identity, or page-token identity.

| Query class | PostgreSQL relational materialization boundary | Required indexes | Bounds and overflow behavior | Timeout and cancellation | Required post-processing | Unsupported or unsafe behavior |
| --- | --- | --- | --- | --- | --- | --- |
| `node_detail` | Lookup by Cadastre `graph_node_id` and return application-owned IDs only. | Node primary key and canonical ref indexes. | Candidate cap `1`; duplicate candidates fail. | Request timeout maps to statement timeout plus application cancellation. | Authorization, redaction, deterministic output, and backend-ID rejection. | Duplicate node emits `GRAPH_POSTGRES_DUPLICATE_NODE_ID`; backend ID leakage emits `GRAPH_BACKEND_ID_FORBIDDEN`. |
| `neighbor_expansion` | Query outbound and inbound adjacency from `graph_edge` filtered by edge type, traversal class, assertion state, confidence, valid time, and known time. | Endpoint adjacency indexes, edge type/traversal indexes, temporal/filter indexes. | Candidate limit from `GraphQueryCandidateLimitTable`; overflow emits `GRAPH_QUERY_CANDIDATE_LIMIT_REACHED`. | Statement timeout plus application cancellation. | Cadastre ordering and page-token generation after redaction. | Unsupported predicate or full scan emits `GRAPH_POSTGRES_QUERY_PLAN_UNSUPPORTED`. |
| `bounded_path` | Recursive candidate traversal over adjacency edges with max depth `1..6`, cycle guard, self-loop handling, traversal-class filter, and candidate cap. | Adjacency indexes plus partial indexes for active traversal/assertion states. | Depth outside `1..6` fails before SQL; candidate overflow fails closed. | Statement timeout plus application cancellation. | Cadastre path ordering, authorization, redaction, page token. | Theoretical reachability remains prohibited; empty traversal class returns empty output without backend traversal. |
| `evidence_drillback` | Join node or edge anchor rows to source gold fact and evidence refs. | Evidence-ref bridge index and source object refs. | Page size plus one; hard cap from `110`. | API timeout. | Evidence metadata only unless raw permission allows more. | Backend IDs, SQL cursors, and physical row locators are invalid drillback refs. |
| `analysis_read_only` | Execute only validated read-only SQL templates mapped through `GraphQueryTranslationProfile` and `130.RuleGraphCompatibilityMatrix`. | Same as query class. | Per rule profile; `TODO:` concrete analysis caps block production promotion when performance gates are in scope. | Statement timeout plus application cancellation. | Expected query checksum, expected result checksum, authorization, redaction, mutation proof. | Raw SQL text and provider-native query handles are forbidden unless validation import maps them into an active profile. |

Bounded path validation must cover depth `1..6`, cycles, self-loops, empty traversal classes, high-degree candidate overflow, timeout, cancellation, page-token mismatch, and proof that SQL natural order is never response order.

#### GraphQueryTranslationProfileRow closure

Each PostgreSQL translation row must include query class, selected backend profile ref, expected translated query checksum, expected result checksum, authorization refs, redaction refs, timeout policy ref, ordering policy ref, page-token policy ref, mutation-prohibition proof, validation refs, package-set refs when package-supplied, and manifest refs. Translation rows must cover exactly `node_detail`, `neighbor_expansion`, `bounded_path`, `evidence_drillback`, and `analysis_read_only`.

| Forbidden backend output identity | Required behavior |
| --- | --- |
| Raw SQL text | Must not appear in API, audit, telemetry, page tokens, validation output, or evidence refs. |
| Prepared statement names | Rejected as response identity and page-token identity. |
| SQL cursor names | Rejected as page-token identity and query handles. |
| PostgreSQL OIDs | Rejected before caller-visible output. |
| Tuple IDs | Rejected before caller-visible output. |
| Sequence values | Rejected before caller-visible output. |
| Physical row locators | Rejected before drillback or replay identity. |
| Literal-bearing query plans | Rejected or redacted before telemetry, audit, and validation output. |
| Backend query handles | Rejected before API, page-token, audit, telemetry, evidence, and replay output. |

#### GraphQueryCandidateLimitTable handoff

`GraphQueryCandidateLimitTable` owns backend candidate caps only. API bounds, page-size bounds, and timeout bounds remain imported from `110` by exact contract name and must not be restated by limit rows.

| Query class | Candidate cap source | Overflow error | Post-processing owner | Timeout and cancellation behavior |
| --- | --- | --- | --- | --- |
| `node_detail` | cap fixed at `1` candidate per application-owned graph node ID. | `GRAPH_POSTGRES_DUPLICATE_NODE_ID` for more than one candidate. | `090.QueryGraph` plus `110` response rendering. | Statement timeout plus application cancellation; no partial object. |
| `neighbor_expansion` | selected candidate-limit row. | `GRAPH_QUERY_CANDIDATE_LIMIT_REACHED`. | `090` ordering, authorization, redaction, and pagination. | Statement timeout plus application cancellation before Cadastre page token creation. |
| `bounded_path` | selected candidate-limit row, depth `1..6`. | `GRAPH_QUERY_CANDIDATE_LIMIT_REACHED`. | `090` path ordering and cycle/self-loop handling. | Timeout or cancellation emits no partial mutation and no backend cursor. |
| `evidence_drillback` | `110` page-size plus one and selected drillback cap. | `GRAPH_QUERY_CANDIDATE_LIMIT_REACHED` when cap exceeded. | `110` evidence drillback rendering and redaction. | Timeout emits owner error before raw payload lookup. |
| `analysis_read_only` | selected rule compatibility row. | `GRAPH_QUERY_CANDIDATE_LIMIT_REACHED` or analysis owner error. | `130.RuleGraphCompatibilityMatrix` and `090` translation. | Timeout emits owner error and no analysis output. |

### PostgreSQL relational apply, rebuild, restore, and migration contract

`GraphApplyProfile` for PostgreSQL relational profiles must define deterministic transaction and resume behavior before graph apply can execute. The PostgreSQL relational default imports the `GraphApplyProfile defaults` in this file and narrows them as follows.

| Apply field | Required PostgreSQL relational value |
| --- | --- |
| `max_batch_deltas` | Default `1000`; minimum `1`; maximum `5000`. |
| `batch_membership` | Deterministic from `GraphDeltaApplyCanonicalOrder` and the declared batch size. Backend natural order is forbidden. |
| `transaction_boundary` | Exactly one declared batch per transaction boundary. Smaller chunks are allowed only when the selected row declares a deterministic split key and includes split validation refs. |
| `retry_policy` | Default `disabled`. No implicit driver retry class may affect output. Every enabled retry class must be declared, bounded, idempotency-proven, and validation-backed. |
| `read_after_write_proof_required` | `true`. `DerivedViewState` must not advance without persisted read-after-write proof. |
| `stale_schema_behavior` | Reject before mutation when `BackendSchemaFingerprint` is missing, stale, mismatched, or unmanifested. |
| `resume_policy` | Resume only through `ResumeGraphApply`; no invisible replay from PostgreSQL state is allowed. |
| `lock_loss_behavior` | Stop writes when possible and emit no derived-view advancement after lock loss. |
| `duplicate_collision_behavior` | Duplicate graph node or edge IDs fail with PostgreSQL-specific owner errors before derived-view advancement. |

`RebuildGraph` must rebuild from authoritative Cadastre graph deltas and owner records, not from PostgreSQL graph state alone. Restore acceptance requires a clean-environment restore proof, schema fingerprint recomputation, graph index consistency check, graph query parity, and derived-view state validation. Schema migration requires fingerprint diff evidence, rollback or rebuild migration plan, and PostgreSQL major-version rehearsal evidence before promotion. Exact PostgreSQL version, patch version, driver package, deployment provider/scope, benchmark thresholds, migration/rollback strategy, provider support evidence, restore evidence, and upgrade evidence remain production blockers until supplied by selected activation-controlled rows and `120` validation artifacts.

#### GraphRestoreRehearsalEvidenceRow schema

`GraphRestoreRehearsalEvidenceRow` must require `backend_profile_ref`, `clean_environment_restore_ref`, `restored_fingerprint_ref`, `index_consistency_check_ref`, `query_parity_ref`, `derived_view_validation_ref`, `package_set_ref`, and `result`. `result` must be `pass` before production promotion. Missing, stale, failed, checksum-mismatched, package-set-mismatched, or unmanifested rows emit `GRAPH_BACKEND_RESTORE_UNVERIFIED` and block production graph serving promotion.

#### GraphUpgradeRehearsalEvidenceRow schema

`GraphUpgradeRehearsalEvidenceRow` must require `backend_profile_ref`, `from_runtime_ref`, `to_runtime_ref`, `schema_migration_diff_ref`, `rollback_or_rebuild_plan_ref`, `major_version_rehearsal_ref`, `package_compatibility_refs`, `package_set_ref`, and `result`. `result` must be `pass` before production promotion. Missing, stale, failed, checksum-mismatched, package-set-mismatched, or unmanifested rows emit `GRAPH_BACKEND_UPGRADE_UNVERIFIED`.

#### GraphBenchmarkThresholdProfile closure

At least `medium_mvp` and `large_stress` benchmark threshold rows are required when backend performance gates are in promotion scope. Each row must include graph size tier, degree distribution ref, run count, warmup rule, p50, p95, and p99 threshold refs, timeout expectation, overflow expectation, WAL, index-build, and bloat metric refs when in scope, metric checksum refs, validation refs, package-set ref when supplied, and manifest refs.

| Benchmark row | Required behavior |
| --- | --- |
| `medium_mvp` | Must cover MVP query classes, normal degree distribution, expected timeout behavior, expected overflow behavior, and metric checksum refs. |
| `large_stress` | Must cover high-degree overflow, bounded path depth `1..6`, restore and upgrade stress when in scope, index build metrics, WAL and bloat refs when in scope, and cancellation behavior. |

TODO: Product governance must supply concrete benchmark thresholds, workload fixture checksums, expected metric checksums, package-set refs when package-supplied, and validation refs. Until supplied, benchmark closure returns `blocked`, not `pass`, and production promotion fails with `GRAPH_BENCHMARK_THRESHOLD_MISSING`.

### Apache AGE optional graph extension profile

`mvp-postgresql-age.v1` is optional and validation-only by default. It may not be selected by omitted backend configuration. It may become production-eligible only when exact PostgreSQL, AGE, provider, package, extension, role, namespace, restore, upgrade, query parity, and benchmark gates pass.

| Field or gate | Required behavior |
| --- | --- |
| `backend_profile_id` | `mvp-postgresql-age.v1`. |
| `provider` | `postgresql_age`. |
| `query_language` | `sql_cypher`. |
| relational anchors | Required; AGE graph objects must mirror or candidate-materialize from relational anchors. |
| materialization mode | Must declare AGE mirror mode or AGE candidate materialization mode. |
| AGE mutation | Forbidden by default in query serving; production mutation requires explicit owner templates and validation gates. |
| AGE internal IDs | AGE vertex, edge, path, graph, label, and agtype object IDs are discarded or rejected and must not escape. |
| Cypher invocation | `ag_catalog.cypher` must be fully qualified; unsafe `search_path` fails preflight. |
| namespace bypass | Direct DML/DDL against AGE graph namespace fails security preflight and negative fixtures. |
| read-only query role | Required for query serving; mutating Cypher clauses fail before execution. |
| provider support evidence | Must name provider, region, engine version, service tier, AGE version, preview/GA status, extension files/control scripts, and upgrade path. |
| production status | `validation_only` or `blocked_until_gates_pass` until all AGE gates pass. |

AGE validation fixtures must prove extension missing/wrong version blocks activation, unsupported provider blocks activation, read-only Cypher mutation fails, namespace DML/DDL fails, AGE internal IDs do not escape, query parity passes, and restore/upgrade rehearsals pass or block production.

AGE production eligibility also requires exact PostgreSQL version refs, exact AGE extension version and package refs, extension files and control script refs, provider support evidence, namespace and security preflight refs, mutating-Cypher rejection proofs, AGE internal-ID rejection proofs, query parity refs, restore refs, upgrade refs, benchmark refs, telemetry redaction refs, package refs, validation refs, and manifest refs. A missing row blocks AGE production with the most specific AGE, provider, package, restore, upgrade, telemetry, or manifest error. Documentation, release notes, provider marketing pages, or research reports must not substitute for these rows.

#### AGE production-eligibility row precision

AGE remains explicit-selection and validation-only by default. If AGE production activation is requested, the selected AGE row set must include the following closed preflight rows before query serving, graph mutation, rebuild promotion, health success, or validation acceptance.

| AGE row | Required behavior | Required rejection |
| --- | --- | --- |
| extension version row | Names exact AGE extension version, supported PostgreSQL major version, extension package ref, extension control file refs, extension SQL/script refs, package-set ref, and validation refs. | Missing, unsupported, stale, package-set-mismatched, or unmanifested rows emit `GRAPH_AGE_EXTENSION_MISSING` or `GRAPH_AGE_EXTENSION_VERSION_UNSUPPORTED`. |
| provider support row | Names provider, region, engine version, service tier, support status, backup/restore support, upgrade path, evidence date, evidence expiry, package-set ref, validation refs, and redaction behavior. | Missing, expired, preview-without-production-approval, unsupported, or private-leaking evidence emits `GRAPH_PROVIDER_SUPPORT_EVIDENCE_MISSING` or `GRAPH_PROVIDER_UNSUPPORTED`. |
| SQL/Cypher qualification row | Requires `ag_catalog.cypher` qualification and forbids unsafe `search_path` resolution. | Unsafe `search_path`, unqualified Cypher execution, or provider-native query handles emit `GRAPH_BACKEND_UNSAFE_SEARCH_PATH` or `GRAPH_QUERY_PLAN_UNSUPPORTED`. |
| AGE mutation prohibition row | Proves mutating Cypher clauses, graph drop, label mutation, namespace DML, and namespace DDL fail with no mutation. | Mutating Cypher emits `GRAPH_AGE_MUTATION_FORBIDDEN`; namespace bypass emits `GRAPH_AGE_NAMESPACE_BYPASS_FORBIDDEN`. |
| AGE ID rejection row | Rejects AGE vertex, edge, path, graph, label, and agtype object IDs before API, page-token, audit, telemetry, validation, evidence, or replay visibility. | Any AGE ID leak emits `GRAPH_AGE_INTERNAL_ID_FORBIDDEN`. |
| AGE parity row | Proves AGE candidate materialization has provider-neutral parity with PostgreSQL relational query classes and relational anchors. | Missing parity emits `GRAPH_QUERY_PLAN_UNSUPPORTED` and blocks AGE promotion. |
| AGE restore/upgrade row | Includes clean-environment restore with extension files/control scripts, restored schema fingerprint, AGE namespace inventory, upgrade rehearsal or rebuild migration proof, and rollback evidence. | Missing proof emits `GRAPH_BACKEND_RESTORE_UNVERIFIED` or `GRAPH_BACKEND_UPGRADE_UNVERIFIED`. |

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

### MVPActiveGraphProjectionProfile

The only active MVP graph projection profile is `mvp-observed-flow-graph.v1`. A production graph projection profile with any other profile ID must remain inactive unless a later authoritative spec-set version adds its profile row, semantics rows, output eligibility rows, query translation rows, validation rows, and promotion refs.

| Profile property | Required MVP value |
| --- | --- |
| `profile_id` | `mvp-observed-flow-graph.v1` |
| Active edge type set | Exactly `observed_connection`. |
| Active node output set | Exactly `projected_canonical_node`. |
| Source fact family | Authorized `GoldFact` records with fact type `observed_network_flow_fact`, an active `080.GoldFactPredicateContractRow`, permitted subject/object one-of kinds, and qualifying flow-role evidence. |
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
| `required_gold_fact_predicate_contract_ref` | Yes | none | Exact active `080.GoldFactPredicateContractRow` ref for the consumed fact family. |
| `allowed_subject_ref_kinds` | Yes | none | Non-empty subset of the selected predicate contract and `040.GoldFactSubjectRefKindRegistry`. |
| `allowed_object_value_kinds` | Yes | none | Non-empty subset of the selected predicate contract and `040.GoldFactObjectValueKindRegistry`. |
| `endpoint_ref_resolution_policy` | Yes | `canonical_entity_required` | Closed enum: `canonical_entity_required`, `resolver_handoff_required`, `not_endpoint_identity`. MVP observed-connection endpoints require canonical entity refs before edge output. |
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
| `activation_scope` | Yes | none | `030.ActivationScope` for the graph output scope. |
| `lifecycle_status` | Yes | none | Production use requires `active`. |

### Closed GoldFact consumption for graph projection

`ProjectGraphDeltas` may consume only `GoldFact` records that pass `040.GoldFactSchema`, resolve one active `080.GoldFactPredicateContractRow`, and pass exact `060` source authority for the fact. Graph projection must not reinterpret `string_value`, OCSF endpoint objects, source-native graph payloads, backend-native IDs, package artifact IDs, telemetry IDs, or selector-only values as endpoint identity.

MVP `observed_connection` projection must require `required_gold_fact_predicate_contract_ref` for the source fact family. Endpoint identities must resolve to `canonical_entity_ref` before emitting `projected_canonical_node`. If the source fact uses `source_asset_ref` or `identifier_ref`, projection may continue only after `070` supplies a qualifying identity decision that resolves the endpoint to a canonical entity. `string_value` must never be treated as endpoint identity.

Backend-native IDs remain forbidden in graph IDs, selectors, evidence refs, page tokens, query cursors, and drillback keys. Missing or ambiguous `FlowRoleEvidence`, unresolved endpoint identity, and invalid subject/object kind must produce no edge and the most specific graph, identity, predicate-contract, or core schema error.

### MVPGraphNodeProjectionRows

| Node projection row | Input | Required identity | Emitted object | Default when unavailable | Validation rows |
| --- | --- | --- | --- | --- | --- |
| `mvp-project-canonical-node-v1` | Resolved endpoint canonical entity ref from `070` output | Qualifying `IdentityDecision` and `CanonicalEntity` ref | `projected_canonical_node` | Emit no node and block dependent edge with `GRAPH_ENDPOINT_IDENTITY_UNRESOLVED`. | `val-090-unresolved-endpoint-no-edge`, `val-070-graph-endpoint-requires-identity-decision` |
| `mvp-structural-global-node-disabled-v1` | Any structural global request | n/a | none | Deterministic no-op; structural global output remains inactive. | `val-090-mvp-active-profile-exact-edge-set` |
| `mvp-generic-external-payload-disabled-v1` | Source-native graph payload or graph key | n/a | none | Deterministic no-op; payload may be retained only outside pathfinding. | `val-090-generic-external-payload-not-pathfinding` |

### MVPGraphEdgeProjectionRows

| Edge projection row | Required inputs | Direction rule | Edge ID inputs | Confidence | Temporal fields | Evidence refs | Traversal class | Non-implication | No-op or error behavior |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `mvp-observed-connection-edge-v1` | `GoldFact` fact type `observed_network_flow_fact`; `required_gold_fact_predicate_contract_ref = gfp-mvp-flow-observed-connection-v1`; allowed subject/object kinds; two resolved endpoint canonical entity refs; qualifying `FlowRoleEvidence`; source `GoldFact` ref | `mvp-flow-role-directed-v1`; source and target endpoint are selected only from qualifying flow-role evidence and resolved canonical entity refs | projection profile ID, predicate contract ref, source gold fact key/ref, from canonical entity ref, to canonical entity ref, direction rule ref, valid interval, known interval, edge type | Import from source fact and serialize through `040.DecimalPrecisionPolicy.confidence_0_1`. | Import from source fact or correction handoff under `080` policies. | Source `GoldFact`, predicate contract, silver observation, raw metadata, temporal resolution, authority refs, identity refs, and flow-role evidence refs. | `observed_connection_path` | Observed traffic only; not theoretical reachability, service access, policy allow, compromise, exploitability, lateral movement, or identity access. | Missing or invalid predicate contract, string endpoint identity, missing flow-role evidence, ambiguous flow-role evidence, unresolved endpoint, backend ID, or OCSF endpoint order emits no edge. |

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

### StructuredInputRepositoryGraphProfileHandoff

Repository-authored graph projection profiles, edge semantics row sets, traversal class rows, output eligibility rows, backend profiles, provider capability matrices, schema profiles, taxonomy mappings, query translation profiles, derived-view lag policies, graph apply profiles, and graph property policies are inert until materialized, exact-snapshot validated, package-set activated when package-supplied, and included in `030.VersionManifest`.

A Git merge, pull request approval, branch update, validation run, or hook success must not enable graph serving, graph apply, query translation, backend selection, schema creation, graph rebuild promotion, derived-view advancement, or graph mutation.

Graph backend selection defaults must not be derived from repository filenames, branch names, package names, provider defaults, path prefixes, or repository URLs. Defaults must resolve through `GraphBackendSelectionPolicy` and the package/materialization gates named by `030` and `100`.

`GraphBackendPreflightResult` must include repository snapshot refs, file manifest checksums, validation refs, materialization refs, package release refs when created, package-set refs when package-supplied, and `VersionManifest` refs when the active graph backend, query profile, schema profile, taxonomy mapping, capability matrix, edge semantics, or projection profile was repository-authored.

`GraphBackendPreflightResult` for repository-authored graph artifacts must also include `030.StructuredInputRepositoryTemplateContract` refs when required, `030.StructuredInputRepositoryCIContract` validation refs when accepted, maintenance tool contract and invocation refs when graph profiles or query translations are generated, `100.StructuredInputPublicationManifest` refs when remote publication is consumed, `030.StructuredInputCandidateSyncRecord` refs when manifest sync imported the candidate, and `030.StructuredInputRepositoryGroup` refs when graph artifacts depend on resolver, source-authority, mapping, or registry artifacts from another repository.

Template status or sync status must not enable graph serving. Generated provider-native query text remains validation-only unless mapped through an active `GraphQueryTranslationProfile` and read-only validation passes. Publication manifest mismatch blocks graph profile activation and graph backend preflight. Multi-repository group mismatch blocks package-set activation for graph-affecting artifacts.

Provider-native query text committed in Git remains validation-only unless mapped through an active `GraphQueryTranslationProfile`. A committed Cypher, AQL, SQL, or provider query string must not bypass query translation, authorization, redaction, deterministic ordering, page-token rules, or mutation prohibition.

| Error code | Required use |
| --- | --- |
| `GRAPH_REPOSITORY_PROFILE_INACTIVE` | Repository-authored graph profile exists but lacks active materialized refs, package-set refs when package-supplied, or manifest refs. |
| `GRAPH_REPOSITORY_VALIDATION_STALE` | Graph validation or preflight does not match exact repository snapshot, file manifest checksum, graph artifact checksum, backend profile checksum, or package-set ref. |
| `GRAPH_REPOSITORY_QUERY_TEXT_FORBIDDEN` | Provider-native query text from repository content attempts to execute without active translation profile and read-only validation. |
| `GRAPH_REPOSITORY_TEMPLATE_MISMATCH` | Repository-authored graph artifact layout, generated output roots, or artifact classes do not match the selected template contract. |
| `GRAPH_REPOSITORY_CI_STALE` | Producer CI evidence for graph profile, query translation, backend matrix, or edge semantics is stale or not exact-snapshot-bound. |
| `GRAPH_REPOSITORY_PUBLICATION_MANIFEST_MISMATCH` | Publication manifest does not match graph artifact digest, size, media type, package type, materialization refs, validation refs, or package release refs. |
| `GRAPH_REPOSITORY_SYNC_NONAUTHORITY` | Candidate sync record is used as graph activation, backend default selection, query serving, rebuild promotion, drift repair, or graph mutation authority. |
| `GRAPH_REPOSITORY_GROUP_MISMATCH` | Required repository group coherence for graph-affecting artifacts fails. |

## Graph Delta Identity

Every graph node and edge must have a Cadastre-owned deterministic ID. Backend-generated node, edge, relationship, vertex, document, element, transaction, shard, or native cursor IDs are forbidden as Cadastre IDs, selectors, evidence refs, replay keys, drillback keys, response IDs, or pagination identity and must fail with `GRAPH_BACKEND_ID_FORBIDDEN` before graph apply, query response, evidence ref generation, replay, or pagination.

## Edge Semantics and Traversal

Every active graph edge type must have exactly one `GraphEdgeSemantics` row. The row must define source fact type, source predicate, direction rule, evidence requirements, allowed assertion states, temporal policy, confidence policy, traversal class, non-implication rules, and no-op conditions.

`GraphTraversalClass` is the only traversal eligibility authority. Parallel fields such as `traversal_eligibility`, `reverse_traversal_eligibility`, or `pathfinding_role` are forbidden.

OCSF `src_endpoint`, `dst_endpoint`, `network_endpoint`, DNS endpoint, DHCP endpoint, or any external schema endpoint order is not `FlowRoleEvidence`. A projection row that has OCSF endpoint fields but lacks qualifying `FlowRoleEvidence` must emit no `observed_connection` edge and must emit `GRAPH_FLOW_ROLE_EVIDENCE_REQUIRED` when the projection attempt is observable.

OCSF direction rejection must be validated against the active `050.ObservationToOCSFMappingRowSet` for every active `network_activity_observation`, DNS, and DHCP row family that emits endpoint-like OCSF fields. A mapping fixture that emits OCSF endpoint fields and no qualifying `FlowRoleEvidence` must produce no `observed_connection` edge. The graph validation row must reference the selected mapping row, source silver observation checksum, graph projection profile, graph edge semantics row, and `VersionManifest`.

### OCSFDirectionMappingFixtureHandoff

Every endpoint-order rejection validation row must prove direction rejection from an active mapping output, not from graph-only synthetic input.

| Required fixture member | Required behavior |
| --- | --- |
| selected mapping row | Exact `050.ObservationToOCSFMappingRow` ref/checksum for `network_activity_observation`, `dns_observation`, or DHCP-discriminated `dhcp_ipam_observation` that emits endpoint-like OCSF fields. |
| source silver observation | Source `CadastreSilverObservation` checksum and `VersionManifest` ref. |
| graph profile refs | Active `GraphProjectionProfile` ref, graph edge semantics row ref, and graph object output eligibility row ref when the attempted output would be search-, neighbor-, finding-, metric-, or pathfinding-visible. |
| expected output | Expected no-edge checksum or expected owner error checksum. |
| mutation prohibition | Proof that no `observed_connection` edge, graph property, pathfinding input, graph delta, graph apply write, or backend mutation occurred. |
| package refs | Package release and production package-set refs when any mapping, graph, or validation artifact is package-supplied. |
| manifest refs | `030.VersionManifest` ref containing selected mapping row refs/checksums, source silver checksum, graph profile refs/checksums, validation refs, and package-set refs when supplied. |

The direction rejection scope includes active `network_activity_observation`, DNS, and DHCP rows that emit endpoint-like OCSF fields. `cadastre_only` rows are graph no-op by default unless they emit qualifying Cadastre-owned `FlowRoleEvidence` accepted by the active graph profile. OCSF `src_endpoint`, `dst_endpoint`, DNS endpoint, DHCP endpoint, and network endpoint ordering must not determine graph direction.

## Backend Preflight

A `GraphBackendProfile` must be active before graph mutation, query, rebuild import, drift check, or graph-serving promotion. `GraphBackendPreflightResult` must validate backend, driver, dialect, topology, storage mode, feature availability, raw-write bypass controls, schema profile, and query translation readiness.

## ProjectGraphDeltas Algorithm

```text
ProjectGraphDeltas(change_set, projection_profile):
1. Validate `GoldFactChangeSet`, graph handoff effect, projection profile, graph semantics rows, and output eligibility rows.
2. If handoff effect is `none`, emit no `GraphDeltaSet` and record a no-op projection diagnostic.
3. If handoff effect is `reproject_fact_key`, recompute projected objects for the affected fact key from authoritative gold inputs and active projection rows.
4. If handoff effect is `expire_projected_object`, require `060.AbsenceDerivationResult.allowed_effects` containing `graph_expiry`, requested effect equality, manifest-complete closure refs, and `closure_outcome = closed` before emitting expiry delta; deterministic block and blocked-validation outcomes emit no graph delta.
5. If handoff effect is `cleanup_projected_object`, require `060.AbsenceDerivationResult.allowed_effects` containing `cleanup`, requested effect equality, manifest-complete closure refs, and `closure_outcome = closed` before emitting cleanup delta; deterministic block and blocked-validation outcomes emit no graph delta.
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
8a. Resolve `030.RunLockCommitGuard` for target graph scope, graph delta set checksum, apply profile, backend profile, and requested output class.
8b. Reject before backend execution when the guard is missing, stale, fenced, lock-lost, or outside scope.
8c. Include the guard ref in `GraphApplyResult` and `VersionManifest`.
9. Apply batches only at declared transaction boundaries.
10. Persist backend evidence rows for transaction semantics, failover, read-after-write, storage mode, index freshness, writer identity, and run-lock commit guard when correctness-affecting.
11. On failure, record committed batch IDs or prove no committed writes.
12. Return GraphApplyResult with status, errors, input checksum, backend evidence, idempotency state, artifact refs, run-lock commit guard refs, and derived view state update eligibility.
```

### RunLockGraphApplyValidationHandoff

`val-090-runlock-graph-apply-resume` must prove all graph-apply run-lock conditions below. The row validates graph apply consequences only and must not define provider-specific implementation mechanics.

| Scenario | Required observable behavior |
| --- | --- |
| missing guard | Reject before backend execution. |
| stale guard | Reject before backend execution. |
| fenced guard | Reject before backend execution. |
| lock-lost guard | Reject before backend execution. |
| out-of-scope guard | Reject before backend execution. |
| partial apply under lock loss | Do not advance `DerivedViewState`. |
| resume after partial apply | Require committed-batch proof. |
| resume idempotency | Use the same `GraphDeltaIdempotencyKey`. |
| resumed guard | Require a new valid `030.RunLockCommitGuard`. |
| result evidence | `GraphApplyResult` includes guard refs and derived-view state update eligibility. |

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
run_lock_asserted
run_lock_lost
run_lock_guard_stale
replay_same_event
```

| Prior state | Event | Next state | Required behavior |
| --- | --- | --- | --- |
| `prepared` | `preflight_passed` | `preflighted` | Persist preflight evidence. |
| `prepared` | `preflight_failed` | `failed_no_commit` | No backend writes. |
| `preflighted` | `run_lock_asserted` | `preflighted` | Same-state evidence; apply may start. |
| `preflighted` | `start_apply` | `applying` | Persist apply-start evidence. |
| `applying` | `batch_committed` | `applying` or `partial_committed` | Persist committed-batch evidence. |
| `applying` | `complete` | `applied` | Persist `GraphApplyResult`; derived-view update eligible. |
| `applying` | `batch_failed_before_commit` | `failed_no_commit` | No derived-view advancement. |
| `applying` | `batch_failed_after_commit` | `failed_partial` | Persist committed-batch proof; no derived-view advancement. |
| `preflighted` or `applying` | `run_lock_guard_stale` | `failed_no_commit` or `failed_partial` | No derived-view advancement; partial only when committed-batch evidence exists. |
| `applying` | `run_lock_lost` | `failed_partial` or `failed_no_commit` | Persist committed-batch proof or no-commit proof. |
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
| `observed_connection` | `observed_network_flow_fact` | `observed_connection`; `required_gold_fact_predicate_contract_ref = gfp-mvp-flow-observed-connection-v1` | `FlowRoleEvidence` only; OCSF endpoint order and backend edge convention are not qualifying evidence | gold, silver, raw, temporal, authority, and flow-role refs | `active`; `stale` only when `060` and eligibility rows permit stale display | `080` | Import from source fact and serialize through `040.DecimalPrecisionPolicy.confidence_0_1` | `observed_connection_path` | Observed traffic only; no theoretical reachability, service access, policy allow, compromise, exploitability, lateral movement, or identity-access implication. | No edge when `FlowRoleEvidence` is missing or ambiguous; unresolved endpoint blocks with `GRAPH_ENDPOINT_IDENTITY_UNRESOLVED`; OCSF endpoint order emits no direction. | `val-090-observed-connection-positive`, `val-090-observed-connection-missing-flow-role`, `val-090-observed-connection-ambiguous-flow-role`, `val-090-ocsf-endpoint-order-no-direction` | active_mvp |
| `has_theoretical_reachability` | none in MVP | none | none | none | none | none | none | none | prohibited | Emit `THEORETICAL_REACHABILITY_SCOPE_ERROR`, `REACHABILITY_DEFERRED_OUTPUT_FORBIDDEN`, or deterministic no-op; emit no fact, edge, property, API claim, or traversal class. | `val-090-theoretical-reachability-prohibited`, `val-090-boolean-reachability-property-prohibited` | inactive_deferred |
| `all_other_edge_types` | none | none | none | none | none | none | none | none | prohibited unless later active row set promotes the edge type | Reject activation or projection with `GRAPH_EDGE_SEMANTICS_ROW_MISSING`; emit no delta. | `val-090-mvp-active-profile-exact-edge-set` | inactive_unmapped |

The MVP active graph edge set is exactly `observed_connection`. Any additional active edge type requires a later authoritative profile row, semantics row, output eligibility row, query translation row, and validation fixture set before projection activation.

### GraphMVPClosureStatus

| Closure area | Required state |
| --- | --- |
| Active MVP edge set | Exactly `observed_connection`. |
| `has_theoretical_reachability` | Inactive and prohibited. |
| All other edge types | Rejected or inactive unless a future profile, semantics row, output eligibility row, query translation row, and fixture set activate them. |
| Direction evidence | Qualifying `FlowRoleEvidence` only. |
| OCSF endpoint order | Never direction authority. |
| Generic external graph payload | Not pathfinding-ready, finding-producing, metrics-producing, or identity-influencing by default. |

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

### PostgreSQL SQL query translation rows

The PostgreSQL relational profile must implement the query-class rows in the PostgreSQL SQL query translation matrix above through an active `GraphQueryTranslationProfile`. Query translation must materialize backend candidates only. It must not emit caller-visible order, page tokens, authorization decisions, redaction decisions, evidence drillback identity, or analysis findings directly from SQL execution.

| Required validation class | Required proof |
| --- | --- |
| node detail parity | One Cadastre node ID returns exactly one authorized node or the configured empty/error response; duplicate anchors fail. |
| neighbor expansion parity | Inbound and outbound adjacency match graph deltas after Cadastre ordering and redaction. |
| bounded path parity | Depths `1..6`, cycles, self-loops, empty traversal classes, and candidate overflow produce deterministic results or errors. |
| evidence drillback parity | Evidence refs are supplied from Cadastre evidence bridges and never from backend-native IDs. |
| analysis read-only parity | SQL and AGE-compatible analysis imports require expected query checksum, expected result checksum, and mutation-prohibition proof. |

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
| `input_dataset_version_refs` | Yes | Required when graph rebuild uses a table-set or logical dataset version. |
| `input_lakehouse_table_profile_refs` | Yes | Required for every authoritative table read by rebuild. |
| `input_snapshot_refs` | Yes | Authoritative lakehouse snapshot or dataset refs. |
| `input_commit_refs` | Yes | Required when rebuild consumes write or commit state. |
| `table_set_checksum` | Yes | Required when more than one authoritative table participates. |
| `cross_table_commit_profile_ref` | Yes | Required when table-set consistency is required. |
| `schema_compatibility_check_refs` | Yes | Required for every input schema affecting graph output. |
| `protected_input_ref_set_checksum` | Yes | SHA-256 over canonical protected input ref set. |
| `replay_retention_policy_refs` | Yes | Required when rebuild inputs must be protected from maintenance. |
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

`020.TableMaintenancePolicy` must treat `GraphRebuildManifest.protected_input_ref_set_checksum` and all listed protected refs as deletion or rewrite blockers unless a later active retention policy explicitly permits the candidate and graph rebuild validation passes.

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
| prior partial with committed batch evidence and prior lock loss | resume after last committed compatible batch only when the new run holds a valid `030.RunLockCommitGuard` and the same `GraphDeltaIdempotencyKey`. |
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
| `GRAPH_BACKEND_DEFAULT_UNRESOLVED` | Omitted backend selection materializes `mvp-postgresql-relational-graph.v1` but required default fields, package refs, provider evidence, schema refs, restore/upgrade refs, validation refs, benchmark thresholds, or blocker-set rows remain `TODO` or unresolved. |
| `GRAPH_PROVIDER_CAPABILITY_MISSING` | The selected profile lacks an active provider capability matrix. |
| `GRAPH_PROVIDER_CAPABILITY_UNSUPPORTED` | The provider capability matrix declares an enabled query, apply, rebuild, or serving capability unsupported. |
| `GRAPH_BACKEND_VERSION_UNPINNED` | Provider, driver, storage adapter, index adapter, or version ref is mutable, missing, or snapshot-only in production scope. |
| `GRAPH_BACKEND_PACKAGE_GATE_FAILED` | A required provider, driver, storage adapter, index adapter, runtime distribution, package release, or package-set gate from `100` failed. |
| `GRAPH_PROVIDER_ADAPTER_UNSUPPORTED` | The selected provider adapter cannot satisfy the declared provider profile, server mode, query language, or driver contract. |
| `GRAPH_SCHEMA_IMPLICIT_CREATION_FORBIDDEN` | Production provider config enables implicit schema creation or omits explicit schema proof. |
| `GRAPH_STORAGE_MODE_UNSAFE` | Storage mode is ephemeral, destructive, validation-only, or lacks durability evidence for production serving. |
| `GRAPH_INDEX_BACKEND_UNAVAILABLE` | Required index provider or mixed-index integration is unavailable for affected query classes. |
| `GRAPH_INDEX_FRESHNESS_REQUIRED` | Index freshness evidence is missing or stale for affected query classes. |
| `GRAPH_QUERY_FULL_SCAN_FORBIDDEN` | Query translation would require unbounded full graph scan or backend natural-order traversal in production. |

| `GRAPH_POSTGRES_SCHEMA_FINGERPRINT_STALE` | PostgreSQL relational schema fingerprint is absent, stale, mismatched, or incomplete. |
| `GRAPH_POSTGRES_DUPLICATE_NODE_ID` | PostgreSQL relational anchors contain or attempt to create a duplicate Cadastre graph node ID. |
| `GRAPH_POSTGRES_DUPLICATE_EDGE_ID` | PostgreSQL relational anchors contain or attempt to create a duplicate Cadastre graph edge ID. |
| `GRAPH_POSTGRES_ORPHAN_EDGE` | A PostgreSQL relational edge endpoint lacks a referenced Cadastre graph node anchor. |
| `GRAPH_POSTGRES_QUERY_TIMEOUT` | PostgreSQL candidate materialization exceeds the active timeout. |
| `GRAPH_POSTGRES_QUERY_PLAN_UNSUPPORTED` | PostgreSQL query translation requires an unsupported predicate, full scan, unsafe plan class, or unbounded natural-order traversal. |
| `GRAPH_POSTGRES_DIRECT_DML_FORBIDDEN` | A read-only or unapproved role attempts direct SQL DML against graph tables. |
| `GRAPH_POSTGRES_SEARCH_PATH_UNSAFE` | PostgreSQL `search_path` or object resolution preflight is unsafe for the selected backend profile. |
| `GRAPH_POSTGRES_RLS_BYPASS_FORBIDDEN` | PostgreSQL role or RLS configuration permits unauthorized bypass for API or apply roles. |
| `GRAPH_PROVIDER_UNSUPPORTED` | Selected provider, region, engine version, service tier, extension support status, or support scope is unsupported for activation. |
| `GRAPH_PROVIDER_SUPPORT_EVIDENCE_MISSING` | Provider support evidence required by the selected backend profile is missing, stale, or checksum-mismatched. |
| `GRAPH_PROVIDER_SUPPORT_EVIDENCE_EXPIRED` | Provider support evidence is past `evidence_expiry`; expired evidence behaves as missing. |
| `GRAPH_POSTGRES_SECURITY_PREFLIGHT_UNSAFE` | PostgreSQL role, RLS, raw-write bypass, or `search_path` preflight is missing, unsafe, stale, checksum-mismatched, package-set-mismatched, or unmanifested. |
| `GRAPH_BENCHMARK_THRESHOLD_MISSING` | Required benchmark threshold profile, workload fixture checksum, expected metric checksum, or validation ref is missing or `TODO:`. |
| `GRAPH_BENCHMARK_THRESHOLD_FAILED` | A selected benchmark run fails the active threshold profile or expected metric checksum. |
| `GRAPH_BACKEND_PACKAGE_COHORT_MISMATCH` | Selected backend runtime, adapter, driver, deployment profile, schema, query, apply, provider support, restore, upgrade, benchmark, or rollback package refs do not belong to the same active package set. |
| `GRAPH_QUERY_PARITY_FAILED` | PostgreSQL relational or AGE query parity fixture fails expected query checksum, expected result checksum, authorization, redaction, ordering, or mutation-prohibition proof. |
| `GRAPH_AGE_EXTENSION_MISSING` | AGE profile is selected but the required AGE extension package, control file, SQL script, or extension presence proof is missing. |
| `GRAPH_AGE_EXTENSION_VERSION_UNSUPPORTED` | AGE extension version or supported PostgreSQL major version does not match selected profile refs. |
| `GRAPH_AGE_MUTATION_FORBIDDEN` | Read-only AGE path attempts mutating Cypher, graph creation, graph drop, or AGE write behavior. |
| `GRAPH_AGE_NAMESPACE_BYPASS_FORBIDDEN` | A role attempts direct DML/DDL against an AGE graph namespace or bypasses declared AGE adapter paths. |
| `GRAPH_AGE_INTERNAL_ID_FORBIDDEN` | AGE vertex, edge, path, graph, label, or agtype object ID appears in response, page token, drillback ref, replay key, audit detail, or telemetry. |
| `GRAPH_BACKEND_RESTORE_UNVERIFIED` | Restore proof is missing, stale, failed, checksum-mismatched, or not run for the selected backend profile. |
| `GRAPH_BACKEND_UPGRADE_UNVERIFIED` | Upgrade rehearsal, migration rollback, or rebuild migration proof is missing, stale, failed, checksum-mismatched, or not run for the selected backend profile. |

### GraphErrorRegistryFragment

This owner fragment feeds `110.GenerateErrorCodeRegistry`. `110` owns the generated caller-visible registry. This table must not render API output by itself. No row may contain `TODO:` when graph serving, graph apply, graph query, graph rebuild, or graph backend activation is in promotion scope.

| error_code | owner_spec | severity | retry_class | caller_visible_fields | audit_visible_fields | redaction_rule | owner_context_schema_ref | fixture_ref |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `GRAPH_ARTIFACT_MISSING` | `090` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `090.GraphErrorContext` | `error-registry-090-graph-artifact-missing` |
| `GRAPH_ARTIFACT_INACTIVE` | `090` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `090.GraphErrorContext` | `error-registry-090-graph-artifact-inactive` |
| `GRAPH_ARTIFACT_CHECKSUM_MISMATCH` | `090` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `090.GraphErrorContext` | `error-registry-090-graph-artifact-checksum-mismatch` |
| `GRAPH_ARTIFACT_SCOPE_MISMATCH` | `090` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `090.GraphErrorContext` | `error-registry-090-graph-artifact-scope-mismatch` |
| `GRAPH_PROJECTION_CORE_OVERRIDE_FORBIDDEN` | `090` | `security_error` | `none` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.security_boundary` | `090.GraphErrorContext` | `error-registry-090-graph-projection-core-override-forbidden` |
| `GRAPH_FLOW_ROLE_EVIDENCE_REQUIRED` | `090` | `error` | `caller_correctable` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `090.GraphErrorContext` | `error-registry-090-graph-flow-role-evidence-required` |
| `GRAPH_EXPIRY_SOURCE_AUTHORITY_REQUIRED` | `090` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `090.GraphErrorContext` | `error-registry-090-graph-expiry-source-authority-required` |
| `GRAPH_HANDOFF_EFFECT_UNSUPPORTED` | `090` | `error` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `090.GraphErrorContext` | `error-registry-090-graph-handoff-effect-unsupported` |
| `GRAPH_EDGE_SEMANTICS_ROW_MISSING` | `090` | `error` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `090.GraphErrorContext` | `error-registry-090-graph-edge-semantics-row-missing` |
| `GRAPH_ENDPOINT_IDENTITY_UNRESOLVED` | `090` | `error` | `retry_after_owner_repair` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `090.GraphErrorContext` | `error-registry-090-graph-endpoint-identity-unresolved` |
| `GRAPH_QUERY_CANDIDATE_LIMIT_REACHED` | `090` | `error` | `caller_correctable` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `090.GraphErrorContext` | `error-registry-090-graph-query-candidate-limit-reached` |
| `GRAPH_PAGE_TOKEN_EXPIRED` | `090` | `error` | `caller_correctable` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `090.GraphErrorContext` | `error-registry-090-graph-page-token-expired` |
| `GRAPH_PAGE_TOKEN_INVALID` | `090` | `security_error` | `none` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.always_forbidden_sensitive_values` | `090.GraphErrorContext` | `error-registry-090-graph-page-token-invalid` |
| `GRAPH_TRAVERSAL_CLASS_REQUIRED` | `090` | `error` | `caller_correctable` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `090.GraphErrorContext` | `error-registry-090-graph-traversal-class-required` |
| `GRAPH_QUERY_TIMEOUT` | `090` | `error` | `transient_retryable` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `090.GraphErrorContext` | `error-registry-090-graph-query-timeout` |
| `GRAPH_QUERY_TRANSLATION_ERROR` | `090` | `error` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `090.GraphErrorContext` | `error-registry-090-graph-query-translation-error` |
| `DERIVED_VIEW_LAG_ERROR` | `090` | `blocked` | `retry_after_refresh` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `090.GraphErrorContext` | `error-registry-090-derived-view-lag-error` |
| `GRAPH_DELTA_IDEMPOTENCY_CONFLICT` | `090` | `error` | `none` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `090.GraphErrorContext` | `error-registry-090-graph-delta-idempotency-conflict` |
| `GRAPH_BACKEND_PROFILE_MISSING` | `090` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `090.GraphErrorContext` | `error-registry-090-graph-backend-profile-missing` |
| `GRAPH_SCHEMA_FINGERPRINT_STALE` | `090` | `blocked` | `retry_after_refresh` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `090.GraphErrorContext` | `error-registry-090-graph-schema-fingerprint-stale` |
| `GRAPH_RAW_WRITE_BYPASS` | `090` | `security_error` | `none` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.always_forbidden_sensitive_values` | `090.GraphErrorContext` | `error-registry-090-graph-raw-write-bypass` |
| `GRAPH_DRIFT_REPAIR_FORBIDDEN` | `090` | `security_error` | `none` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.security_boundary` | `090.GraphErrorContext` | `error-registry-090-graph-drift-repair-forbidden` |
| `GRAPH_APPLY_RESUME_UNSAFE` | `090` | `blocked` | `retry_after_owner_repair` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `090.GraphErrorContext` | `error-registry-090-graph-apply-resume-unsafe` |
| `GRAPH_REBUILD_EQUIVALENCE_FAILED` | `090` | `blocked` | `retry_after_owner_repair` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `090.GraphErrorContext` | `error-registry-090-graph-rebuild-equivalence-failed` |
| `THEORETICAL_REACHABILITY_SCOPE_ERROR` | `090` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `090.GraphErrorContext` | `error-registry-090-theoretical-reachability-scope-error` |
| `REACHABILITY_DEFERRED_OUTPUT_FORBIDDEN` | `090` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `090.GraphErrorContext` | `error-registry-090-reachability-deferred-output-forbidden` |
| `GRAPH_BACKEND_CONFIG_INCOMPLETE` | `090` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `090.GraphErrorContext` | `error-registry-090-graph-backend-config-incomplete` |
| `GRAPH_BACKEND_DEFAULT_UNRESOLVED` | `090` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `090.GraphErrorContext` | `error-registry-090-graph-backend-default-unresolved` |
| `GRAPH_PROVIDER_CAPABILITY_MISSING` | `090` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `090.GraphErrorContext` | `error-registry-090-graph-provider-capability-missing` |
| `GRAPH_PROVIDER_CAPABILITY_UNSUPPORTED` | `090` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `090.GraphErrorContext` | `error-registry-090-graph-provider-capability-unsupported` |
| `GRAPH_BACKEND_VERSION_UNPINNED` | `090` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `090.GraphErrorContext` | `error-registry-090-graph-backend-version-unpinned` |
| `GRAPH_BACKEND_PACKAGE_GATE_FAILED` | `090` | `blocked` | `retry_after_owner_repair` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `090.GraphErrorContext` | `error-registry-090-graph-backend-package-gate-failed` |
| `GRAPH_PROVIDER_ADAPTER_UNSUPPORTED` | `090` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `090.GraphErrorContext` | `error-registry-090-graph-provider-adapter-unsupported` |
| `GRAPH_SCHEMA_IMPLICIT_CREATION_FORBIDDEN` | `090` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `090.GraphErrorContext` | `error-registry-090-graph-schema-implicit-creation-forbidden` |
| `GRAPH_STORAGE_MODE_UNSAFE` | `090` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `090.GraphErrorContext` | `error-registry-090-graph-storage-mode-unsafe` |
| `GRAPH_INDEX_BACKEND_UNAVAILABLE` | `090` | `blocked` | `retry_after_owner_repair` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `090.GraphErrorContext` | `error-registry-090-graph-index-backend-unavailable` |
| `GRAPH_INDEX_FRESHNESS_REQUIRED` | `090` | `blocked` | `retry_after_refresh` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `090.GraphErrorContext` | `error-registry-090-graph-index-freshness-required` |
| `GRAPH_QUERY_FULL_SCAN_FORBIDDEN` | `090` | `error` | `caller_correctable` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `090.GraphErrorContext` | `error-registry-090-graph-query-full-scan-forbidden` |

| `GRAPH_POSTGRES_SCHEMA_FINGERPRINT_STALE` | `090` | `blocked` | `retry_after_refresh` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `090.GraphErrorContext` | `error-registry-090-graph-postgres-schema-fingerprint-stale` |
| `GRAPH_POSTGRES_DUPLICATE_NODE_ID` | `090` | `error` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `090.GraphErrorContext` | `error-registry-090-graph-postgres-duplicate-node-id` |
| `GRAPH_POSTGRES_DUPLICATE_EDGE_ID` | `090` | `error` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `090.GraphErrorContext` | `error-registry-090-graph-postgres-duplicate-edge-id` |
| `GRAPH_POSTGRES_ORPHAN_EDGE` | `090` | `error` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `090.GraphErrorContext` | `error-registry-090-graph-postgres-orphan-edge` |
| `GRAPH_POSTGRES_QUERY_TIMEOUT` | `090` | `error` | `transient_retryable` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `090.GraphErrorContext` | `error-registry-090-graph-postgres-query-timeout` |
| `GRAPH_POSTGRES_QUERY_PLAN_UNSUPPORTED` | `090` | `error` | `caller_correctable` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `090.GraphErrorContext` | `error-registry-090-graph-postgres-query-plan-unsupported` |
| `GRAPH_POSTGRES_DIRECT_DML_FORBIDDEN` | `090` | `security_error` | `none` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.security_boundary` | `090.GraphErrorContext` | `error-registry-090-graph-postgres-direct-dml-forbidden` |
| `GRAPH_POSTGRES_SEARCH_PATH_UNSAFE` | `090` | `security_error` | `none` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.security_boundary` | `090.GraphErrorContext` | `error-registry-090-graph-postgres-search-path-unsafe` |
| `GRAPH_POSTGRES_RLS_BYPASS_FORBIDDEN` | `090` | `security_error` | `none` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.security_boundary` | `090.GraphErrorContext` | `error-registry-090-graph-postgres-rls-bypass-forbidden` |
| `GRAPH_PROVIDER_UNSUPPORTED` | `090` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `090.GraphErrorContext` | `error-registry-090-graph-provider-unsupported` |
| `GRAPH_PROVIDER_SUPPORT_EVIDENCE_MISSING` | `090` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `090.GraphErrorContext` | `error-registry-090-graph-provider-support-evidence-missing` |
| `GRAPH_PROVIDER_SUPPORT_EVIDENCE_EXPIRED` | `090` | `blocked` | `retry_after_refresh` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `090.GraphErrorContext` | `error-registry-090-graph-provider-support-evidence-expired` |
| `GRAPH_POSTGRES_SECURITY_PREFLIGHT_UNSAFE` | `090` | `security_error` | `none` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.security_boundary` | `090.GraphErrorContext` | `error-registry-090-graph-postgres-security-preflight-unsafe` |
| `GRAPH_BENCHMARK_THRESHOLD_MISSING` | `090` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `090.GraphErrorContext` | `error-registry-090-graph-benchmark-threshold-missing` |
| `GRAPH_BENCHMARK_THRESHOLD_FAILED` | `090` | `blocked` | `retry_after_owner_repair` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `090.GraphErrorContext` | `error-registry-090-graph-benchmark-threshold-failed` |
| `GRAPH_BACKEND_PACKAGE_COHORT_MISMATCH` | `090` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `090.GraphErrorContext` | `error-registry-090-graph-backend-package-cohort-mismatch` |
| `GRAPH_QUERY_PARITY_FAILED` | `090` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `090.GraphErrorContext` | `error-registry-090-graph-query-parity-failed` |
| `GRAPH_AGE_EXTENSION_MISSING` | `090` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `090.GraphErrorContext` | `error-registry-090-graph-age-extension-missing` |
| `GRAPH_AGE_EXTENSION_VERSION_UNSUPPORTED` | `090` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `090.GraphErrorContext` | `error-registry-090-graph-age-extension-version-unsupported` |
| `GRAPH_AGE_MUTATION_FORBIDDEN` | `090` | `security_error` | `none` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.security_boundary` | `090.GraphErrorContext` | `error-registry-090-graph-age-mutation-forbidden` |
| `GRAPH_AGE_NAMESPACE_BYPASS_FORBIDDEN` | `090` | `security_error` | `none` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.security_boundary` | `090.GraphErrorContext` | `error-registry-090-graph-age-namespace-bypass-forbidden` |
| `GRAPH_AGE_INTERNAL_ID_FORBIDDEN` | `090` | `security_error` | `none` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.security_boundary` | `090.GraphErrorContext` | `error-registry-090-graph-age-internal-id-forbidden` |
| `GRAPH_BACKEND_RESTORE_UNVERIFIED` | `090` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `090.GraphErrorContext` | `error-registry-090-graph-backend-restore-unverified` |
| `GRAPH_BACKEND_UPGRADE_UNVERIFIED` | `090` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `090.GraphErrorContext` | `error-registry-090-graph-backend-upgrade-unverified` |

### GraphErrorContext

`GraphErrorContext` is the owner context schema for `090` graph projection, graph query, graph apply, graph rebuild, backend preflight, and endpoint-handoff registry rows.

| Field | Required | Rule |
| --- | ---: | --- |
| `context_schema_version` | Yes | Immutable `090` context schema version. |
| `owner_spec` | Yes | Must be `090`. |
| `error_code` | Yes | Must match the generated registry row. |
| `failure_class` | Yes | Closed token: `projection`, `flow_role`, `graph_expiry`, `graph_handoff`, `edge_semantics`, `endpoint_identity`, `query`, `page_token`, `apply`, `rebuild`, `backend_profile`, `schema`, `provider_capability`, `index_freshness`, `reachability_boundary`, or `raw_write_bypass`. |
| `operation` | Yes | Graph projection, query translation, query execution, graph apply, rebuild, backend preflight, drift check, or derived-view serving operation. |
| `affected_record_type` | Yes | Graph profile, graph delta, graph edge, graph query, apply result, backend profile, derived view, or endpoint handoff type. |
| `field_path` | Yes | Exact field path when applicable; null for artifact-wide failures. |
| `artifact_refs` | Yes | Canonically sorted refs to graph profiles, graph deltas, edge semantics rows, traversal classes, output eligibility rows, query translation profiles, apply profiles, derived-view lag policies, backend profiles, provider capability rows, schema fingerprints, graph apply results, graph rebuild manifests, index consistency checks, validation fixtures, package-set refs, or version manifests consulted by the error; empty only when no artifact was consulted. |
| `graph_operation` | Yes | Closed operation token. |
| `projection_profile_ref` | No | Required when graph projection was consulted. |
| `edge_type` | No | Required when edge semantics or endpoint identity is involved. |
| `query_class` | No | Required for query and page-token failures. |
| `backend_profile_ref` | No | Required for backend, provider, schema, storage, index, or adapter failures. |
| `derived_view_state_ref` | No | Required when serving state or lag affected the failure. |
| `graph_handoff_effect` | No | Required when a graph handoff effect was evaluated. |
| `missing_authority_refs` | No | Required when source authority blocks graph expiry or cleanup. |
| `backend_evidence_refs` | No | Redacted refs only; backend IDs and provider-native query text remain forbidden. |
| `blocking_reason` | Yes when generated row severity is `blocked` | Bounded reason; otherwise null or omitted. |
| `validation_refs` | Yes | Exact `120` graph fixture refs. |
| `redaction_classes` | Yes | Backend IDs, provider-native query text, raw graph property values, private bindings, credentials, raw payload bytes, and source-native identity values must map to `always_forbidden`. |

### ActivationControlledRowSchemaPrecisionHandoff

The following `090` row families can affect graph projection, serving, query translation, backend selection, provider capability, edge semantics, output eligibility, schema/apply/query policies, graph rebuild, or derived-view lag. Each output-affecting family must use a complete `030.ActivationControlledRowField` table before production selection. Until the required table is present and non-`TODO`, `ValidateSpecSet` must classify the family as `blocked_validation`.

| row_family | production classification | required precision status |
| --- | --- | --- |
| `GraphBackendProfile` | output_affecting | Closed for backend selection by `GraphBackendActivationControlledRowFieldClosure`; production still requires concrete selected row refs, provider support evidence, package-set refs, validation refs, and manifest inclusion. |
| `GraphBackendActivationBlockerSet` | output_affecting | Closed by `GraphBackendActivationControlledRowFieldClosure`; unresolved blockers force selection-only behavior and `GRAPH_BACKEND_DEFAULT_UNRESOLVED`. |
| `GraphProviderCapabilityMatrix` | output_affecting for provider activation | Closed by `GraphBackendActivationControlledRowFieldClosure`; unsupported or partial behavior must be explicit and validation-backed. |
| `GraphProviderSupportEvidenceRow` | output_affecting for provider activation | Closed by `GraphBackendActivationControlledRowFieldClosure`; missing, expired, or unsupported evidence blocks production. |
| `GraphProjectionProfile` | output_affecting | Existing graph projection rows remain owner-local; production fails when selected projection row refs, edge refs, eligibility refs, or validation refs are missing or TODO-bearing. |
| `GraphEdgeSemanticsRegistry` | output_affecting | Existing edge semantics rows remain owner-local; production fails when selected edge semantics row refs, traversal refs, or validation refs are missing or TODO-bearing. |
| `GraphObjectOutputEligibilityRow` | output_affecting | Existing output-eligibility rows remain owner-local; production fails when selected eligibility refs or validation refs are missing or TODO-bearing. |
| `GraphBackendTaxonomyMappingProfile` | output_affecting | Backend taxonomy mapping must use structured row refs and backend-ID prohibition; missing concrete row refs block production. |
| `GraphQueryTranslationProfile` | output_affecting | Closed for PostgreSQL/AGE backend closure by `GraphBackendActivationControlledRowFieldClosure`; candidate limits are imported from `GraphQueryCandidateLimitTable`. |
| `GraphReadModelSchemaProfile` | output_affecting | Closed for PostgreSQL/AGE backend closure by `GraphBackendActivationControlledRowFieldClosure`; relational anchor schema and fingerprint refs are mandatory. |
| `BackendSchemaFingerprint` | output_affecting | Closed by `GraphBackendActivationControlledRowFieldClosure`; stale or incomplete fingerprints block apply, query, rebuild, restore, upgrade, and replay. |
| `GraphApplyProfile` | output_affecting | Closed for PostgreSQL relational apply by `GraphBackendActivationControlledRowFieldClosure`; `max_batch_deltas` defaults to `1000`, minimum `1`, maximum `5000`, and retries are disabled by default. |
| `GraphRestoreRehearsalEvidenceRow` | output_affecting for promotion | Closed by `GraphBackendActivationControlledRowFieldClosure`; missing or stale proof blocks promotion. |
| `GraphUpgradeRehearsalEvidenceRow` | output_affecting for promotion | Closed by `GraphBackendActivationControlledRowFieldClosure`; missing or stale proof blocks promotion. |
| `GraphBenchmarkThresholdProfile` | output_affecting when performance gates are in scope | Closed by `GraphBackendActivationControlledRowFieldClosure`; missing thresholds keep acceptance blocked. |
| `DerivedViewLagPolicy` | output_affecting for graph serving | Existing lag rows remain owner-local; selected refs, health/API handoff refs, and validation refs are mandatory before serving. |

Backend refs must be structured artifact refs with package-set requirements. Provider subprofiles must have closed extension maps and must not alter Cadastre graph IDs, output ordering, page-token identity, evidence refs, error categories, authorization, redaction, or replay semantics. Arrays of refs, node types, edge types, properties, labels, traversal classes, and validation refs must declare `canonical_set` or `ordered_sequence` behavior.

Graph projection, graph query, graph apply, graph rebuild, derived-view serving, or backend activation must fail before mutation or serving when any selected `090` row family remains `TODO:`, uses a backend-native ID as a ref, uses a bare string row ref, or omits selected row refs from `030.VersionManifest`.

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
| `090-GRAPH-TELEMETRY-HANDOFF-AC-001` | Graph telemetry may report graph operation diagnostics but cannot expose backend IDs or provider query text, cannot repair drift, cannot authorize graph mutation, and cannot substitute for graph apply, rebuild, schema, index, or derived-view state records. |
| `090-GRAPH-BACKEND-PACKAGE-TOKEN-AC-001` | Graph backend package gates consume exact `100.PackageType` tokens and selected `100.PackageTypePolicyRowSet` coverage; broad labels, provider-specific package names, module names, runtime labels, and version strings fail before backend preflight succeeds. |
| `090-GRAPH-BACKEND-PACKAGE-HANDOFF-AC-001` | When graph backend artifacts are package-supplied, backend mutation, query serving, rebuild promotion, drift check, and health success require `100.PackageReleaseManifest`, `100.ProductionPackageSetManifest`, selected policy/deprecation/compatibility/trust/repository/validation refs, and package `030.VersionManifest` refs. |

| `090-EDGESET-AC-001` | The active MVP graph edge set is exactly `observed_connection`; theoretical reachability remains inactive and prohibited. |
| `090-GRAPH-ID-COLLISION-AC-001` | Graph node and edge delta ID collisions fail with `GRAPH_NODE_DELTA_ID_COLLISION` or `GRAPH_EDGE_DELTA_ID_COLLISION` before graph apply commits. |
| `090-REBUILD-MANIFEST-AC-001` | Every graph rebuild promotion has a `GraphRebuildManifest` with input refs, profile refs, schema fingerprint, output checksum, status, and errors. |
| `090-REBUILD-MANIFEST-TABLESTATE-AC-001` | Graph rebuild manifests expose dataset version refs, table profile refs, snapshot refs, commit refs, table-set checksum, cross-table profile refs, schema compatibility refs, protected input checksum, and replay-retention refs when applicable. |
| `090-REBUILD-RETENTION-PROTECTION-AC-001` | Table maintenance treats protected graph rebuild inputs as deletion/rewrite blockers unless an active retention policy explicitly permits the candidate and rebuild validation passes. |
| `090-VOLATILITY-AC-001` | Inactive graph projection profile, query translation checksum mismatch, apply profile manifest omission, backend ID exposure, and attempted `has_theoretical_reachability` activation fail before mutation or serving. |
| `090-VOLATILITY-AC-002` | Graph apply default values, maximums, retry classes, and retry schedule are explicit and no backend default is inherited. |
| `090-LIFECYCLE-AC-001` | Graph apply lifecycle matrix is total. |
| `090-LIFECYCLE-AC-002` | Identical reapply produces no mutation and no duplicate graph objects. |
| `090-LIFECYCLE-AC-003` | Partial apply without committed-batch proof fails with `GRAPH_APPLY_RESUME_UNSAFE`. |
| `090-LIFECYCLE-AC-004` | Failed, partial, aborted, or schema-preflight-failed apply never advances derived-view or watermark state. |
| `090-GRAPH-APPLY-DEFAULTS-AC-001` | Graph apply defaults and maximums are explicit. |
| `090-OCSF-DIRECTION-AC-001` | OCSF endpoint order alone never determines `observed_connection` edge direction, and the rejection fixture includes the active `050.ObservationToOCSFMappingRow` ref, source silver observation checksum, graph projection profile ref, edge semantics ref, and `VersionManifest` ref. |
| `090-SOURCE-CLOSURE-GRAPH-AC-001` | Graph expiry delta without `060` requested-effect authorization fails before apply. |
| `090-SOURCE-CLOSURE-GRAPH-AC-002` | Graph derived-view lag never maps to source staleness or source absence. |
| `090-SOURCE-CLOSURE-GRAPH-AC-003` | Missing flow evidence emits no absence edge and no expiry. |
| `090-SOURCE-CLOSURE-GRAPH-AC-004` | Deterministic source-authority block rows emit no graph delta and no backend mutation. |
| `090-SOURCE-CLOSURE-GRAPH-AC-005` | A graph expiry or cleanup delta missing closure row-set refs, closure validation refs, or underlying `060` row refs fails before graph apply. |
| `090-SOURCE-CLOSURE-GRAPH-AC-006` | Backend cleanup, missing backend object, graph apply success, graph drift result, derived-view stale state, source-history no-change proof, missing flow evidence, ambiguous `FlowRoleEvidence`, and OCSF endpoint order never authorize graph expiry or cleanup without exact `060` requested-effect authorization. |
| `090-SOURCE-CLOSURE-GRAPH-AC-007` | Every `000` closure state and upstream block reason in `SourceEffectGraphOutcomeMatrix` maps to exactly one graph behavior with no backend mutation unless `closed` authorizes the exact graph effect. |
| `090-SOURCE-DATASET-CATALOG-GRAPH-AC-001` | Expiry and cleanup deltas require selected `020.SourceDatasetCatalogRow` refs/checksums in the producing `VersionManifest`. |
| `090-SOURCE-DATASET-CATALOG-GRAPH-AC-002` | Missing, blocked, ambiguous, inactive, checksum-mismatched, unvalidated, or unmanifested source-dataset catalog rows reject before graph delta persistence and produce no backend mutation. |
| `090-GRAPH-HANDOFF-AC-001` | `none` handoff emits no graph delta. |
| `090-GRAPH-HANDOFF-AC-002` | `reproject_fact_key` recomputes affected projected graph objects and includes source `GoldFactChangeSet` refs. |
| `090-GRAPH-HANDOFF-AC-003` | `expire_projected_object` emits expiry only when `060` authorizes `graph_expiry`. |
| `090-GRAPH-HANDOFF-AC-004` | `cleanup_projected_object` emits cleanup only when `060` authorizes `cleanup`. |
| `090-GRAPH-HANDOFF-AC-005` | `conflict_visibility_update` is not pathfinding-eligible by default. |
| `090-GRAPH-HANDOFF-AC-006` | `identity_split_handoff` routes through projection only and does not let the resolver or correction engine mutate graph state. |
| `090-OCSF-DIRECTION-AC-002` | A `network_activity_observation` with valid OCSF Network Activity fields but missing or ambiguous `FlowRoleEvidence` emits no `observed_connection` edge, and the fixture includes active mapping-row refs rather than graph-only synthetic inputs. |
| `090-OCSF-DIRECTION-AC-003` | DNS and DHCP observations do not emit `observed_connection` edges unless a graph projection row and `FlowRoleEvidence` explicitly permit the edge, and direction fixtures include active mapping-row refs and expected no graph mutation. |
| `090-IDENTITY-SPLIT-PROJECTION-AC-001` | Identity split handoff projection consumes `070.GraphCorrectionHandoff` only as projection input and never permits resolver graph mutation. |
| `090-IDENTITY-SPLIT-PROJECTION-AC-002` | The same split handoff, affected fact refs, projection profile, and version manifest produce byte-identical graph deltas across replay. |
| `090-IDENTITY-SPLIT-HANDOFF-VALID-AC-001` | `val-090-identity-split-handoff-valid` proves complete `070.GraphCorrectionHandoff` metadata projects deterministic graph deltas. |
| `090-IDENTITY-SPLIT-HANDOFF-MISSING-AC-001` | `val-090-identity-split-handoff-missing-metadata` rejects before delta persistence when required handoff fields are absent. |
| `090-IDENTITY-SPLIT-HANDOFF-CHECKSUM-AC-001` | `val-090-identity-split-handoff-checksum-drift` rejects before delta persistence when the handoff checksum drifts. |
| `090-UNRESOLVED-ENDPOINT-NO-EDGE-AC-001` | `val-090-unresolved-endpoint-no-edge` emits `GRAPH_ENDPOINT_IDENTITY_UNRESOLVED`, no node, no edge, and no identity mutation. |
| `090-SELECTOR-ONLY-NO-ENDPOINT-AC-001` | `val-090-selector-only-no-endpoint` proves selector-only input cannot create graph endpoint identity. |
| `090-IDENTITY-SPLIT-PROJECTION-AC-003` | Missing split handoff metadata emits no graph delta and rejects before delta persistence. |
| `090-IDENTITY-SPLIT-PROJECTION-AC-004` | `VersionManifest` includes graph correction handoff refs and resolver explanation checksum refs when identity split projection occurs. |
| `090-GRAPH-PROFILE-CLOSURE-AC-001` | `mvp-observed-flow-graph.v1` has exactly one active edge type, `observed_connection`, and rejects all other edge types. |
| `090-GRAPH-PROFILE-CLOSURE-AC-002` | Positive observed connection projection emits nodes and one edge only when both endpoint identities are resolved and qualifying `FlowRoleEvidence` exists. |
| `090-GRAPH-PROFILE-CLOSURE-AC-003` | Missing or ambiguous `FlowRoleEvidence` emits no edge, no absence edge, no expiry, and no watermark. |
| `090-GRAPH-PROFILE-CLOSURE-AC-004` | Generic external graph payloads and synthetic structural objects are non-emitting and not pathfinding-eligible in MVP. |
| `090-GRAPH-QUERY-CLOSURE-AC-001` | Bounded path queries with omitted traversal classes fail, and empty traversal-class arrays return no paths. |
| `090-GRAPH-QUERY-CLOSURE-AC-002` | Candidate-limit, expired-token, and token-mismatch cases fail before backend query execution with the declared graph error codes. |
| `090-GRAPH-APPLY-ORDER-AC-001` | Same graph delta set and apply profile produce byte-identical canonical delta ordering and batch membership. |
| `090-RUNLOCK-GRAPH-AC-001` | `val-090-runlock-graph-apply-resume` proves missing, stale, fenced, lock-lost, or out-of-scope guards reject before backend execution; partial apply under lock loss does not advance derived view; resume requires committed-batch proof, the same `GraphDeltaIdempotencyKey`, a new valid `030.RunLockCommitGuard`, and `GraphApplyResult` guard refs. |
| `090-GRAPH-REBUILD-CLOSURE-AC-001` | Rebuild equivalence includes profile, edge semantics, output eligibility, taxonomy, query translation, property policy, schema, index consistency, and canonical output checksums. |
| `090-GRAPH-BACKEND-SELECTION-AC-001` | Graph serving enabled with omitted backend profile materializes `mvp-postgresql-relational-graph.v1`; graph serving disabled materializes no backend; explicit AGE remains validation-only unless production gates pass; unresolved default fields block production serving with `GRAPH_BACKEND_DEFAULT_UNRESOLVED`. |
| `090-GRAPH-BACKEND-DEFAULT-AC-002` | When graph serving is enabled and backend profile input is omitted, the implementation materializes `mvp-postgresql-relational-graph.v1`; if any required production profile field, package ref, provider support row, capability row, schema ref, relational anchor schema fingerprint, restore/upgrade evidence, storage/index declaration, validation ref, or performance threshold is unresolved, graph mutation, query, rebuild promotion, and drift check fail with `GRAPH_BACKEND_DEFAULT_UNRESOLVED`. |
| `090-GRAPH-BACKEND-PREFLIGHT-AC-001` | Backend preflight rejects missing profile, inactive profile, checksum mismatch, omitted required profile fields, unpinned provider version, unsafe storage mode, implicit schema creation, and missing package gates before mutation or query. |
| `090-GRAPH-BACKEND-PACKAGE-TOKEN-AC-001` | Graph backend package gates use exact tokens `graph_backend_provider_package`, `graph_provider_adapter_package`, `graph_backend_driver_package`, `graph_storage_backend_adapter_package`, `graph_index_backend_adapter_package`, `graph_backend_runtime_distribution`, and `graph_backend_deployment_profile`; broad labels and provider-specific package names fail before backend preflight succeeds. |
| `090-GRAPH-PROVIDER-CAPABILITY-AC-001` | Every active graph provider profile satisfies `GraphProviderCapabilityMatrix` or declares deterministic unsupported behavior for each query, apply, rebuild, and serving class. |
| `090-GRAPH-POSTGRES-SCHEMA-AC-001` | PostgreSQL relational schema fixtures reject duplicate node IDs, duplicate edge IDs, orphan edges, invalid edge types, invalid traversal classes, invalid intervals, stale fingerprints, missing role/RLS/search-path proofs, and undeclared JSONB property paths. |
| `090-GRAPH-POSTGRES-QUERY-AC-001` | PostgreSQL SQL query translation fixtures prove provider-neutral `QueryGraph` outputs for node detail, neighbor expansion, bounded path depths `1..6`, evidence drillback, and analysis read-only; reject full scans, unsupported predicates, timeout, cancellation, SQL natural-order output, backend IDs, SQL cursor names, prepared statement names, OIDs, tuple IDs, and sequence values. |
| `090-GRAPH-POSTGRES-APPLY-AC-001` | PostgreSQL apply fixtures prove deterministic batch membership, idempotent reapply, idempotency conflict, duplicate collision handling, partial apply resume, lock loss no advancement, stale fingerprint rejection, and read-after-write proof. |
| `090-GRAPH-POSTGRES-RESTORE-UPGRADE-AC-001` | PostgreSQL restore, clean-environment rebuild, schema migration, rollback, and major-version upgrade fixtures pass or block production promotion with owner errors. |
| `090-GRAPH-POSTGRES-BENCHMARK-AC-001` | PostgreSQL benchmark closure blocks production until `medium_mvp` and `large_stress` threshold rows have concrete workload, threshold, metric checksum, package-set, manifest, and validation refs. |
| `090-GRAPH-POSTGRES-SECURITY-PREFLIGHT-AC-001` | PostgreSQL role, RLS, raw-write bypass, and `search_path` preflight rows fail closed with `GRAPH_POSTGRES_SECURITY_PREFLIGHT_UNSAFE` and never inherit provider defaults. |
| `090-GRAPH-AGE-PREFLIGHT-AC-001` | AGE extension missing, unsupported version, unsupported provider, unsafe `search_path`, missing extension files/control scripts, or missing provider support evidence blocks AGE activation. |
| `090-GRAPH-AGE-SECURITY-AC-001` | AGE read-only Cypher mutation attempts, AGE graph namespace DML/DDL, graph drop, and namespace bypass attempts fail with no mutation. |
| `090-GRAPH-AGE-ID-LEAK-AC-001` | AGE internal vertex, edge, path, graph, label, and agtype object IDs never appear in API output, page tokens, audit output, telemetry, validation reports, evidence refs, or replay keys. |
| `090-GRAPH-PROVIDER-PORTABILITY-AC-001` | A future provider can satisfy the same capability, taxonomy, query translation, schema, apply, rebuild, ordering, ID, page-token, and error contracts without vendor-specific fields leaking into Cadastre outputs. |
| `090-CLOSED-FACT-CONSUMPTION-AC-001` | `observed_connection` emits only when both endpoints resolve to canonical entities through qualifying identity decisions. |
| `090-CLOSED-FACT-CONSUMPTION-AC-002` | `string_value` endpoints, OCSF endpoint objects, graph keys, backend IDs, and source-native graph payloads do not project as endpoint identity. |
| `090-CLOSED-FACT-CONSUMPTION-AC-003` | `identifier_ref` and `source_asset_ref` endpoint candidates require `070` resolver handoff before graph output. |
| `090-CLOSED-FACT-CONSUMPTION-AC-004` | Invalid `GoldFact.subject_ref` or `GoldFact.object_value` kinds produce no graph mutation and emit the most specific owner error. |

### Structured input graph acceptance criteria

| ID | Criterion |
| --- | --- |
| `090-STRUCTURED-INPUT-GRAPH-AC-001` | Repository-authored graph profile omitted from materialization, package-set, validation, or manifest refs fails backend preflight before graph serving, graph apply, query, rebuild promotion, or mutation. |
| `090-STRUCTURED-INPUT-GRAPH-AC-002` | Git-only backend profile cannot serve queries, mutate graph, advance derived view, create schema, or promote rebuild output. |
| `090-STRUCTURED-INPUT-GRAPH-AC-003` | Provider query text committed in Git cannot bypass `GraphQueryTranslationProfile`, authorization, redaction, deterministic ordering, page-token rules, or mutation prohibition. |
| `090-STRUCTURED-INPUT-GRAPH-AC-004` | Template-only graph profile activation, stale CI for query translation, publication manifest mismatch, sync-only backend default selection, generated query text bypass, and repository group mismatch fail before graph mutation or query serving. |

## Definition of Done

| ID | Criterion |
| --- | --- |
| `090-SCOPE-GRAPH-AC-001` | Exact graph profile selection, ambiguous graph profile rejection, backend activation scope mismatch, backend-generated ID leakage, selector-only endpoint rejection, and graph scope context manifest-inclusion fixtures pass. |
| `090-SCOPE-GRAPH-AC-002` | Graph profile or backend ambiguity selects no row, emits no graph delta, and performs no backend mutation. |
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

### Production activation blockers

All PostgreSQL relational TODO rows are production activation blockers for the omitted-backend default profile. AGE TODO rows are blockers only when AGE is selected or AGE production activation is requested.

| ID | Blocker | Blocking scope | Required owner decision | Default until resolved |
| --- | --- | --- | --- | --- |
| `090-TODO-POSTGRES-VERSION-PIN` | TODO: Pin exact PostgreSQL major and patch version, PostgreSQL runtime distribution, provider adapter package, driver package, and deployment provider/scope using exact `100.PackageType` tokens before active production profile. | P0 before active production default profile. | Product governance plus `100` package-set evidence. | `mvp-postgresql-relational-graph.v1` fails with `GRAPH_BACKEND_DEFAULT_UNRESOLVED`. |
| `090-POSTGRES-APPLY-BATCH-SIZE-RESOLVED` | PostgreSQL relational apply imports `GraphApplyProfile` defaults with `max_batch_deltas = 1000`, minimum `1`, maximum `5000`, retry disabled by default, no implicit retry classes, one declared batch per transaction boundary, required read-after-write proof, stale schema rejection, and `ResumeGraphApply` resume semantics. | Closed for schema behavior; production still requires concrete selected row refs and `120-GRAPH-POSTGRES-APPLY-*` fixture checksums. | `090.GraphApplyProfile`, `030.VersionManifest`, and `120` validation rows. | Missing concrete validation rows keep acceptance blocked, not unspecified. |
| `090-TODO-POSTGRES-BENCHMARK-THRESHOLDS` | TODO: Provide medium-MVP and large-stress graph size tiers and p50/p95/p99 latency, throughput, overflow, WAL volume, index build, bloat/vacuum, timeout, cancellation, restore, and upgrade thresholds. | P0 for production promotion when performance gates are in scope. | Product governance plus `120-GRAPH-BENCHMARK-*`. | Acceptance returns `blocked`, not `pass`. |
| `090-TODO-POSTGRES-RESTORE-UPGRADE-EVIDENCE` | TODO: Provide clean-environment restore, migration rollback, rebuild migration, and PostgreSQL major-version upgrade evidence. | P0 before production promotion. | `090`, `100`, and `120`. | Restore or upgrade state blocks promotion with `GRAPH_BACKEND_RESTORE_UNVERIFIED` or `GRAPH_BACKEND_UPGRADE_UNVERIFIED`. |
| `090-TODO-POSTGRES-PROVIDER-SUPPORT` | TODO: Provide provider, region, engine version, service tier, support status, backup/restore support, and upgrade path evidence. | P0 before production promotion. | Product governance plus provider support validation rows. | Backend preflight fails with `GRAPH_PROVIDER_SUPPORT_EVIDENCE_MISSING` or `GRAPH_PROVIDER_UNSUPPORTED`. |
| `090-TODO-AGE-EXTENSION-VERSION` | TODO: Pin exact AGE extension version, supported PostgreSQL major version, extension package, extension files/control scripts, provider support status, and upgrade path if AGE is selected. | P0 only for AGE selection or AGE production request. | Product governance plus `120-GRAPH-AGE-*`. | AGE remains validation-only and blocked. |
| `090-TODO-ERROR-FRAGMENT-COMPLETION` | TODO: Complete all graph backend owner error fragments using `110.ErrorCodeRegistryRow` fields. | Error registry generation and API/export visibility. | `090` plus `110.GenerateErrorCodeRegistry` validation. | `110.GenerateErrorCodeRegistry` fails with `ERROR_REGISTRY_OWNER_FRAGMENT_INCOMPLETE`. |
