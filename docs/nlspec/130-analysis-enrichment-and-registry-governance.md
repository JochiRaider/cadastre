---
doc_id: CADASTRE-NLSPEC-130
title: Analysis, Enrichment, and Registry Governance
doc_type: nlspec
status: candidate
---

## Authority

This document owns the contracts listed in `Exports`. Other Cadastre NLSpecs may import those contracts by exact name and must not restate them. This document has implementation authority only after the document registry marks it `authoritative` and its acceptance criteria pass.

## Purpose

Define non-authoritative analysis outputs, enrichment records, lineage mapping, and governance artifacts without confusing them with facts, source authority, identity, or graph mutation.

## Explicit Non-Scope

- Gold derivation core algorithm except derivation-rule interface.
- Graph query execution.
- Package trust.
- Source completeness or identity authority.

## Imports

- `GraphProjectionProfile`
- `CadastreSilverObservation`
- `SourceAuthorityProfile`
- `GraphQueryResponse`
- `RedactionPolicy`
- `ValidationMatrix`
- `ActivationControlledArtifactRef`
- `EvidenceRef`
- `EvidenceArtifactClassRegistry`
- `090.GraphEdgeSemantics`
- `090.GraphTraversalClass`
- `090.GraphObjectOutputEligibilityRow`
- `090.GraphQueryTranslationProfile`
- `090.DerivedViewLagPolicy`
- `090.DerivedViewState`
- `090.GraphPropertyEvidencePolicy`
- `080.DeriveFacts`
- `080.ReplayEquivalencePolicy`
- `060.SourceAuthorityProfileRow`
- `060.AbsenceDerivationResult`
- `030.ActivationScope`
- `030.ScopeSelectorContext`
- `030.ScopeSelectorCovers`
- `030.ActivationControlledRowSchema`
- `030.ActivationControlledRowField`
- `030.ActivationControlledRowRef`
- `030.ActivationControlledRowSetSchema`
- `020.SourceDatasetCatalogRow`

## Exports

- `AnalysisRuleBundle`
- `AnalysisRule`
- `AnalysisFinding`
- `AnalysisMetric`
- `RiskAcceptanceRecord`
- `DerivationRuleBundle`
- `DerivedGraphEdgeRule`
- `RuleGraphCompatibilityMatrix`
- `ThreatIntelEnrichmentProfile`
- `ThreatIntelEnrichmentRecord`
- `ThreatIntelDistributionMappingPolicy`
- `ThreatIntelArtifactRef`
- `RunDatasetIOContract`
- `LineageFacetMappingPolicy`
- `ArtifactClassPolicy`
- `RegistryArtifactGovernance`
- `RegistryCustomPropertySchema`
- `RegistryClassificationPolicy`
- `AnalyzeGraph`
- `ApplyDerivationRules`
- `CreateRiskAcceptance`
- `ValidateAnalysisQueryImport`
- `AnalysisRegistryArtifactLifecycleGuardRows`

## Non-Authority Rule

Analysis, enrichment, lineage, and registry outputs must not mutate facts, graph state, completeness, watermarks, identity, package state, or source authority unless another active owner spec grants a named interface.

## Analysis Rules

`AnalysisRuleBundle` contains read-only rules. A rule may query declared read models only when `RuleGraphCompatibilityMatrix` passes for the active graph projection profile, graph edge semantics row refs, traversal class set, graph object output eligibility row refs, graph property refs, query translation profile, derived-view lag policy, query class, node types, edge types, edge directions, temporal fields, authorization/redaction refs, expected query checksum, expected result checksum, and read-only mutation proof.

An `AnalysisRule` must not embed provider-native Cypher, AQL, SQL, SQL/Cypher composition, AGE `ag_catalog.cypher()` strings, prepared statement text, query-plan text, or other backend query text as production graph behavior unless `ValidateAnalysisQueryImport` maps the query into an active `090.GraphQueryTranslationProfile` row and `RuleGraphCompatibilityMatrix` proves read-only behavior, expected query checksum, expected result checksum, authorization/redaction behavior, and mutation prohibition. Provider-native query text is validation-only by default.

PostgreSQL and AGE compatibility rows must name `provider = postgresql` or `provider = postgresql_age` before a rule can target those profiles. Every PostgreSQL/AGE-compatible analysis rule must include an expected translated query checksum, expected result checksum, mutation-prohibition proof for SQL DML/DDL, mutation-prohibition proof for AGE mutating Cypher clauses, authorization refs, redaction refs, derived-view state refs, and `030.VersionManifest` refs. A rule embedding raw SQL, AGE Cypher, SQL/Cypher composition, prepared statement text, query-plan text, or provider-native query handles without those rows fails before analysis execution.

`observed_connection` is analysis-readable only when the compatibility matrix names `observed_connection_path` or a read-only detail query and the active output eligibility row permits that context. `generic_external_graph_payload` must not produce findings, metrics, identity influence, or pathfinding input.

### PostgreSQL and AGE analysis-query compatibility rows

A PostgreSQL-compatible analysis rule must use `provider = postgresql`. An AGE-compatible analysis rule must use `provider = postgresql_age`. Provider-native query text remains validation-only until every row below resolves with concrete refs and checksums.

| Compatibility row class | Required fields | Mutation-prohibition proof | Required downstream refs |
| --- | --- | --- | --- |
| PostgreSQL relational read-only query | `provider = postgresql`, `query_class`, active `090.GraphQueryTranslationProfile` ref/checksum, expected translated query checksum, expected result checksum, authorization refs, redaction refs, derived-view state refs, selected `030.VersionManifest` refs, validation refs | SQL DML, SQL DDL, transaction-control statements, unsafe `search_path`, provider-native query handles, prepared statement text, and literal-bearing query plans must be rejected before analysis execution | `090` translation refs, `110` redaction/API refs, `140` telemetry redaction refs, `120-GRAPH-POSTGRES-QUERY-*` fixtures |
| AGE read-only query | `provider = postgresql_age`, active AGE profile refs/checksums, AGE translation refs, expected translated query checksum, expected result checksum, authorization refs, redaction refs, derived-view state refs, selected `030.VersionManifest` refs, validation refs | Mutating Cypher clauses, AGE namespace DML/DDL, graph drop, unsafe `search_path`, SQL/Cypher mutation composition, AGE internal ID leakage, prepared statement text, and literal-bearing plans must be rejected before analysis execution | `090` AGE refs, `110` redaction/API refs, `140` telemetry redaction refs, `120-GRAPH-AGE-*` fixtures |

Raw SQL text, raw Cypher text, SQL/Cypher composition, query-plan text, and prepared statement text must remain validation-only diagnostics unless translated, checksummed, mutation-proven, redacted, authorized, accepted by `RuleGraphCompatibilityMatrix`, and included in `030.VersionManifest`.

#### RuleGraphCompatibilityMatrixPostgresClosure

PostgreSQL and AGE analysis-query compatibility is closed only through `RuleGraphCompatibilityMatrix` rows that import `090.GraphQueryTranslationProfile` by exact ref and checksum. PostgreSQL provider token must be exactly `postgresql`. AGE provider token must be exactly `postgresql_age`. Raw SQL, raw Cypher, SQL/Cypher composition, prepared statement text, and query-plan text remain validation-only unless mapped into a translated row.

| Field | Required behavior |
| --- | --- |
| `provider` | Exactly `postgresql` or `postgresql_age`. |
| `query_class` | One of the `090` query classes allowed for analysis read-only execution. |
| `graph_projection_profile_ref` | Active graph projection profile ref and checksum. |
| `graph_query_translation_profile_ref` | Active translation profile ref and checksum. |
| `expected_translated_query_checksum` | Required; raw SQL or Cypher bytes are not output identity. |
| `expected_result_checksum` | Required for deterministic parity. |
| `derived_view_state_ref` | Required when graph read model state affects analysis output. |
| `authorization_refs` | Required before execution. |
| `redaction_refs` | Required before execution and before telemetry. |
| `mutation_prohibition_proof` | Required for every SQL and AGE row. |
| `validation_refs` | Non-empty `120-GRAPH-POSTGRES-QUERY-*` or `120-GRAPH-AGE-*` refs. |
| `package_set_refs` | Required when the rule, query profile, or backend profile is package-supplied. |
| `version_manifest_refs` | Must include all selected rule, graph, query, authorization, redaction, validation, package, and expected checksum refs. |

Explicit rejection rows:

| Rejected input | Required result |
| --- | --- |
| SQL DML or DDL | Reject before execution with no mutation. |
| Transaction-control statements | Reject before execution with no mutation. |
| Unsafe `search_path` | Reject before execution. |
| Provider-native query handles | Reject before execution and before page-token creation. |
| Prepared statement text | Reject unless translated and checksummed through `090`. |
| Literal-bearing query plans | Reject or redact before validation, audit, and telemetry output. |
| AGE mutating Cypher clauses | Reject with no mutation. |
| AGE namespace DML or DDL | Reject with no mutation. |
| Graph drop | Reject with no mutation. |
| AGE internal ID leakage | Reject before API, audit, telemetry, evidence, page-token, or replay output. |

`AnalysisFinding`, `AnalysisMetric`, and `RiskAcceptanceRecord` are workflow outputs. They must not represent remediation, risk reduction, fact retraction, graph edge removal, or source completeness change.

### AnalysisReplayEquivalenceHandoff

`130` owns analysis-specific included and excluded fields for replay output class `080.ReplayEquivalencePolicy.output_class = analysis_output`. `080` owns checksum algorithm and replay preflight ordering.

| Analysis output subset | Included replay fields | Excluded volatile fields | Required validation |
| --- | --- | --- | --- |
| `analysis_finding` | `AnalysisRuleBundle` ref, `AnalysisRule` row refs, `RuleGraphCompatibilityMatrix` refs, query target refs, graph derived-view state refs, authorization/redaction refs, finding canonical output checksum, `VersionManifest` ref | request correlation ID, UI display label, runtime duration, non-output diagnostic ordering artifacts | exact replay, rule bundle mismatch, graph compatibility mismatch, authorization mismatch |
| `analysis_metric` | rule bundle ref, metric rule refs, query target refs, graph derived-view state refs, authorization/redaction refs, metric canonical output checksum, `VersionManifest` ref | request correlation ID, UI display label, runtime duration | exact replay and derived-view mismatch |
| `risk_acceptance_record` | acceptance workflow record checksum, authorization refs, related finding refs, redaction refs, `VersionManifest` ref | UI display label, request correlation ID, runtime duration | exact replay and authorization mismatch |
| `analysis_rule_execution_summary` | rule bundle ref, executed rule refs, graph profile refs, query target refs, canonical summary checksum, `VersionManifest` ref | non-output diagnostic ordering artifacts, runtime duration | exact replay and shadow-only comparison |

Changed output-affecting inputs reject production replay before output. Shadow-only comparison is allowed only when an active owner row permits shadow output and the result is not production-visible. Analysis replay must not mutate raw, silver, identity, gold, graph-delta, graph-serving, completeness, watermark, package, or source-authority state.

#### AnalysisReplayFieldSelectionRow

`AnalysisReplayFieldSelectionRow` is the exact owner-local row family imported by `080.ReplayEquivalencePolicyOutputClassRow` for `output_class = analysis_output`. A prose table is not sufficient for replay field selection.

| Field | Required behavior |
| --- | --- |
| `row_id` | Stable row ID scoped to the analysis replay field-selection row set. |
| `output_subset` | One of `analysis_finding`, `analysis_metric`, `risk_acceptance_record`, or `analysis_rule_execution_summary`. |
| `included_replay_fields` | Ordered sequence or canonical set of exact fields that affect analysis output. Wildcards are forbidden. |
| `excluded_volatile_fields` | Canonical set of volatile fields such as request correlation ID, UI display label, runtime duration, and non-output diagnostic ordering artifacts. |
| `required_validation_refs` | Non-empty refs for exact replay, rule-bundle mismatch, graph compatibility mismatch, authorization mismatch, volatile-only difference, and mutation prohibition. |
| `row_checksum` | SHA-256 over canonical row bytes after defaults, excluding only `row_checksum`. |
| `row_set_checksum` | SHA-256 over sorted row checksums and row-set metadata. |
| `activation_scope` | `030.ActivationScope`. |
| `lifecycle_status` | Production use requires `active`. |
| `version_manifest_requirements` | `030.VersionManifest` must include selected row ref, row checksum, row-set checksum, rule bundle refs, graph refs, authorization/redaction refs, validation refs, package-set refs when package-supplied, and output checksum refs. |

The `080.analysis_output` replay row must reference `AnalysisReplayFieldSelectionRow` by structured row ref and checksum. Missing, inactive, checksum-mismatched, package-set-mismatched, unmanifested, or TODO-bearing field-selection rows fail replay before analysis output.

## Derivation Rule Boundary

`DerivationRuleBundle` may emit `GoldFact` records only through the gold derivation interface owned by `080`. A derived graph edge rule may emit graph deltas only through the graph projection interface owned by `090`. Direct graph mutation is forbidden.

### DerivedGraphEdgeRule schema

`DerivedGraphEdgeRule` is the row-level interface for backend-generated, query-generated, post-processed, or generated path edges. Concrete row sets are activation-controlled artifacts. Stable graph projection and apply behavior remains owned by `090`; stable gold derivation behavior remains owned by `080`.

| Field | Required | Default or omission behavior | Rule |
| --- | ---: | --- | --- |
| `rule_id` | Yes | none | Stable ID scoped to the derived-edge rule row set. |
| `rule_version` | Yes | none | Immutable owner version included in output identity and replay checksums. |
| `input_fact_selector` | Yes | none | Names exact fact types and predicates. Wildcards are forbidden. |
| `required_gold_fact_predicate_contract_refs` | Required when `allowed_output_effect` is `graph_delta_via_090` or `gold_fact_via_080`; required for `analysis_only` when the result is pathfinding-visible or metric-visible over a fact-backed edge | none | Exact structured `030.ActivationControlledRowRef` values for active `080.GoldFactPredicateContractRow` rows. Fact/predicate strings are not substitutes. Missing, inactive, blocked, checksum-mismatched, or unmanifested refs emit explicit no-op or owner error with no graph, gold, metric-visible, pathfinding-visible, or watermark-affecting output. |
| `required_supporting_fact_refs_policy` | Required for graph-emitting rules | none | Non-empty for any rule whose output can affect graph output. Missing support fails with `DERIVED_GRAPH_EDGE_SUPPORTING_FACTS_REQUIRED`. |
| `required_source_dataset_catalog_row_refs` | Required when output is `graph_delta_via_090` or `gold_fact_via_080` and support is absence-sensitive, cleanup-sensitive, expiry-sensitive, retraction-sensitive, no-change, or watermark-affecting | none | Exact selected `020.SourceDatasetCatalogRow` refs or deterministic source-dataset block refs. Missing refs fail before analysis, gold, graph, metric, finding, or watermark output. |
| `required_source_authority_closure_matrix_refs` | Required when supporting facts affect output | none | Exact `060.SourceAuthorityClosureMatrixRow` refs or deterministic block row refs. |
| `required_absence_derivation_result_refs` | Required when absence-sensitive support affects output | none | Exact `060.AbsenceDerivationResult` refs for the requested effect. |
| `required_mutation_prohibition_refs` | Required for no-op/block rows | none | Refs proving no fact, graph delta, cleanup, watermark, pass/fail, no-change proof, or analysis-visible mutation was emitted. |
| `required_source_authority_refs` | Required when supporting facts affect output | none | Exact `060.SourceAuthorityProfileRow` refs. |
| `required_completeness_refs` | Required for negative, absence-sensitive, cleanup-sensitive, or expiry-sensitive output | none | Exact completeness, coverage, staleness, absence, and progress-signal refs required by `060`. |
| `required_temporal_refs` | Yes | none | Exact temporal resolution and replay refs required for deterministic replay. |
| `graph_projection_profile_ref` | Required for graph output | null only when output effect is not graph-emitting | Missing graph output ref fails with `DERIVED_GRAPH_EDGE_PROJECTION_FORBIDDEN`. |
| `graph_edge_semantics_ref` | Required for graph edge output | null only when no graph edge output is allowed | Must match `090.GraphEdgeSemantics`; mismatch fails before output. |
| `graph_object_output_eligibility_refs` | Required for analysis-visible output | none | Exact `090.GraphObjectOutputEligibilityRow` refs controlling search, pathfinding, finding, metric, and identity-influence contexts. |
| `deterministic_id_input_set` | Yes | none | Exact ordered list of rule refs, supporting fact refs, authority refs, completeness refs, temporal refs, and projection refs used for output identity. |
| `allowed_output_effect` | Yes | `analysis_only` when analysis output is requested; `no_output` otherwise | Closed enum: `analysis_only`, `graph_delta_via_090`, `gold_fact_via_080`, `no_output`. |
| `unsupported_behavior` | No | `explicit_no_op` | Unknown, unsupported, or unrepresentable rule outputs emit an explicit no-op unless the active row declares a stricter owner error. |
| `validation_refs` | Yes | none | Non-empty refs to `120` derived-edge positive, negative, no-op, replay, and routing rows. |
| `activation_scope` | Yes | none | `030.ActivationScope` in which the row may affect output. |
| `lifecycle_status` | Yes | none | Production use requires `active`. |

`allowed_output_effect` defaults to `analysis_only` or `no_output`. Direct graph mutation is forbidden. Gold fact output must route through `080`. Graph delta output must route through `090`. Unsupported or supportless generated edges remain analysis-only or no-op outputs.

A derived graph edge rule that consumes absence-sensitive, cleanup-sensitive, expiry-sensitive, retraction-sensitive, no-change, or watermark-affecting support must not emit analysis, gold, graph, metric, finding, or watermark-affecting output unless every supporting fact and source-effect row required by `060` is present, active, checksum-valid, scoped, validated, and manifest-included. Missing source-dataset catalog refs, source-authority closure refs, supporting fact refs, absence derivation result refs, mutation-prohibition refs, or `VersionManifest` refs must emit an explicit no-op or owner error with no graph mutation, no gold fact, and no watermark advancement.

A derived-edge rule must validate `required_gold_fact_predicate_contract_refs` before execution whenever it consumes or emits fact-backed output. The refs must match active `080` rows, be checksum-valid, lifecycle-active, package-set-valid when package-supplied, and included in `030.VersionManifest`. A rule must not become an alternate predicate activation mechanism by consuming fact-type and predicate strings without the selected row refs.

### AnalysisRegistryScopeSelectorContext

`AnalysisRegistryScopeSelectorContext` is the owner context family for derived-edge, analysis-rule, enrichment, lineage, and registry governance artifact activation. It instantiates `030.ScopeSelectorContext`; it does not grant fact, identity, source authority, package, graph apply, or watermark authority.

| Artifact family | Required dimensions | Optional dimensions | Default subset behavior | Permitted effect of scope match |
| --- | --- | --- | --- | --- |
| `DerivedGraphEdgeRule` | `artifact_family`, `rule_id`, `allowed_output_effect` | `graph_profile_id`, `fact_type`, `predicate` | none | Eligibility to route through the declared `allowed_output_effect` only. |
| `AnalysisRuleBundle` | `artifact_family`, `analysis_rule_bundle_ref` | `query_class`, `graph_profile_id` | none | Eligibility for read-only analysis execution only. |
| `ThreatIntelEnrichmentProfile` | `artifact_family`, `enrichment_profile_ref` | `source_category`, `source_dataset` | none | Eligibility for enrichment-only output only. |
| `LineageFacetMappingPolicy` | `artifact_family`, `facet_namespace` | `dataset_ref`, `schema_url` | none | Eligibility for lineage metadata mapping only. |
| `RegistryArtifactGovernance` | `artifact_family`, `registry_artifact_ref` | `domain`, `classification` | none | Eligibility for governance metadata activation only. |

Scope matching must not widen `allowed_output_effect`, bypass `090` projection, bypass `080` derivation, create source authority, mutate identity, activate packages, or advance watermarks.

### DerivedGraphEdgeRule effect routing

| `allowed_output_effect` | Required route | Forbidden output | Failure or no-op condition |
| --- | --- | --- | --- |
| `analysis_only` | `AnalysisFinding` or `AnalysisMetric` only. | `GoldFact`, `GraphDeltaSet`, backend write, source authority, identity. | Missing analysis validation refs fails before output. |
| `graph_delta_via_090` | `090.DerivedGraphEdgeRule graph handoff`. | Direct graph backend write or graph-serving mutation. | Missing supporting fact refs, required predicate row refs, or projection refs fails before delta persistence. |
| `gold_fact_via_080` | `080.DerivationRuleBundle handoff from 130`. | Package-emitted or analysis-emitted raw `GoldFact` bytes. | Missing required predicate row refs, temporal, authority, completeness, replay, or validation refs fails before fact ID computation. |
| `no_output` | Explicit no-op record or no visible output as declared by validation row. | Any fact, graph, identity, completeness, package, or watermark mutation. | Unsupported behavior defaults to `explicit_no_op`. |

`130` owns derived-edge rule activation and effect routing only; `080` owns gold fact derivation and `090` owns graph delta projection, graph delta identity, graph apply, query, rebuild, and backend behavior.

### SourceDatasetDerivedRuleClosureHandoff

A derived-edge or analysis rule whose support is absence-sensitive, cleanup-sensitive, expiry-sensitive, retraction-sensitive, no-change, or watermark-affecting must include selected `020.SourceDatasetCatalogRow` refs or deterministic source-dataset block refs, selected `060.SourceAuthorityClosureMatrixRow` refs or deterministic block refs, `060.AbsenceDerivationResult` refs for the requested effect, mutation-prohibition refs for blocked/no-op cases, and `030.VersionManifest` refs before producing findings, metrics, pathfinding-visible output, gold facts through `080`, graph deltas through `090`, or any caller-visible analysis output.

| Support family | Required analysis behavior |
| --- | --- |
| `network_flow` non-observation | Must not produce analysis-visible absence, risk finding, metric, graph delta, pathfinding input, cleanup, retraction, graph expiry, or watermark. Positive observed-flow analysis may read only authorized `observed_connection` output. |
| `source_history` no-change | Must not produce no-change findings, metrics, graph deltas, or gold output unless exact retention, coverage, source-authority closure, validation, package-set, and manifest refs exist. |
| `future_reachability` | Must not produce reachability facts, graph edges, graph properties, metrics, findings, pathfinding input, or unqualified reachability wording while `200` is inactive deferred. |
| deterministic source-dataset block | Must emit explicit no-op or owner error only and include mutation-prohibition proof. |

## Threat-Intel Enrichment

Threat-intel indicators, sightings, taxonomies, galaxies, object templates, confidence labels, distribution labels, sharing groups, and TLP-like classifications are enrichment context by default. They must be materialized through `ThreatIntelEnrichmentProfile`, `ThreatIntelEnrichmentRecord`, `ThreatIntelDistributionMappingPolicy`, and `ThreatIntelArtifactRef`.

They must not become identity, source completeness, source authority, gold fact, graph edge, graph serving, absence, cleanup, retraction, coverage, or watermark authority by themselves.

### ThreatIntelEnrichmentProfile schema

`ThreatIntelEnrichmentProfile` is an activation-controlled artifact. It permits `ThreatIntelEnrichmentRecord` emission only. It must not grant fact, identity, source-authority, source-completeness, graph, package, coverage, absence, cleanup, retraction, watermark, production-approval, validation-acceptance, or remediation effects.

The table below is the complete `030.ActivationControlledRowField` interface for `ThreatIntelEnrichmentProfile`. The field table narrows the shared `AnalysisRegistryActivationControlledRowFieldRule` defaults and all fields participate in the row checksum unless explicitly stated otherwise.

| field_path | type | required | default | null_allowed | omit_allowed | bounds | array_semantics | duplicate_policy | canonical_sort_key | id_input | checksum_input | extension_policy | redaction_owner | version_manifest_requirement | missing_error | invalid_error |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `profile_id` | `040.ScalarType.string` | `yes` | `none` | `no` | `no` | non-empty stable ID scoped to `130` | `n/a` | `reject` | `n/a` | `ordered:1` | `yes` | `closed` | `110` | selected profile ref and checksum required | `THREAT_INTEL_PROFILE_MISSING` | `REGISTRY_ARTIFACT_OWNER_MISMATCH` |
| `profile_version` | `040.ScalarType.string` | `yes` | `none` | `no` | `no` | immutable owner version | `n/a` | `reject` | `n/a` | `ordered:2` | `yes` | `closed` | `110` | profile version and row checksum required | `THREAT_INTEL_PROFILE_MISSING` | `REGISTRY_ARTIFACT_OWNER_MISMATCH` |
| `source_format` | `130.ThreatIntelSourceFormat` | `yes` | `none` | `no` | `no` | closed token declared by active profile row set | `n/a` | `reject` | `n/a` | `ordered:3` | `yes` | `closed` | `110` | selected profile row required before enrichment | `THREAT_INTEL_PROFILE_MISSING` | `THREAT_INTEL_PROFILE_MISSING` |
| `immutable_artifact_refs` | `array<130.ThreatIntelArtifactRef>` | `yes` | `none` | `no` | `no` | non-empty immutable refs with checksums | `canonical_set` | `reject` | `artifact_ref` | `ordered:4` | `yes` | `closed` | `110` | artifact refs and checksums required | `THREAT_INTEL_PROFILE_MISSING` | `THREAT_INTEL_ARTIFACT_CHECKSUM_MISMATCH` |
| `indicator_normalization_policy_ref` | `030.ActivationControlledArtifactRef` | `yes` | `none` | `no` | `no` | active owner artifact ref | `n/a` | `reject` | `n/a` | `ordered:5` | `yes` | `closed` | `110` | policy ref and checksum required | `THREAT_INTEL_PROFILE_MISSING` | `REGISTRY_VOLATILITY_BOUNDARY_VIOLATION` |
| `object_template_refs` | `array<030.ActivationControlledArtifactRef>` | `no` | `[]` | `no` | `yes` | immutable refs when object templates are used | `canonical_set` | `reject` | `artifact_ref` | `no` | `yes` | `closed` | `110` | selected refs required when context uses templates | `THREAT_INTEL_PROFILE_MISSING` | `REGISTRY_VOLATILITY_BOUNDARY_VIOLATION` |
| `taxonomy_refs` | `array<030.ActivationControlledArtifactRef>` | `no` | `[]` | `no` | `yes` | immutable refs when taxonomies are used | `canonical_set` | `reject` | `artifact_ref` | `no` | `yes` | `closed` | `110` | selected refs required when taxonomy context exists | `THREAT_INTEL_PROFILE_MISSING` | `REGISTRY_VOLATILITY_BOUNDARY_VIOLATION` |
| `galaxy_refs` | `array<030.ActivationControlledArtifactRef>` | `no` | `[]` | `no` | `yes` | immutable refs when galaxies are used | `canonical_set` | `reject` | `artifact_ref` | `no` | `yes` | `closed` | `110` | selected refs required when galaxy context exists | `THREAT_INTEL_PROFILE_MISSING` | `REGISTRY_VOLATILITY_BOUNDARY_VIOLATION` |
| `sighting_policy` | `130.ThreatIntelSightingPolicy` | `yes` | `observability_only` | `no` | `no` | only MVP value is `observability_only` | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | policy value included when sightings affect output | `THREAT_INTEL_PROFILE_MISSING` | `REGISTRY_VOLATILITY_BOUNDARY_VIOLATION` |
| `distribution_policy_ref` | `030.ActivationControlledArtifactRef` | `yes` | `none` | `no` | `no` | active `ThreatIntelDistributionMappingPolicy` ref | `n/a` | `reject` | `n/a` | `ordered:6` | `yes` | `closed` | `110` | selected distribution policy ref/checksum required | `THREAT_INTEL_PROFILE_MISSING` | `THREAT_INTEL_DISTRIBUTION_UNMAPPED` |
| `unknown_value_behavior` | `130.ThreatIntelUnknownValueBehavior` | `yes` | `preserve_as_unknown_enrichment_context` | `no` | `no` | `preserve_as_unknown_enrichment_context` or `reject_profile_row` | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | behavior ref included when unknown values exist | `THREAT_INTEL_PROFILE_MISSING` | `REGISTRY_VOLATILITY_BOUNDARY_VIOLATION` |
| `deprecated_artifact_behavior` | `130.ThreatIntelDeprecatedArtifactBehavior` | `yes` | `reject_for_production` | `no` | `no` | production default and only MVP production value is `reject_for_production` | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | lifecycle evidence required for deprecated artifact rejection | `THREAT_INTEL_PROFILE_MISSING` | `REGISTRY_ARTIFACT_INACTIVE` |
| `allowed_output_record_classes` | `array<record_class_token>` | `yes` | `[ThreatIntelEnrichmentRecord]` | `no` | `no` | default and only MVP value is `ThreatIntelEnrichmentRecord` | `canonical_set` | `reject` | `record_class` | `no` | `yes` | `closed` | `110` | selected output class included in manifest | `THREAT_INTEL_PROFILE_MISSING` | `ANALYSIS_MUTATION_FORBIDDEN` |
| `graph_effect` | `130.AnalysisEffect` | `yes` | `none` | `no` | `no` | default and only MVP value is `none` | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | graph-effect no-op evidence required when attempted | `THREAT_INTEL_PROFILE_MISSING` | `REGISTRY_VOLATILITY_BOUNDARY_VIOLATION` |
| `identity_effect` | `130.AnalysisEffect` | `yes` | `none` | `no` | `no` | default and only MVP value is `none` | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | identity-effect no-op evidence required when attempted | `THREAT_INTEL_PROFILE_MISSING` | `THREAT_INTEL_IDENTITY_FORBIDDEN` |
| `authority_effect` | `130.AnalysisEffect` | `yes` | `none` | `no` | `no` | default and only MVP value is `none` | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | authority-effect no-op evidence required when attempted | `THREAT_INTEL_PROFILE_MISSING` | `REGISTRY_VOLATILITY_BOUNDARY_VIOLATION` |
| `redaction_policy_ref` | `030.ActivationControlledArtifactRef` | `yes` | `none` | `no` | `no` | active `110.RedactionPolicy` ref | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | redaction ref/checksum required before storage or visibility | `THREAT_INTEL_PROFILE_MISSING` | `REGISTRY_VOLATILITY_BOUNDARY_VIOLATION` |
| `validation_refs` | `array<030.ActivationControlledArtifactRef>` | `yes` | `none` | `no` | `no` | non-empty `120` threat-intel positive, rejection, distribution, redaction, non-authority, replay, package, and manifest rows | `canonical_set` | `reject` | `artifact_ref` | `no` | `yes` | `closed` | `110` | every validation ref and checksum required | `THREAT_INTEL_PROFILE_MISSING` | `REGISTRY_VOLATILITY_BOUNDARY_VIOLATION` |
| `activation_scope` | `030.ActivationScope` | `yes` | `none` | `no` | `no` | must cover request scope through `130.AnalysisRegistryScopeSelectorContext` | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | selector context and checksum required | `THREAT_INTEL_PROFILE_MISSING` | `REGISTRY_ARTIFACT_OWNER_MISMATCH` |
| `lifecycle_status` | `030.LifecycleStatus` | `yes` | `none` | `no` | `no` | production use requires `active` | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | lifecycle transition evidence required | `REGISTRY_ARTIFACT_INACTIVE` | `REGISTRY_ARTIFACT_INACTIVE` |
| `package_set_ref` | `030.ActivationControlledArtifactRef` | `conditional:package-supplied profile` | `null` | `yes` | `no` | required when package-supplied; null otherwise | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | package-set ref/checksum required when package-supplied | `THREAT_INTEL_PROFILE_MISSING` | `REGISTRY_VOLATILITY_BOUNDARY_VIOLATION` |
| `version_manifest_requirement` | `130.VersionManifestRequirement` | `yes` | `selected_profile_artifact_distribution_redaction_validation_package_lifecycle_output_refs` | `no` | `no` | output-affecting refs listed in `030.VersionManifestCompletenessMatrix` | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | manifest refs required before output | `THREAT_INTEL_PROFILE_MISSING` | `REGISTRY_VOLATILITY_BOUNDARY_VIOLATION` |

Unknown `source_format` values must fail with `THREAT_INTEL_PROFILE_MISSING`. Ref arrays are canonical sets, reject duplicates, sort by immutable ref ID, and participate in the row checksum. Indicator equality, sightings, taxonomies, galaxies, object templates, confidence labels, and distribution labels remain enrichment context only unless another active owner spec exports an exact named interface.

### ThreatIntelEnrichmentRecord schema

`ThreatIntelEnrichmentRecord` is a `runtime_state_record_schema`, not an activation-controlled row. It is emitted by enrichment execution after the selected `ThreatIntelEnrichmentProfile`, artifact refs, distribution mapping policy, redaction policy, validation refs, package-set refs when package-supplied, lifecycle refs, and `030.VersionManifest` refs validate.

| field_path | type | required | default | null_allowed | omit_allowed | bounds | canonical_sort_key | id_input | checksum_input | extension_policy | redaction_owner | missing_error | invalid_error |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `record_id` | `040.ScalarType.string` | `yes` | `derived:ComputeThreatIntelEnrichmentRecordId` | `no` | `no` | deterministic Cadastre ID | `n/a` | `derived` | `yes` | `closed` | `110` | `THREAT_INTEL_PROFILE_MISSING` | `THREAT_INTEL_ARTIFACT_CHECKSUM_MISMATCH` |
| `profile_ref` | `030.ActivationControlledArtifactRef` | `yes` | `none` | `no` | `no` | active `ThreatIntelEnrichmentProfile` ref | `n/a` | `ordered:1` | `yes` | `closed` | `110` | `THREAT_INTEL_PROFILE_MISSING` | `THREAT_INTEL_PROFILE_MISSING` |
| `profile_checksum` | `040.ScalarType.sha256` | `yes` | `none` | `no` | `no` | lowercase SHA-256 hex | `n/a` | `ordered:2` | `yes` | `closed` | `110` | `THREAT_INTEL_PROFILE_MISSING` | `THREAT_INTEL_PROFILE_MISSING` |
| `artifact_refs` | `array<130.ThreatIntelArtifactRef>` | `yes` | `none` | `no` | `no` | non-empty immutable refs with checksums | `artifact_ref` | `ordered:3` | `yes` | `closed` | `110` | `THREAT_INTEL_PROFILE_MISSING` | `THREAT_INTEL_ARTIFACT_CHECKSUM_MISMATCH` |
| `indicator_context` | `map` | `yes` | `{}` | `no` | `no` | bounded by selected profile and redaction policy | lexical key order | `ordered:4` | `yes` | `closed` | `110` | `THREAT_INTEL_PROFILE_MISSING` | `THREAT_INTEL_IDENTITY_FORBIDDEN` |
| `sighting_context` | `map` | `yes` | `{}` | `no` | `no` | observability-only context | lexical key order | `ordered:5` | `yes` | `closed` | `110` | `THREAT_INTEL_PROFILE_MISSING` | `REGISTRY_VOLATILITY_BOUNDARY_VIOLATION` |
| `taxonomy_context` | `map` | `yes` | `{}` | `no` | `no` | bounded by selected taxonomy refs | lexical key order | `ordered:6` | `yes` | `closed` | `110` | `THREAT_INTEL_PROFILE_MISSING` | `REGISTRY_VOLATILITY_BOUNDARY_VIOLATION` |
| `galaxy_context` | `map` | `yes` | `{}` | `no` | `no` | bounded by selected galaxy refs | lexical key order | `ordered:7` | `yes` | `closed` | `110` | `THREAT_INTEL_PROFILE_MISSING` | `REGISTRY_VOLATILITY_BOUNDARY_VIOLATION` |
| `object_template_context` | `map` | `yes` | `{}` | `no` | `no` | bounded by selected object template refs | lexical key order | `ordered:8` | `yes` | `closed` | `110` | `THREAT_INTEL_PROFILE_MISSING` | `REGISTRY_VOLATILITY_BOUNDARY_VIOLATION` |
| `distribution_label` | `130.ThreatIntelNormalizedDistributionLabel` | `yes` | `restricted_visibility when source label missing` | `no` | `no` | selected by `ResolveThreatIntelDistribution` | `n/a` | `ordered:9` | `yes` | `closed` | `110` | `THREAT_INTEL_DISTRIBUTION_UNMAPPED` | `THREAT_INTEL_DISTRIBUTION_UNMAPPED` |
| `distribution_policy_ref` | `030.ActivationControlledArtifactRef` | `yes` | `none` | `no` | `no` | active mapping policy ref | `n/a` | `ordered:10` | `yes` | `closed` | `110` | `THREAT_INTEL_PROFILE_MISSING` | `THREAT_INTEL_DISTRIBUTION_UNMAPPED` |
| `visibility_class` | `130.VisibilityClass` | `yes` | `derived:ResolveThreatIntelDistribution` | `no` | `no` | closed visibility token; missing source label produces restricted visibility | `n/a` | `ordered:11` | `yes` | `closed` | `110` | `THREAT_INTEL_DISTRIBUTION_UNMAPPED` | `REGISTRY_VOLATILITY_BOUNDARY_VIOLATION` |
| `redaction_policy_ref` | `030.ActivationControlledArtifactRef` | `yes` | `none` | `no` | `no` | active redaction policy ref | `n/a` | `ordered:12` | `yes` | `closed` | `110` | `THREAT_INTEL_PROFILE_MISSING` | `REGISTRY_VOLATILITY_BOUNDARY_VIOLATION` |
| `authority_class` | `010.AuthorityClass` | `yes` | `non_authoritative_analysis` | `no` | `no` | default and only MVP value is `non_authoritative_analysis` | `n/a` | `no` | `yes` | `closed` | `110` | `THREAT_INTEL_PROFILE_MISSING` | `REGISTRY_VOLATILITY_BOUNDARY_VIOLATION` |
| `canonical_enrichment_payload_checksum` | `040.ScalarType.sha256` | `yes` | `derived:canonical context bytes` | `no` | `no` | lowercase SHA-256 hex | `n/a` | `ordered:13` | `yes` | `closed` | `110` | `THREAT_INTEL_PROFILE_MISSING` | `THREAT_INTEL_ARTIFACT_CHECKSUM_MISMATCH` |
| `output_checksum` | `040.ScalarType.sha256` | `yes` | `derived:record canonical bytes excluding output checksum` | `no` | `no` | lowercase SHA-256 hex | `n/a` | `no` | `no` | `closed` | `110` | `THREAT_INTEL_PROFILE_MISSING` | `THREAT_INTEL_ARTIFACT_CHECKSUM_MISMATCH` |
| `version_manifest_ref` | `030.VersionManifest` | `yes` | `none` | `no` | `no` | manifest must contain profile, artifacts, distribution, redaction, validation, package, lifecycle, and output checksum refs | `n/a` | `ordered:14` | `yes` | `closed` | `110` | `THREAT_INTEL_PROFILE_MISSING` | `REGISTRY_VOLATILITY_BOUNDARY_VIOLATION` |

`ComputeThreatIntelEnrichmentRecordId` must be deterministic over profile ref, profile checksum, artifact refs and checksums, normalized context bytes, distribution policy ref, selected distribution result, redaction refs, and `canonical_enrichment_payload_checksum`. A missing source distribution label materializes `distribution_label = restricted_visibility`, sets `visibility_class = restricted_visibility`, sets export permission to false, and records distribution omission diagnostics. This default disables export.

### ThreatIntelDistributionMappingPolicy

`ThreatIntelDistributionMappingPolicy` is an activation-controlled row set. It maps external distribution, sharing-group, and TLP-like labels to Cadastre visibility and export behavior. It must not grant fact, identity, source-authority, source-completeness, graph, package, watermark, or production-approval authority.

| field_path | type | required | default | null_allowed | omit_allowed | bounds | array_semantics | duplicate_policy | canonical_sort_key | id_input | checksum_input | extension_policy | redaction_owner | version_manifest_requirement | missing_error | invalid_error |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `policy_id` | `040.ScalarType.string` | `yes` | `none` | `no` | `no` | stable policy ID scoped to `130` | `n/a` | `reject` | `n/a` | `ordered:1` | `yes` | `closed` | `110` | selected policy ref/checksum required | `THREAT_INTEL_DISTRIBUTION_UNMAPPED` | `THREAT_INTEL_DISTRIBUTION_UNMAPPED` |
| `policy_version` | `040.ScalarType.string` | `yes` | `none` | `no` | `no` | immutable owner version | `n/a` | `reject` | `n/a` | `ordered:2` | `yes` | `closed` | `110` | selected version required | `THREAT_INTEL_DISTRIBUTION_UNMAPPED` | `THREAT_INTEL_DISTRIBUTION_UNMAPPED` |
| `external_label_namespace` | `040.ScalarType.string` | `yes` | `none` | `no` | `no` | non-empty lower-snake-case or externally pinned namespace token | `n/a` | `reject` | `n/a` | `ordered:3` | `yes` | `closed` | `110` | namespace ref included in mapping checksum | `THREAT_INTEL_DISTRIBUTION_UNMAPPED` | `THREAT_INTEL_DISTRIBUTION_UNMAPPED` |
| `external_label_value` | `040.ScalarType.string` | `yes` | `none` | `no` | `no` | exact external label value after declared normalization; string similarity forbidden | `n/a` | `reject` | `n/a` | `ordered:4` | `yes` | `closed` | `110` | selected label row required when label exists | `THREAT_INTEL_DISTRIBUTION_UNMAPPED` | `THREAT_INTEL_DISTRIBUTION_UNMAPPED` |
| `normalized_label` | `130.ThreatIntelNormalizedDistributionLabel` | `yes` | `none` | `no` | `no` | closed normalized label token | `n/a` | `reject` | `n/a` | `ordered:5` | `yes` | `closed` | `110` | selected normalized label required | `THREAT_INTEL_DISTRIBUTION_UNMAPPED` | `THREAT_INTEL_DISTRIBUTION_UNMAPPED` |
| `restrictiveness_rank` | `040.ScalarType.integer` | `yes` | `none` | `no` | `no` | integer `0..1000`; lower is more restrictive | `n/a` | `reject` | `n/a` | `ordered:6` | `yes` | `closed` | `110` | selected rank required for conflicts | `THREAT_INTEL_DISTRIBUTION_UNMAPPED` | `THREAT_INTEL_DISTRIBUTION_UNMAPPED` |
| `visibility_class` | `130.VisibilityClass` | `yes` | `none` | `no` | `no` | closed visibility token | `n/a` | `reject` | `n/a` | `ordered:7` | `yes` | `closed` | `110` | visibility class included before output | `THREAT_INTEL_DISTRIBUTION_UNMAPPED` | `REGISTRY_VOLATILITY_BOUNDARY_VIOLATION` |
| `export_allowed` | `boolean` | `yes` | `false when source label missing or unknown preserved` | `no` | `no` | boolean | `n/a` | `reject` | `n/a` | `ordered:8` | `yes` | `closed` | `110` | export decision and policy ref required before export | `THREAT_INTEL_DISTRIBUTION_UNMAPPED` | `REGISTRY_VOLATILITY_BOUNDARY_VIOLATION` |
| `unknown_label_behavior` | `130.UnknownDistributionLabelBehavior` | `yes` | `reject` | `no` | `no` | `reject` or `preserve_redacted_restricted` | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | behavior required before unknown label storage | `THREAT_INTEL_DISTRIBUTION_UNMAPPED` | `THREAT_INTEL_DISTRIBUTION_UNMAPPED` |
| `conflict_behavior` | `130.DistributionConflictBehavior` | `yes` | `most_restrictive_visibility_wins` | `no` | `no` | only MVP value is `most_restrictive_visibility_wins` | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | conflict behavior included in output checksum | `THREAT_INTEL_DISTRIBUTION_UNMAPPED` | `THREAT_INTEL_DISTRIBUTION_UNMAPPED` |
| `raw_label_exposure` | `130.RawLabelExposurePolicy` | `yes` | `redacted_context_only` | `no` | `no` | raw labels never caller-visible by default | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | redaction refs required when raw labels retained | `THREAT_INTEL_DISTRIBUTION_UNMAPPED` | `REGISTRY_VOLATILITY_BOUNDARY_VIOLATION` |
| `redaction_policy_ref` | `030.ActivationControlledArtifactRef` | `yes` | `none` | `no` | `no` | active redaction policy ref | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | redaction ref/checksum required | `THREAT_INTEL_DISTRIBUTION_UNMAPPED` | `REGISTRY_VOLATILITY_BOUNDARY_VIOLATION` |
| `validation_refs` | `array<030.ActivationControlledArtifactRef>` | `yes` | `none` | `no` | `no` | non-empty `120-THREAT-INTEL-DISTRIBUTION-*` refs | `canonical_set` | `reject` | `artifact_ref` | `no` | `yes` | `closed` | `110` | validation refs/checksums required | `THREAT_INTEL_DISTRIBUTION_UNMAPPED` | `REGISTRY_VOLATILITY_BOUNDARY_VIOLATION` |
| `activation_scope` | `030.ActivationScope` | `yes` | `none` | `no` | `no` | must cover enrichment request scope | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | selector context and checksum required | `THREAT_INTEL_DISTRIBUTION_UNMAPPED` | `REGISTRY_ARTIFACT_OWNER_MISMATCH` |
| `lifecycle_status` | `030.LifecycleStatus` | `yes` | `none` | `no` | `no` | production selection requires `active` | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | lifecycle transition evidence required | `REGISTRY_ARTIFACT_INACTIVE` | `REGISTRY_ARTIFACT_INACTIVE` |

```text
ResolveThreatIntelDistribution(input_labels, policy_row_set, profile):
1. Validate active policy row-set ref, row checksum, package-set ref when package-supplied, lifecycle status, and validation refs.
2. If no label is present, emit `visibility_class = restricted_visibility`, `export_allowed = false`, and `distribution_omitted = true`.
3. Normalize all labels through exact policy rows; string similarity is forbidden.
4. If any label is unknown and policy `unknown_label_behavior = reject`, emit `THREAT_INTEL_DISTRIBUTION_UNMAPPED`.
5. If unknown labels are preserved, store only redacted unknown context, set restricted visibility, and disable export.
6. If more than one known label exists, choose the lowest `restrictiveness_rank`; lower is more restrictive.
7. Return visibility class, export permission, selected label refs, diagnostics, and checksum.
```

### ThreatIntelArtifactRef

`ThreatIntelArtifactRef` is the immutable artifact identity contract for imported threat-intel source artifacts used by a `ThreatIntelEnrichmentProfile`.

| field_path | type | required | default | null_allowed | omit_allowed | bounds | array_semantics | duplicate_policy | canonical_sort_key | id_input | checksum_input | extension_policy | redaction_owner | version_manifest_requirement | missing_error | invalid_error |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `artifact_ref` | `030.ActivationControlledArtifactRef` | `yes` | `none` | `no` | `no` | immutable ref only; branches, tags, `latest`, and mutable URLs forbidden | `n/a` | `reject` | `n/a` | `ordered:1` | `yes` | `closed` | `110` | artifact ref required in manifest before enrichment | `THREAT_INTEL_PROFILE_MISSING` | `THREAT_INTEL_ARTIFACT_CHECKSUM_MISMATCH` |
| `artifact_format` | `130.ThreatIntelSourceFormat` | `yes` | `none` | `no` | `no` | closed token declared by active profile | `n/a` | `reject` | `n/a` | `ordered:2` | `yes` | `closed` | `110` | format required before profile match | `THREAT_INTEL_PROFILE_MISSING` | `THREAT_INTEL_PROFILE_MISSING` |
| `artifact_version` | `040.ScalarType.string` | `yes` | `none` | `no` | `no` | immutable version or checksum-addressed version | `n/a` | `reject` | `n/a` | `ordered:3` | `yes` | `closed` | `110` | version required in manifest | `THREAT_INTEL_PROFILE_MISSING` | `THREAT_INTEL_ARTIFACT_CHECKSUM_MISMATCH` |
| `artifact_bytes_checksum` | `040.ScalarType.sha256` | `yes` | `none` | `no` | `no` | lowercase SHA-256 over exact artifact bytes | `n/a` | `reject` | `n/a` | `ordered:4` | `yes` | `closed` | `110` | checksum required before enrichment output | `THREAT_INTEL_PROFILE_MISSING` | `THREAT_INTEL_ARTIFACT_CHECKSUM_MISMATCH` |
| `artifact_schema_ref` | `030.ActivationControlledArtifactRef` | `conditional:schema validation used` | `null` | `yes` | `no` | immutable schema ref; null means no schema validation is declared by the profile | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | schema ref/checksum required when profile uses schema validation | `THREAT_INTEL_PROFILE_MISSING` | `THREAT_INTEL_ARTIFACT_CHECKSUM_MISMATCH` |
| `artifact_source_metadata_checksum` | `040.ScalarType.sha256` | `yes` | `none` | `no` | `no` | SHA-256 over owner-declared canonical source metadata bytes | `n/a` | `reject` | `n/a` | `ordered:5` | `yes` | `closed` | `110` | metadata checksum required before output | `THREAT_INTEL_PROFILE_MISSING` | `THREAT_INTEL_ARTIFACT_CHECKSUM_MISMATCH` |
| `redaction_policy_ref` | `030.ActivationControlledArtifactRef` | `yes` | `none` | `no` | `no` | active `110.RedactionPolicy` ref | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | redaction ref/checksum required before raw-derived storage | `THREAT_INTEL_PROFILE_MISSING` | `REGISTRY_VOLATILITY_BOUNDARY_VIOLATION` |
| `validation_refs` | `array<030.ActivationControlledArtifactRef>` | `yes` | `none` | `no` | `no` | non-empty checksum, deprecation, unknown-value, redaction, and non-authority refs | `canonical_set` | `reject` | `artifact_ref` | `no` | `yes` | `closed` | `110` | validation refs/checksums required | `THREAT_INTEL_PROFILE_MISSING` | `REGISTRY_VOLATILITY_BOUNDARY_VIOLATION` |
| `activation_scope` | `030.ActivationScope` | `yes` | `none` | `no` | `no` | must cover enrichment request scope | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | scope selector refs required | `THREAT_INTEL_PROFILE_MISSING` | `REGISTRY_ARTIFACT_OWNER_MISMATCH` |
| `lifecycle_status` | `030.LifecycleStatus` | `yes` | `none` | `no` | `no` | production use requires `active`; deprecated production use rejects | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | lifecycle evidence required | `REGISTRY_ARTIFACT_INACTIVE` | `REGISTRY_ARTIFACT_INACTIVE` |
| `package_set_ref` | `030.ActivationControlledArtifactRef` | `conditional:package-supplied artifact` | `null` | `yes` | `no` | required when package-supplied and null otherwise | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | package-set ref/checksum required when package-supplied | `THREAT_INTEL_PROFILE_MISSING` | `REGISTRY_VOLATILITY_BOUNDARY_VIOLATION` |

Mutable refs, branches, tags, `latest`, missing checksums, and deprecated artifact production use must reject before enrichment output. A `ThreatIntelArtifactRef` must not create an `EvidenceRef.artifact_id.kind`; evidence eligibility remains owned by `040.EvidenceArtifactClassRegistry`.

## Lineage and Artifact Boundary

`RunDatasetIOContract` and `LineageFacetMappingPolicy` may map external lineage facets to Cadastre lineage fields, diagnostic fields, non-authoritative metadata, or rejection. External lineage events and facets are non-authoritative by default.

`ArtifactClassPolicy` must prevent static DAG artifacts, executed-run artifacts, freshness artifacts, semantic artifacts, validation artifacts, lineage artifacts, table snapshot artifacts, table commit artifacts, and graph rebuild artifacts from substituting for one another.

### EvidenceArtifactSubstitutionHandoff

`ArtifactClassPolicy` rows may classify and constrain artifacts, but they must not add `EvidenceRef.artifact_id` kinds or authorize artifact substitution. `EvidenceRef` eligibility requires the referenced artifact class to match one `040.EvidenceArtifactClassRegistry` row and the referenced owner artifact to satisfy its owner checksum, lifecycle, redaction, and `VersionManifest` rules.

Lineage facets cannot substitute for evidence refs unless an owner row maps them to a specific non-authoritative artifact class. Registry governance artifacts cannot define evidence refs by themselves. Analysis findings and metrics may reference evidence metadata but must not become source evidence, source completeness, source authority, gold facts, graph truth, package activation, or validation acceptance.

Artifact substitution failures must use owner-specific errors when available and must be covered by `120` volatility-boundary and evidence-ref fixture rows.

## Registry Governance

`RegistryArtifactGovernance`, `RegistryCustomPropertySchema`, and `RegistryClassificationPolicy` manage owners, domains, classifications, glossary labels, policies, approval, lifecycle, custom properties, and checksums. Registry metadata must not define Cadastre fact authority, source authority, graph edge semantics, evidence refs, source completeness, or production approval by itself.

### StructuredInputRepositoryRegistryHandoff

Repository-authored registry, analysis, lineage, threat-intel, governance, custom-property, classification, and derived-edge artifacts are inert until active owner refs, exact-snapshot validation refs, materialization refs when packaged, package-set refs when package-supplied, and `030.VersionManifest` refs exist.

Repository approval metadata is governance metadata only. It must not become production approval, fact authority, source authority, graph authority, source completeness, evidence refs, validation acceptance, package activation authority, or rollback eligibility.

`ArtifactClassPolicy` must reject a repository snapshot, validation report, governance approval, branch name, tag, hook result, pull request approval, or merge event as a substitute for package release, source evidence, validation acceptance, source completeness, graph rebuild evidence, owner row set, or production approval.

Repository-authored analysis and registry activation must also include repository template contract refs when required, producer CI exact-snapshot validation refs when accepted, maintenance tool invocation refs when artifacts are generated, publication manifest refs when remote publication is consumed, candidate sync record refs when imported by sync, and repository group refs when cross-repository analysis artifacts depend on graph, resolver, source-authority, mapping, or registry artifacts.

The following signals have no production effect by themselves:

| Signal | Required no-effect behavior |
| --- | --- |
| governance approval metadata as production approval | No package activation, owner row activation, source authority, graph authority, or finding authority. |
| template conformance as registry activation | No registry artifact becomes selectable. |
| sync record as analysis execution permission | No analysis rule, enrichment rule, lineage row, registry row, derived edge, finding, or metric executes. |
| publication manifest as finding authority | No finding, metric, risk acceptance, fact, graph delta, or registry authority is emitted. |

| Error code | Required use |
| --- | --- |
| `REGISTRY_REPOSITORY_ARTIFACT_INACTIVE` | Repository-authored registry or analysis artifact lacks active materialized owner refs, validation refs, package-set refs when package-supplied, or manifest refs. |
| `REGISTRY_REPOSITORY_AUTHORITY_FORBIDDEN` | Repository approval metadata, branch state, validation report, or governance label attempts to become fact, source, graph, completeness, evidence, package, or production approval authority. |
| `REGISTRY_REPOSITORY_SUBSTITUTION_FORBIDDEN` | Repository artifact attempts to substitute for package artifact, validation acceptance, source evidence, owner row set, graph rebuild evidence, or production approval. |
| `REGISTRY_REPOSITORY_TEMPLATE_MISMATCH` | Repository-authored registry or analysis artifact layout, generated output roots, or artifact classes do not match the selected template contract. |
| `REGISTRY_REPOSITORY_CI_STALE` | Producer CI evidence is stale or not exact-snapshot-bound. |
| `REGISTRY_REPOSITORY_PUBLICATION_MANIFEST_MISMATCH` | Publication manifest does not match registry or analysis artifact digest, materialization refs, validation refs, package type, or release refs. |
| `REGISTRY_REPOSITORY_SYNC_NONAUTHORITY` | Candidate sync record is used as registry activation, analysis execution permission, finding authority, package activation, or production approval. |
| `REGISTRY_REPOSITORY_GROUP_MISMATCH` | Cross-repository dependency coherence fails for analysis or registry artifacts. |

### RegistryArtifactGovernance schema

`RegistryArtifactGovernance` is an activation-controlled artifact. It records governance metadata for a production-active registry artifact. It must not become production approval, source authority, source completeness, identity authority, fact authority, graph authority, package activation authority, validation acceptance, or watermark authority by itself.

| field_path | type | required | default | null_allowed | omit_allowed | bounds | array_semantics | duplicate_policy | canonical_sort_key | id_input | checksum_input | extension_policy | redaction_owner | version_manifest_requirement | missing_error | invalid_error |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `governance_id` | `040.ScalarType.string` | `yes` | `none` | `no` | `no` | stable governance ID scoped to `130` | `n/a` | `reject` | `n/a` | `ordered:1` | `yes` | `closed` | `110` | selected governance ref/checksum required | `REGISTRY_ARTIFACT_INACTIVE` | `REGISTRY_ARTIFACT_OWNER_MISMATCH` |
| `governance_version` | `040.ScalarType.string` | `yes` | `none` | `no` | `no` | immutable owner version | `n/a` | `reject` | `n/a` | `ordered:2` | `yes` | `closed` | `110` | version required | `REGISTRY_ARTIFACT_INACTIVE` | `REGISTRY_ARTIFACT_OWNER_MISMATCH` |
| `owner_spec` | `owner_spec_token` | `yes` | `none` | `no` | `no` | exact owner spec that owns the artifact behavior | `n/a` | `reject` | `n/a` | `ordered:3` | `yes` | `closed` | `110` | owner spec included in manifest | `REGISTRY_ARTIFACT_OWNER_MISMATCH` | `REGISTRY_ARTIFACT_OWNER_MISMATCH` |
| `artifact_class` | `040.EvidenceArtifactClassRegistry.artifact_class` | `yes` | `none` | `no` | `no` | class must validate through `040` when evidence refs are emitted | `n/a` | `reject` | `n/a` | `ordered:4` | `yes` | `closed` | `110` | artifact class ref/checksum required | `REGISTRY_ARTIFACT_INACTIVE` | `ARTIFACT_EVIDENCE_REF_FORBIDDEN` |
| `artifact_ref` | `030.ActivationControlledArtifactRef or EvidenceRef` | `yes` | `none` | `no` | `no` | immutable artifact ref only | `n/a` | `reject` | `n/a` | `ordered:5` | `yes` | `closed` | `110` | artifact ref/checksum required | `REGISTRY_ARTIFACT_INACTIVE` | `REGISTRY_ARTIFACT_OWNER_MISMATCH` |
| `artifact_checksum` | `040.ScalarType.sha256` | `yes` | `none` | `no` | `no` | lowercase SHA-256 over owner-declared artifact bytes | `n/a` | `reject` | `n/a` | `ordered:6` | `yes` | `closed` | `110` | checksum required | `REGISTRY_ARTIFACT_INACTIVE` | `REGISTRY_ARTIFACT_OWNER_MISMATCH` |
| `governance_owner_refs` | `array<030.ActivationControlledArtifactRef>` | `yes` | `none` | `no` | `no` | non-empty owner refs | `canonical_set` | `reject` | `artifact_ref` | `no` | `yes` | `closed` | `110` | owner refs required | `REGISTRY_ARTIFACT_OWNER_MISMATCH` | `REGISTRY_ARTIFACT_OWNER_MISMATCH` |
| `domain_refs` | `array<030.ActivationControlledArtifactRef>` | `no` | `[]` | `no` | `yes` | bounded governance metadata refs | `canonical_set` | `reject` | `artifact_ref` | `no` | `yes` | `closed` | `110` | refs required when exposed | `REGISTRY_ARTIFACT_INACTIVE` | `REGISTRY_AUTHORITY_FORBIDDEN` |
| `classification_refs` | `array<030.ActivationControlledArtifactRef>` | `no` | `[]` | `no` | `yes` | selected `RegistryClassificationPolicy` refs | `canonical_set` | `reject` | `artifact_ref` | `no` | `yes` | `closed` | `110` | refs required when labels attached | `REGISTRY_ARTIFACT_INACTIVE` | `REGISTRY_CLASSIFICATION_AUTHORITY_FORBIDDEN` |
| `glossary_refs` | `array<030.ActivationControlledArtifactRef>` | `no` | `[]` | `no` | `yes` | governance metadata only | `canonical_set` | `reject` | `artifact_ref` | `no` | `yes` | `closed` | `110` | refs required when exposed | `REGISTRY_ARTIFACT_INACTIVE` | `REGISTRY_AUTHORITY_FORBIDDEN` |
| `policy_refs` | `array<030.ActivationControlledArtifactRef>` | `no` | `[]` | `no` | `yes` | governance policy refs only | `canonical_set` | `reject` | `artifact_ref` | `no` | `yes` | `closed` | `110` | refs required when policy affects visibility | `REGISTRY_ARTIFACT_INACTIVE` | `REGISTRY_AUTHORITY_FORBIDDEN` |
| `custom_property_schema_refs` | `array<030.ActivationControlledArtifactRef>` | `no` | `[]` | `no` | `yes` | active `RegistryCustomPropertySchema` refs | `canonical_set` | `reject` | `artifact_ref` | `no` | `yes` | `closed` | `110` | refs required when custom properties attached | `REGISTRY_ARTIFACT_INACTIVE` | `REGISTRY_CUSTOM_PROPERTY_SCHEMA_INVALID` |
| `approval_refs` | `array<030.ActivationControlledArtifactRef>` | `yes` | `none` | `no` | `no` | non-empty governance approval metadata refs; not production approval | `canonical_set` | `reject` | `artifact_ref` | `no` | `yes` | `closed` | `110` | approval metadata refs required for governance activation | `REGISTRY_ARTIFACT_INACTIVE` | `REGISTRY_AUTHORITY_FORBIDDEN` |
| `governance_purpose` | `130.RegistryGovernancePurpose` | `yes` | `metadata_governance` | `no` | `no` | closed governance-purpose token | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | purpose included in manifest | `REGISTRY_ARTIFACT_INACTIVE` | `REGISTRY_AUTHORITY_FORBIDDEN` |
| `authority_grant_ref` | `030.ActivationControlledArtifactRef` | `no` | `null` | `yes` | `no` | non-null only when another owner exports an exact authority interface and validation refs pass | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | authority grant interface refs required when non-null | `REGISTRY_AUTHORITY_FORBIDDEN` | `REGISTRY_AUTHORITY_FORBIDDEN` |
| `activation_scope` | `030.ActivationScope` | `yes` | `none` | `no` | `no` | must cover registry activation request | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | selector context and checksum required | `REGISTRY_ARTIFACT_INACTIVE` | `REGISTRY_ARTIFACT_OWNER_MISMATCH` |
| `package_set_ref` | `030.ActivationControlledArtifactRef` | `conditional:package-supplied artifact` | `null` | `yes` | `no` | required when package-supplied and null otherwise | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | package-set ref/checksum required when package-supplied | `REGISTRY_ARTIFACT_INACTIVE` | `REGISTRY_VOLATILITY_BOUNDARY_VIOLATION` |
| `validation_refs` | `array<030.ActivationControlledArtifactRef>` | `yes` | `none` | `no` | `no` | non-empty registry governance validation refs | `canonical_set` | `reject` | `artifact_ref` | `no` | `yes` | `closed` | `110` | validation refs/checksums required | `REGISTRY_ARTIFACT_INACTIVE` | `REGISTRY_AUTHORITY_FORBIDDEN` |
| `lifecycle_status` | `030.LifecycleStatus` | `yes` | `none` | `no` | `no` | production activation requires `active` | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | lifecycle evidence required | `REGISTRY_ARTIFACT_INACTIVE` | `REGISTRY_ARTIFACT_INACTIVE` |
| `version_manifest_requirement` | `130.VersionManifestRequirement` | `yes` | `selected_registry_artifact_governance_refs` | `no` | `no` | all selected refs/checksums in `030.VersionManifest` | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | manifest refs required before activation visibility | `REGISTRY_ARTIFACT_INACTIVE` | `REGISTRY_VOLATILITY_BOUNDARY_VIOLATION` |

`approval_refs` are governance metadata only and must not become production approval. A non-null `authority_grant_ref` is valid only when another owner spec exports an exact authority interface and every validation ref for that interface passes.

### RegistryCustomPropertySchema

`RegistryCustomPropertySchema` is an activation-controlled schema row for typed, bounded governance metadata. It must not create core fields, source-extension fields, gold facts, graph properties, identity evidence, source authority, package activation authority, or authority grants.

Namespace pattern is dot-separated lower-snake-case segments, with 1 to 8 segments and a maximum length of 128 characters. `property_path` is a dot-separated lower-snake-case path, with 1 to 12 segments and a maximum length of 256 characters. Wildcards are forbidden.

| field_path | type | required | default | null_allowed | omit_allowed | bounds | array_semantics | duplicate_policy | canonical_sort_key | id_input | checksum_input | extension_policy | redaction_owner | version_manifest_requirement | missing_error | invalid_error |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `schema_id` | `040.ScalarType.string` | `yes` | `none` | `no` | `no` | stable schema ID scoped to `130` | `n/a` | `reject` | `n/a` | `ordered:1` | `yes` | `closed` | `110` | selected schema ref/checksum required | `REGISTRY_CUSTOM_PROPERTY_SCHEMA_INVALID` | `REGISTRY_CUSTOM_PROPERTY_SCHEMA_INVALID` |
| `schema_version` | `040.ScalarType.string` | `yes` | `none` | `no` | `no` | immutable owner version | `n/a` | `reject` | `n/a` | `ordered:2` | `yes` | `closed` | `110` | schema version included | `REGISTRY_CUSTOM_PROPERTY_SCHEMA_INVALID` | `REGISTRY_CUSTOM_PROPERTY_SCHEMA_INVALID` |
| `namespace` | `040.ScalarType.string` | `yes` | `none` | `no` | `no` | dot-separated lower-snake-case; max 8 segments; max 128 chars | `n/a` | `reject` | `n/a` | `ordered:3` | `yes` | `closed` | `110` | namespace checksum required | `REGISTRY_CUSTOM_PROPERTY_SCHEMA_INVALID` | `REGISTRY_CUSTOM_PROPERTY_SCHEMA_INVALID` |
| `property_path` | `040.ScalarType.string` | `yes` | `none` | `no` | `no` | dot-separated lower-snake-case; 1..12 segments; max 256 chars; wildcards forbidden | `n/a` | `reject` | `n/a` | `ordered:4` | `yes` | `closed` | `110` | path checksum required | `REGISTRY_CUSTOM_PROPERTY_SCHEMA_INVALID` | `REGISTRY_CUSTOM_PROPERTY_SCHEMA_INVALID` |
| `scalar_type` | `040.ScalarType or 130.RegistryScalarType` | `yes` | `none` | `no` | `no` | closed scalar type; unbounded objects forbidden | `n/a` | `reject` | `n/a` | `ordered:5` | `yes` | `closed` | `110` | type included | `REGISTRY_CUSTOM_PROPERTY_SCHEMA_INVALID` | `REGISTRY_CUSTOM_PROPERTY_SCHEMA_INVALID` |
| `enum_values` | `array<040.ScalarType.string>` | `conditional:scalar_type is enum` | `[]` | `no` | `no` | non-empty bounded canonical set for enum; max 256 values | `canonical_set` | `reject` | `value` | `ordered:6` | `yes` | `closed` | `110` | enum values checksum required for enum type | `REGISTRY_CUSTOM_PROPERTY_SCHEMA_INVALID` | `REGISTRY_CUSTOM_PROPERTY_SCHEMA_INVALID` |
| `cardinality` | `130.RegistryCustomPropertyCardinality` | `yes` | `one` | `no` | `no` | `one`, `optional_one`, `array`, or `map` | `n/a` | `reject` | `n/a` | `ordered:7` | `yes` | `closed` | `110` | cardinality included | `REGISTRY_CUSTOM_PROPERTY_SCHEMA_INVALID` | `REGISTRY_CUSTOM_PROPERTY_SCHEMA_INVALID` |
| `bounds` | `map` | `yes` | `none` | `no` | `no` | finite min/max length, count, key, numeric, object, or enum bounds; unbounded values forbidden | lexical key order | `reject` | `key` | `ordered:8` | `yes` | `closed` | `110` | bounds checksum required | `REGISTRY_CUSTOM_PROPERTY_SCHEMA_INVALID` | `REGISTRY_CUSTOM_PROPERTY_SCHEMA_INVALID` |
| `default_value` | `owner scalar or null` | `no` | `null` | `yes` | `no` | must validate against type, bounds, cardinality, and null policy | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | default checksum required when materialized | `REGISTRY_CUSTOM_PROPERTY_SCHEMA_INVALID` | `REGISTRY_CUSTOM_PROPERTY_SCHEMA_INVALID` |
| `null_allowed` | `boolean` | `yes` | `false` | `no` | `no` | false by default | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | null policy included | `REGISTRY_CUSTOM_PROPERTY_SCHEMA_INVALID` | `REGISTRY_CUSTOM_PROPERTY_SCHEMA_INVALID` |
| `omit_behavior` | `130.RegistryOmitBehavior` | `yes` | `required_for_cardinality_one_else_omit_means_absent_metadata` | `no` | `no` | closed behavior by cardinality | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | omit behavior included | `REGISTRY_CUSTOM_PROPERTY_SCHEMA_INVALID` | `REGISTRY_CUSTOM_PROPERTY_SCHEMA_INVALID` |
| `redaction_class` | `110.RedactionDataClassMatrix` | `yes` | `none` | `no` | `no` | must be one closed redaction class | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | redaction class required before exposure | `REGISTRY_CUSTOM_PROPERTY_SCHEMA_INVALID` | `REGISTRY_CUSTOM_PROPERTY_SCHEMA_INVALID` |
| `allowed_attachment_targets` | `array<130.RegistryAttachmentTarget>` | `yes` | `none` | `no` | `no` | non-empty closed target set | `canonical_set` | `reject` | `target` | `no` | `yes` | `closed` | `110` | attachment target refs required | `REGISTRY_CUSTOM_PROPERTY_SCHEMA_INVALID` | `REGISTRY_CUSTOM_PROPERTY_SCHEMA_INVALID` |
| `authority_effect` | `130.AnalysisEffect` | `yes` | `none` | `no` | `no` | only MVP value is `none` | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | authority no-op refs required for attempted authority | `REGISTRY_CUSTOM_PROPERTY_SCHEMA_INVALID` | `REGISTRY_AUTHORITY_FORBIDDEN` |
| `extension_policy` | `130.RegistryCustomPropertyExtensionPolicy` | `yes` | `closed` | `no` | `no` | unknown properties reject unless active schema row declares them | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | extension policy included | `REGISTRY_CUSTOM_PROPERTY_SCHEMA_INVALID` | `REGISTRY_CUSTOM_PROPERTY_SCHEMA_INVALID` |
| `validation_refs` | `array<030.ActivationControlledArtifactRef>` | `yes` | `none` | `no` | `no` | non-empty custom-property validation refs | `canonical_set` | `reject` | `artifact_ref` | `no` | `yes` | `closed` | `110` | validation refs/checksums required | `REGISTRY_CUSTOM_PROPERTY_SCHEMA_INVALID` | `REGISTRY_CUSTOM_PROPERTY_SCHEMA_INVALID` |
| `activation_scope` | `030.ActivationScope` | `yes` | `none` | `no` | `no` | must cover registry request scope | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | selector context and checksum required | `REGISTRY_CUSTOM_PROPERTY_SCHEMA_INVALID` | `REGISTRY_ARTIFACT_OWNER_MISMATCH` |
| `lifecycle_status` | `030.LifecycleStatus` | `yes` | `none` | `no` | `no` | production use requires `active` | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | lifecycle evidence required | `REGISTRY_ARTIFACT_INACTIVE` | `REGISTRY_ARTIFACT_INACTIVE` |
| `package_set_ref` | `030.ActivationControlledArtifactRef` | `conditional:package-supplied schema` | `null` | `yes` | `no` | required when package-supplied and null otherwise | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | package-set ref/checksum required when package-supplied | `REGISTRY_CUSTOM_PROPERTY_SCHEMA_INVALID` | `REGISTRY_VOLATILITY_BOUNDARY_VIOLATION` |
| `version_manifest_requirement` | `130.VersionManifestRequirement` | `yes` | `selected_custom_property_schema_refs` | `no` | `no` | selected schema refs/checksums, validation refs, lifecycle refs, package refs when applicable | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | manifest refs required | `REGISTRY_CUSTOM_PROPERTY_SCHEMA_INVALID` | `REGISTRY_VOLATILITY_BOUNDARY_VIOLATION` |

Unknown custom properties must reject unless declared by exactly one active schema row. Unbounded strings, arrays, maps, numerics, objects, and enums must fail with `REGISTRY_CUSTOM_PROPERTY_SCHEMA_INVALID`.

### RegistryClassificationPolicy

`RegistryClassificationPolicy` governs governance labels and glossary/classification metadata. It must not grant source authority, fact authority, identity authority, graph authority, package activation, production approval, completeness, watermark, remediation, or validation acceptance by itself.

| field_path | type | required | default | null_allowed | omit_allowed | bounds | array_semantics | duplicate_policy | canonical_sort_key | id_input | checksum_input | extension_policy | redaction_owner | version_manifest_requirement | missing_error | invalid_error |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `policy_id` | `040.ScalarType.string` | `yes` | `none` | `no` | `no` | stable policy ID scoped to `130` | `n/a` | `reject` | `n/a` | `ordered:1` | `yes` | `closed` | `110` | selected policy ref/checksum required | `REGISTRY_CLASSIFICATION_AUTHORITY_FORBIDDEN` | `REGISTRY_CLASSIFICATION_AUTHORITY_FORBIDDEN` |
| `policy_version` | `040.ScalarType.string` | `yes` | `none` | `no` | `no` | immutable owner version | `n/a` | `reject` | `n/a` | `ordered:2` | `yes` | `closed` | `110` | version included | `REGISTRY_CLASSIFICATION_AUTHORITY_FORBIDDEN` | `REGISTRY_CLASSIFICATION_AUTHORITY_FORBIDDEN` |
| `label_namespace` | `040.ScalarType.string` | `yes` | `none` | `no` | `no` | bounded classification namespace token | `n/a` | `reject` | `n/a` | `ordered:3` | `yes` | `closed` | `110` | namespace checksum required | `REGISTRY_CLASSIFICATION_AUTHORITY_FORBIDDEN` | `REGISTRY_CLASSIFICATION_AUTHORITY_FORBIDDEN` |
| `allowed_labels` | `array<040.ScalarType.string>` | `yes` | `none` | `no` | `no` | non-empty canonical set; label values bounded and redaction-aware | `canonical_set` | `reject` | `label` | `ordered:4` | `yes` | `closed` | `110` | selected labels required before attachment | `REGISTRY_CLASSIFICATION_AUTHORITY_FORBIDDEN` | `REGISTRY_CLASSIFICATION_AUTHORITY_FORBIDDEN` |
| `attachment_targets` | `array<130.RegistryAttachmentTarget>` | `yes` | `none` | `no` | `no` | non-empty closed target set | `canonical_set` | `reject` | `target` | `ordered:5` | `yes` | `closed` | `110` | target refs required | `REGISTRY_CLASSIFICATION_AUTHORITY_FORBIDDEN` | `REGISTRY_CLASSIFICATION_AUTHORITY_FORBIDDEN` |
| `label_conflict_behavior` | `130.RegistryLabelConflictBehavior` | `yes` | `reject` | `no` | `no` | `reject`, `most_restrictive_visibility_wins`, or `owner_ordered_precedence` | `n/a` | `reject` | `n/a` | `ordered:6` | `yes` | `closed` | `110` | conflict behavior included | `REGISTRY_CLASSIFICATION_AUTHORITY_FORBIDDEN` | `REGISTRY_CLASSIFICATION_AUTHORITY_FORBIDDEN` |
| `redaction_effect` | `130.RegistryRedactionEffect` | `yes` | `none` | `no` | `no` | may only narrow visibility through `110.RedactionPolicy` | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | redaction refs required when narrowing visibility | `REGISTRY_CLASSIFICATION_AUTHORITY_FORBIDDEN` | `REGISTRY_CLASSIFICATION_AUTHORITY_FORBIDDEN` |
| `export_effect` | `130.RegistryExportEffect` | `yes` | `metadata_only` | `no` | `no` | default and MVP permitted value is `metadata_only` unless owner narrows | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | export effect included | `REGISTRY_CLASSIFICATION_AUTHORITY_FORBIDDEN` | `REGISTRY_CLASSIFICATION_AUTHORITY_FORBIDDEN` |
| `authority_effect` | `130.AnalysisEffect` | `yes` | `none` | `no` | `no` | default and only MVP value is `none` | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | no-op refs required for attempted authority | `REGISTRY_CLASSIFICATION_AUTHORITY_FORBIDDEN` | `REGISTRY_CLASSIFICATION_AUTHORITY_FORBIDDEN` |
| `unknown_label_behavior` | `130.RegistryUnknownLabelBehavior` | `yes` | `reject` | `no` | `no` | `reject` or `store_as_non_authoritative_diagnostic_metadata` | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | behavior included | `REGISTRY_CLASSIFICATION_AUTHORITY_FORBIDDEN` | `REGISTRY_CLASSIFICATION_AUTHORITY_FORBIDDEN` |
| `validation_refs` | `array<030.ActivationControlledArtifactRef>` | `yes` | `none` | `no` | `no` | non-empty classification validation refs | `canonical_set` | `reject` | `artifact_ref` | `no` | `yes` | `closed` | `110` | validation refs/checksums required | `REGISTRY_CLASSIFICATION_AUTHORITY_FORBIDDEN` | `REGISTRY_CLASSIFICATION_AUTHORITY_FORBIDDEN` |
| `activation_scope` | `030.ActivationScope` | `yes` | `none` | `no` | `no` | must cover registry label request scope | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | selector context and checksum required | `REGISTRY_CLASSIFICATION_AUTHORITY_FORBIDDEN` | `REGISTRY_ARTIFACT_OWNER_MISMATCH` |
| `lifecycle_status` | `030.LifecycleStatus` | `yes` | `none` | `no` | `no` | production use requires `active` | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | lifecycle evidence required | `REGISTRY_ARTIFACT_INACTIVE` | `REGISTRY_ARTIFACT_INACTIVE` |
| `package_set_ref` | `030.ActivationControlledArtifactRef` | `conditional:package-supplied policy` | `null` | `yes` | `no` | required when package-supplied and null otherwise | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | package-set ref/checksum required when package-supplied | `REGISTRY_CLASSIFICATION_AUTHORITY_FORBIDDEN` | `REGISTRY_VOLATILITY_BOUNDARY_VIOLATION` |
| `version_manifest_requirement` | `130.VersionManifestRequirement` | `yes` | `selected_classification_policy_refs` | `no` | `no` | selected policy refs/checksums, validation refs, lifecycle refs, package refs when applicable | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | manifest refs required | `REGISTRY_CLASSIFICATION_AUTHORITY_FORBIDDEN` | `REGISTRY_VOLATILITY_BOUNDARY_VIOLATION` |

Unknown labels must reject before attachment unless a row explicitly permits storage as non-authoritative diagnostic metadata. `redaction_effect` may only narrow visibility through `110.RedactionPolicy`; it must not widen export or visibility.

### ActivateRegistryArtifact

```text
ActivateRegistryArtifact(governance_row, artifact_ref, package_set_ref, validation_matrix):
1. Validate `artifact_ref` through `030.ActivationControlledArtifactRef` using owner spec `130` or the imported owner spec named by the governance row.
2. Validate lifecycle status is `active`, artifact checksum matches, validation refs are non-empty, and `030.ScopeSelectorCovers(artifact_ref.activation_scope, request_scope, selected_context)` returns covered.
3. If the artifact is package-supplied, require `package_set_ref` and require it to match `governance_row.package_set_ref`.
4. Validate every referenced `RegistryCustomPropertySchema` and `RegistryClassificationPolicy` row.
5. If `authority_grant_ref` is null, activate governance metadata only.
6. If `authority_grant_ref` is non-null, resolve the exact exporting owner spec interface and validate owner refs before activation.
7. If the authority interface is missing, inactive, mismatched, or unvalidated, fail with `REGISTRY_AUTHORITY_FORBIDDEN` or `REGISTRY_VOLATILITY_BOUNDARY_VIOLATION` before any effect.
8. Emit activation metadata only; do not mutate raw, silver, identity, gold, graph-delta, graph-serving, completeness, watermark, package, or source-authority state.
```

## Registry and analysis artifact volatility boundary

Analysis, enrichment, lineage, and registry artifacts may be active only through owner-governed activation refs. Registry metadata cannot create authority unless an owner stable core contract grants the specific behavior.

| Artifact or record | Volatility class | Required handling |
| --- | --- | --- |
| `AnalysisRuleBundle` | `activation_controlled_artifact` | Must be active and compatible before analysis execution. |
| `AnalysisRule` | `activation_controlled_artifact` | Row inside a bundle; read-only by default. |
| `AnalysisFinding`, `AnalysisMetric`, `RiskAcceptanceRecord` | non-authoritative `runtime_state_record` | Workflow records; no mutation authority. |
| `DerivationRuleBundle` | `activation_controlled_artifact` | Gold output only through `080`. |
| `DerivedGraphEdgeRule` | `activation_controlled_artifact` | Graph deltas only through `090`. |
| `RuleGraphCompatibilityMatrix` | `activation_controlled_artifact` | Validation artifact. |
| `ThreatIntelEnrichmentProfile` | `activation_controlled_artifact` | Enrichment only unless another owner grants behavior. |
| `ThreatIntelArtifactRef` | activation-controlled artifact or runtime evidence ref depending on role | Must be immutable and checksummed. |
| `RunDatasetIOContract` | `runtime_state_record` | Runtime lineage state; not evidence authority by itself. |
| `LineageFacetMappingPolicy` | `activation_controlled_artifact` | Must be active before lineage facet effects. |
| `ArtifactClassPolicy` | stable governance contract plus activation-controlled owner-specific rows | Prevents artifact substitution. |
| `RegistryArtifactGovernance` | `activation_controlled_artifact` | Governance metadata with no fact authority by itself. |
| `RegistryCustomPropertySchema` | `activation_controlled_artifact` | Defines typed custom-property metadata only. |
| `RegistryClassificationPolicy` | `activation_controlled_artifact` | Classification metadata only. |

### AnalysisRegistryArtifactLifecycleGuardRows

Analysis, enrichment, lineage, and registry activation-controlled artifacts use `030.ActivationControlledArtifactLifecycleMachine.v1`. `130` owns only the non-authority and owner-specific guard rows below.

| Artifact class | Required guard before `active` | Quarantine trigger |
| --- | --- | --- |
| `AnalysisRuleBundle` | Rule graph compatibility passes, query target is active, rule is read-only, and validation refs pass. | Attempts mutation or graph profile incompatibility. |
| `DerivationRuleBundle` | Routes gold output only through `080`, with authority and completeness refs present. | Emits gold outside `080`. |
| `DerivedGraphEdgeRule` | Routes graph output only through `090`, with supporting facts, rule refs, and profile refs persisted. | Emits graph deltas without supporting facts or active projection path. |
| `ThreatIntelEnrichmentProfile` | Enrichment-only outputs and distribution mapping validate. | Attempts identity, source authority, completeness, or fact authority. |
| `LineageFacetMappingPolicy` | Immutable schema URL, schema bytes, checksum, and collision behavior validate. | Schema checksum mismatch or mutable schema URL. |
| `RegistryArtifactGovernance` | Owner, domain, lifecycle, checksum, approval, validation refs, activation scope, and package-set ref when package-supplied are present. | Registry metadata attempts production authority without owner spec. |
| `RegistryCustomPropertySchema` | Namespace, property path, scalar type, cardinality, explicit bounds, redaction class, allowed targets, authority effect, and validation refs pass. | Missing bounds, invalid redaction, or authority effect without owner interface. |
| `RegistryClassificationPolicy` | Label namespace, allowed labels, attachment targets, redaction/export effects, authority effect, and validation refs pass. | Classification attempts authority without owner interface. |
| `ThreatIntelDistributionMappingPolicy` | Distribution mapping is total for active profile values and redaction/export behavior passes. | Unmapped distribution or export of restricted values. |

## Analysis, Enrichment, and Registry Contract Details

### RiskScoringBoundary

Default `numeric_scoring_authority = disabled`. Numeric risk or exposure scores may become authoritative only after a future accepted scoring policy defines formula, bounds, inputs, defaults, calibration, authority, and validation rows. Until then, risk and exposure outputs are analysis metrics only.

### AnalysisMutationProhibitionMatrix

| Target state | Mutation by analysis/enrichment/registry | Default result |
| --- | --- | --- |
| raw | forbidden | `ANALYSIS_MUTATION_FORBIDDEN` |
| silver | forbidden | `ANALYSIS_MUTATION_FORBIDDEN` |
| identity | forbidden | `ANALYSIS_MUTATION_FORBIDDEN` |
| gold | forbidden unless routed through `080` derivation interface | error/no-op |
| graph-delta | forbidden unless routed through `090` projection interface | error/no-op |
| graph-serving | forbidden | `ANALYSIS_MUTATION_FORBIDDEN` |
| completeness | forbidden | `ANALYSIS_MUTATION_FORBIDDEN` |
| watermarks | forbidden | `ANALYSIS_MUTATION_FORBIDDEN` |
| package state | forbidden | `ANALYSIS_MUTATION_FORBIDDEN` |
| source authority | forbidden | `ANALYSIS_MUTATION_FORBIDDEN` |

### LineageFacetMappingPolicy

`LineageFacetMappingPolicy` maps external lineage facets to Cadastre lineage fields, diagnostic metadata, non-authoritative raw-facet metadata, or rejection. External lineage facets must remain non-authoritative by default and must not satisfy source evidence, source completeness, source authority, absence, cleanup, retraction, graph expiry, watermark, fact authority, identity authority, graph authority, package activation, production approval, or validation acceptance.

#### LineageFacetMappingRow schema

`LineageFacetMappingRow` is an activation-controlled row. The table below is its complete `030.ActivationControlledRowField` interface.

| field_path | type | required | default | null_allowed | omit_allowed | bounds | array_semantics | duplicate_policy | canonical_sort_key | id_input | checksum_input | extension_policy | redaction_owner | version_manifest_requirement | missing_error | invalid_error |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `row_id` | `040.ScalarType.string` | `yes` | `none` | `no` | `no` | stable row ID scoped to row set | `n/a` | `reject` | `n/a` | `ordered:1` | `yes` | `closed` | `110` | selected row ref/checksum required | `LINEAGE_FACET_POLICY_MISSING` | `LINEAGE_FACET_CHECKSUM_MISMATCH` |
| `row_version` | `040.ScalarType.string` | `yes` | `none` | `no` | `no` | immutable owner version | `n/a` | `reject` | `n/a` | `ordered:2` | `yes` | `closed` | `110` | row version included | `LINEAGE_FACET_POLICY_MISSING` | `LINEAGE_FACET_CHECKSUM_MISMATCH` |
| `lineage_system` | `130.LineageSystem` | `yes` | `none` | `no` | `no` | MVP enum: `openlineage`, `cadastre_internal_lineage` | `n/a` | `reject` | `n/a` | `ordered:3` | `yes` | `closed` | `110` | selected row required by lineage system | `LINEAGE_FACET_POLICY_MISSING` | `LINEAGE_FACET_POLICY_MISSING` |
| `facet_namespace` | `040.ScalarType.string` | `yes` | `none` | `no` | `no` | immutable namespace token or URL namespace | `n/a` | `reject` | `n/a` | `ordered:4` | `yes` | `closed` | `110` | namespace included | `LINEAGE_FACET_POLICY_MISSING` | `LINEAGE_FACET_NAMESPACE_COLLISION` |
| `facet_name` | `040.ScalarType.string` | `yes` | `none` | `no` | `no` | non-empty bounded facet name | `n/a` | `reject` | `n/a` | `ordered:5` | `yes` | `closed` | `110` | facet name included | `LINEAGE_FACET_POLICY_MISSING` | `LINEAGE_FACET_POLICY_MISSING` |
| `schema_url` | `040.ScalarType.string` | `yes` | `none` | `no` | `no` | immutable URL or checksum-addressed URI; mutable URLs forbidden | `n/a` | `reject` | `n/a` | `ordered:6` | `yes` | `closed` | `110` | schema URL included | `LINEAGE_FACET_POLICY_MISSING` | `LINEAGE_FACET_SCHEMA_MUTABLE` |
| `schema_bytes_checksum` | `040.ScalarType.sha256` | `yes` | `none` | `no` | `no` | SHA-256 over exact schema bytes | `n/a` | `reject` | `n/a` | `ordered:7` | `yes` | `closed` | `110` | schema checksum required | `LINEAGE_FACET_POLICY_MISSING` | `LINEAGE_FACET_CHECKSUM_MISMATCH` |
| `facet_bytes_checksum_policy` | `130.LineageFacetBytesChecksumPolicy` | `yes` | `required_for_mapped_or_stored_facets` | `no` | `no` | closed checksum policy; missing bytes fail when required | `n/a` | `reject` | `n/a` | `ordered:8` | `yes` | `closed` | `110` | facet checksum policy included | `LINEAGE_FACET_POLICY_MISSING` | `LINEAGE_FACET_CHECKSUM_MISMATCH` |
| `collision_behavior` | `130.LineageFacetCollisionBehavior` | `yes` | `reject` | `no` | `no` | default `reject`; no silent overwrite | `n/a` | `reject` | `n/a` | `ordered:9` | `yes` | `closed` | `110` | collision decision and row refs required | `LINEAGE_FACET_POLICY_MISSING` | `LINEAGE_FACET_NAMESPACE_COLLISION` |
| `raw_facet_storage` | `130.RawFacetStoragePolicy` | `yes` | `store_as_non_authoritative_metadata` | `no` | `no` | raw facet bytes never become evidence authority | `n/a` | `reject` | `n/a` | `ordered:10` | `yes` | `closed` | `110` | raw-facet storage policy and redaction refs required | `LINEAGE_FACET_POLICY_MISSING` | `REGISTRY_VOLATILITY_BOUNDARY_VIOLATION` |
| `mapped_field_set` | `130.LineageMappedFieldSet` | `yes` | `none` | `no` | `no` | closed enum: `none`, `run_dataset_io_metadata`, `dataset_version_metadata`, `diagnostic_metadata`, `non_authoritative_raw_facet_metadata`, `reject` | `n/a` | `reject` | `n/a` | `ordered:11` | `yes` | `closed` | `110` | mapped field set required | `LINEAGE_FACET_POLICY_MISSING` | `REGISTRY_VOLATILITY_BOUNDARY_VIOLATION` |
| `authority_class` | `010.AuthorityClass` | `yes` | `non_authoritative_analysis` | `no` | `no` | default and only MVP value is `non_authoritative_analysis` | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | authority class included | `LINEAGE_FACET_POLICY_MISSING` | `REGISTRY_VOLATILITY_BOUNDARY_VIOLATION` |
| `allowed_effects` | `array<130.AnalysisEffect>` | `yes` | `[]` | `no` | `no` | empty by default; any non-empty value requires explicit owner handoff | `canonical_set` | `reject` | `effect` | `no` | `yes` | `closed` | `110` | allowed-effect refs/no-op proofs required | `LINEAGE_FACET_POLICY_MISSING` | `REGISTRY_VOLATILITY_BOUNDARY_VIOLATION` |
| `redaction_policy_ref` | `030.ActivationControlledArtifactRef` | `yes` | `none` | `no` | `no` | active redaction policy ref | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | redaction ref/checksum required | `LINEAGE_FACET_POLICY_MISSING` | `REGISTRY_VOLATILITY_BOUNDARY_VIOLATION` |
| `validation_refs` | `array<030.ActivationControlledArtifactRef>` | `yes` | `none` | `no` | `no` | non-empty lineage facet validation refs | `canonical_set` | `reject` | `artifact_ref` | `no` | `yes` | `closed` | `110` | validation refs/checksums required | `LINEAGE_FACET_POLICY_MISSING` | `REGISTRY_VOLATILITY_BOUNDARY_VIOLATION` |
| `activation_scope` | `030.ActivationScope` | `yes` | `none` | `no` | `no` | must cover lineage request scope | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | selector context and checksum required | `LINEAGE_FACET_POLICY_MISSING` | `REGISTRY_ARTIFACT_OWNER_MISMATCH` |
| `lifecycle_status` | `030.LifecycleStatus` | `yes` | `none` | `no` | `no` | production use requires `active` | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | lifecycle evidence required | `REGISTRY_ARTIFACT_INACTIVE` | `REGISTRY_ARTIFACT_INACTIVE` |
| `package_set_ref` | `030.ActivationControlledArtifactRef` | `conditional:package-supplied row` | `null` | `yes` | `no` | required when package-supplied and null otherwise | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | package-set ref/checksum required when package-supplied | `LINEAGE_FACET_POLICY_MISSING` | `REGISTRY_VOLATILITY_BOUNDARY_VIOLATION` |
| `version_manifest_requirement` | `130.VersionManifestRequirement` | `yes` | `selected_lineage_facet_refs` | `no` | `no` | selected row refs/checksums, schema bytes checksum, facet checksum policy, collision decision, raw storage policy, redaction refs, validation refs, package refs when applicable | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | manifest refs required | `LINEAGE_FACET_POLICY_MISSING` | `REGISTRY_VOLATILITY_BOUNDARY_VIOLATION` |

Freshness facets must remain diagnostic only and must not satisfy completeness, source authority, absence, cleanup, retraction, graph expiry, or watermark effects.

### ArtifactClassPolicy substitution matrix

`ArtifactClassPolicy` is an activation-controlled row family that constrains artifact substitution by class. It does not create new `EvidenceRef.artifact_id.kind` values. Evidence-ref class/kind closure remains owned by `040.EvidenceArtifactClassRegistry`.

#### ArtifactClassPolicy row schema

| field_path | type | required | default | null_allowed | omit_allowed | bounds | array_semantics | duplicate_policy | canonical_sort_key | id_input | checksum_input | extension_policy | redaction_owner | version_manifest_requirement | missing_error | invalid_error |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `policy_id` | `040.ScalarType.string` | `yes` | `none` | `no` | `no` | stable policy ID scoped to `130` | `n/a` | `reject` | `n/a` | `ordered:1` | `yes` | `closed` | `110` | selected policy ref/checksum required | `ARTIFACT_CLASS_POLICY_MISSING` | `ARTIFACT_CLASS_POLICY_AMBIGUOUS` |
| `policy_version` | `040.ScalarType.string` | `yes` | `none` | `no` | `no` | immutable owner version | `n/a` | `reject` | `n/a` | `ordered:2` | `yes` | `closed` | `110` | version included | `ARTIFACT_CLASS_POLICY_MISSING` | `ARTIFACT_CLASS_POLICY_AMBIGUOUS` |
| `artifact_class` | `040.EvidenceArtifactClassRegistry.artifact_class` | `yes` | `none` | `no` | `no` | source artifact class governed by this row | `n/a` | `reject` | `n/a` | `ordered:3` | `yes` | `closed` | `110` | source class row ref/checksum required | `ARTIFACT_CLASS_POLICY_MISSING` | `ARTIFACT_EVIDENCE_REF_FORBIDDEN` |
| `owner_spec` | `owner_spec_token` | `yes` | `none` | `no` | `no` | owner of the artifact class and checksum basis | `n/a` | `reject` | `n/a` | `ordered:4` | `yes` | `closed` | `110` | owner ref required | `ARTIFACT_CLASS_POLICY_MISSING` | `ARTIFACT_AUTHORITY_CLASS_MISMATCH` |
| `volatility_class` | `000.VolatilityClass` | `yes` | `none` | `no` | `no` | one closed volatility class | `n/a` | `reject` | `n/a` | `ordered:5` | `yes` | `closed` | `110` | volatility classification ref required | `ARTIFACT_CLASS_POLICY_MISSING` | `REGISTRY_VOLATILITY_BOUNDARY_VIOLATION` |
| `authority_class` | `010.AuthorityClass` | `yes` | `non_authoritative_analysis` | `no` | `no` | candidate substitute must be same-authority-or-narrower | `n/a` | `reject` | `n/a` | `ordered:6` | `yes` | `closed` | `110` | authority class included | `ARTIFACT_CLASS_POLICY_MISSING` | `ARTIFACT_AUTHORITY_CLASS_MISMATCH` |
| `permitted_substitute_classes` | `array<040.EvidenceArtifactClassRegistry.artifact_class>` | `yes` | `[]` | `no` | `no` | default substitution is forbidden | `canonical_set` | `reject` | `artifact_class` | `ordered:7` | `yes` | `closed` | `110` | permitted class refs required when any substitution allowed | `ARTIFACT_CLASS_POLICY_MISSING` | `ARTIFACT_SUBSTITUTION_FORBIDDEN` |
| `forbidden_substitute_classes` | `array<040.EvidenceArtifactClassRegistry.artifact_class>` | `yes` | `all non-identical classes unless narrowed by active row` | `no` | `no` | canonical set; may narrow only through explicit active row | `canonical_set` | `reject` | `artifact_class` | `ordered:8` | `yes` | `closed` | `110` | forbidden class refs required | `ARTIFACT_CLASS_POLICY_MISSING` | `ARTIFACT_SUBSTITUTION_FORBIDDEN` |
| `substitution_effect_scope` | `130.ArtifactSubstitutionEffectScope` | `yes` | `none` | `no` | `no` | `none`, `visibility_only`, `validation_only`, `diagnostic_only` | `n/a` | `reject` | `n/a` | `ordered:9` | `yes` | `closed` | `110` | selected effect scope required | `ARTIFACT_CLASS_POLICY_MISSING` | `ARTIFACT_SUBSTITUTION_FORBIDDEN` |
| `evidence_ref_policy` | `130.ArtifactEvidenceRefPolicy` | `yes` | `no_new_artifact_id_kind` | `no` | `no` | must validate final class/kind pair through `040` | `n/a` | `reject` | `n/a` | `ordered:10` | `yes` | `closed` | `110` | `040` evidence registry row refs required when evidence ref is emitted | `ARTIFACT_EVIDENCE_REF_FORBIDDEN` | `ARTIFACT_EVIDENCE_REF_FORBIDDEN` |
| `redaction_policy_ref` | `030.ActivationControlledArtifactRef` | `yes` | `none` | `no` | `no` | active redaction policy ref | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | redaction ref/checksum required | `ARTIFACT_CLASS_POLICY_MISSING` | `REGISTRY_VOLATILITY_BOUNDARY_VIOLATION` |
| `validation_refs` | `array<030.ActivationControlledArtifactRef>` | `yes` | `none` | `no` | `no` | non-empty artifact-class policy validation refs | `canonical_set` | `reject` | `artifact_ref` | `no` | `yes` | `closed` | `110` | validation refs/checksums required | `ARTIFACT_CLASS_POLICY_MISSING` | `ARTIFACT_SUBSTITUTION_FORBIDDEN` |
| `activation_scope` | `030.ActivationScope` | `yes` | `none` | `no` | `no` | must cover substitution request scope | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | selector context and checksum required | `ARTIFACT_CLASS_POLICY_MISSING` | `REGISTRY_ARTIFACT_OWNER_MISMATCH` |
| `lifecycle_status` | `030.LifecycleStatus` | `yes` | `none` | `no` | `no` | production selection requires `active` | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | lifecycle evidence required | `REGISTRY_ARTIFACT_INACTIVE` | `REGISTRY_ARTIFACT_INACTIVE` |
| `package_set_ref` | `030.ActivationControlledArtifactRef` | `conditional:package-supplied policy` | `null` | `yes` | `no` | required when package-supplied and null otherwise | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | package-set ref/checksum required when package-supplied | `ARTIFACT_CLASS_POLICY_MISSING` | `REGISTRY_VOLATILITY_BOUNDARY_VIOLATION` |
| `version_manifest_requirement` | `130.VersionManifestRequirement` | `yes` | `selected_artifact_class_policy_refs` | `no` | `no` | selected policy refs/checksums, evidence registry checksum, candidate artifact checksum, request checksum, validation refs, mutation-prohibition refs, package refs when applicable | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | manifest refs required | `ARTIFACT_CLASS_POLICY_MISSING` | `REGISTRY_VOLATILITY_BOUNDARY_VIOLATION` |

Default substitution is forbidden. `permitted_substitute_classes` defaults to `[]`; `forbidden_substitute_classes` defaults to all non-identical classes unless narrowed by an explicit active row. A candidate substitute must be explicit, same-authority-or-narrower, checksum-valid, validation-backed, lifecycle-active, redaction-valid, and manifest-included. A row must not add new `EvidenceRef.artifact_id.kind` values.

```text
EvaluateArtifactClassSubstitution(request, policy_row_set, evidence_registry):
1. Resolve exactly one active `ArtifactClassPolicy` row for requested source class, target class, effect scope, and activation scope.
2. If no row resolves, emit `ARTIFACT_CLASS_POLICY_MISSING`.
3. If more than one maximal row resolves, emit `ARTIFACT_CLASS_POLICY_AMBIGUOUS`.
4. Validate candidate artifact checksum, lifecycle status, validation refs, redaction refs, and package-set ref when package-supplied.
5. Validate `EvidenceRef.artifact_class` and `artifact_id.kind` against `040.EvidenceArtifactClassRegistry` when the request creates or validates an evidence ref.
6. Reject authority widening with `ARTIFACT_AUTHORITY_CLASS_MISMATCH`.
7. Reject forbidden substitutions with `ARTIFACT_SUBSTITUTION_FORBIDDEN`.
8. Emit only the permitted effect scope and include selected row refs in `030.VersionManifest`.
```

| Candidate substitute | Default behavior |
| --- | --- |
| lineage facet for source evidence | `ARTIFACT_SUBSTITUTION_FORBIDDEN` |
| freshness artifact for source completeness | `ARTIFACT_SUBSTITUTION_FORBIDDEN` |
| validation report for production evidence | `ARTIFACT_SUBSTITUTION_FORBIDDEN` |
| registry label for source authority | `ARTIFACT_SUBSTITUTION_FORBIDDEN` |
| graph rebuild artifact for source truth | `ARTIFACT_SUBSTITUTION_FORBIDDEN` |
| candidate evidence ref with unsupported class/kind pair | `ARTIFACT_EVIDENCE_REF_FORBIDDEN` or `EVIDENCE_ARTIFACT_CLASS_KIND_MISMATCH` according to `040` precedence |

### RegistryActivationPolicy

Registry governance artifacts require `030.ActivationControlledArtifactRef`. Registry governance artifacts may become active only when owner, artifact class, artifact checksum, lifecycle status, governance purpose, classification refs, glossary refs, custom-property schema refs, approval refs, validation refs, activation scope, and package-set ref when package-supplied are present. Registry metadata remains governance metadata unless another active owner spec grants authority through an exact named interface.

| Activation condition | Required behavior |
| --- | --- |
| Required governance field omitted | Activation fails with `REGISTRY_ARTIFACT_INACTIVE` or `REGISTRY_CUSTOM_PROPERTY_SCHEMA_INVALID` before effect. |
| Artifact owner does not match owner behavior | Activation fails with `REGISTRY_ARTIFACT_OWNER_MISMATCH`. |
| `authority_grant_ref = null` | Activate governance metadata only. |
| `authority_grant_ref` names an active owner interface and validation refs pass | Activate only the imported owner behavior permitted by that interface. |
| `authority_grant_ref` is non-null but owner interface is missing, inactive, mismatched, or unvalidated | Activation fails with `REGISTRY_AUTHORITY_FORBIDDEN` or `REGISTRY_VOLATILITY_BOUNDARY_VIOLATION`. |
| Package-supplied artifact omits package-set ref | Activation fails before output; package-set membership is execution eligibility only. |

### Analysis output authority table

`AnalysisOutputAuthorityMatrix` is total for the analysis, enrichment, lineage, registry, and derived-edge output classes in this spec.

| Output class | Required fields | Default authority | Permitted downstream use | Prohibited mutations | Replay checksum inputs | Fixture refs |
| --- | --- | --- | --- | --- | --- | --- |
| `AnalysisFinding` | `finding_id`, rule bundle ref, rule row refs, query target refs, graph compatibility refs when graph-backed, authorization refs, redaction refs, evidence metadata refs, canonical finding payload, output checksum, `VersionManifest` ref | `non_authoritative_analysis` | UI, API read-only analysis, audit, workflow context | raw, silver, identity, gold, graph-delta, graph-serving, completeness, watermark, package state, source authority | rule bundle refs, rule row refs, query target refs, graph derived-view refs, authorization/redaction refs, canonical output checksum, validation refs | `fixture-130-analysis-finding-non-authority`, `fixture-130-analysis-finding-mutation-forbidden`, `fixture-130-analysis-replay-exact` |
| `AnalysisMetric` | `metric_id`, metric name, metric value, numeric precision row, rule refs, query target refs, authorization refs, redaction refs, output checksum, `VersionManifest` ref | `non_authoritative_analysis` | Dashboards, reports, read-only API output | authoritative risk score, fact mutation, remediation, graph mutation, package state | metric row refs, numeric scoring boundary refs, query target refs, derived-view refs, authorization/redaction refs, output checksum | `fixture-130-analysis-metric-non-authority`, `fixture-130-analysis-metric-risk-score-forbidden` |
| `RiskAcceptanceRecord` | `risk_acceptance_id`, related finding refs, actor/approver authorization refs, workflow state, acceptance reason, `expires_at` or explicit null, redaction refs, output checksum, `VersionManifest` ref | `non_authoritative_analysis` workflow metadata | Audit and workflow context | remediation, retraction, risk reduction proof, graph edge removal, source completeness change | workflow record checksum, authorization refs, related finding refs, redaction refs, expiry/null marker, `VersionManifest` ref | `fixture-130-risk-acceptance-no-remediation` |
| `ThreatIntelEnrichmentRecord` | `record_id`, profile ref, artifact refs, indicator context, sighting context, taxonomy/galaxy/object-template context, distribution mapping ref, visibility class, redaction policy ref, output checksum, `VersionManifest` ref | `non_authoritative_analysis` | Enrichment context, filtering, analysis display, restricted export when mapping permits | identity evidence, source authority, source completeness, gold fact, graph edge, absence, cleanup, retraction, coverage, watermark | profile ref, artifact refs, distribution mapping ref, raw-value redaction refs, output checksum, validation refs | `fixture-130-threat-intel-known-indicator`, `fixture-130-threat-intel-identity-forbidden`, `fixture-130-threat-intel-distribution-restricted` |
| `RegistryArtifactGovernance` | `governance_id`, owner spec, artifact class, artifact checksum, lifecycle status, governance purpose, classification refs, glossary refs, custom-property schema refs, approval refs, activation scope, package-set ref when package-supplied, optional authority grant ref, validation refs | `non_authoritative_analysis` governance metadata | Ownership, approval metadata, labels, custom-property context, registry display | fact authority, source authority, graph edge semantics, evidence refs, source completeness, production approval by itself | governance row ref, custom-property schema refs, classification refs, lifecycle transition evidence, package-set ref when package-supplied, artifact checksum, activation scope, validation refs | `fixture-130-registry-activation-positive`, `fixture-130-registry-label-no-fact-authority`, `fixture-130-registry-package-set-ref-required` |

Numeric scoring authority remains disabled by default. Risk and exposure metrics remain `AnalysisMetric` outputs only unless a future accepted scoring policy defines formula, bounds, inputs, defaults, calibration, authority, and validation rows.

### External lineage facet table

| External facet | Cadastre placement | Authority class | Required checksum | Rejected conditions |
| --- | --- | --- | --- | --- |
| run/job/dataset facet | `RunDatasetIOContract` metadata | non-authoritative lineage | schema bytes and facet bytes | mutable schema URL, missing bytes, checksum mismatch |
| custom facet | raw facet storage or mapped diagnostic field | non-authoritative metadata | schema and raw facet | namespace collision, missing policy row |
| schema facet | schema diagnostic only unless owner maps | non-authoritative metadata | schema bytes | mismatch, stale, missing owner |
| freshness facet | freshness diagnostic only | non-authoritative | facet bytes | use as completeness proof |

### AnalysisApiHandoff

Analysis output exposed through API surfaces must import `110.CommonApiResponseEnvelope`; `130` must not redefine the envelope. Analysis remains non-authoritative and read-only unless another owner spec grants a named mutation interface.

| Analysis API condition | Required output or error | Required refs |
| --- | --- | --- |
| `analysis_read_only` success payload | Read-only result, finding preview, metric, or execution summary wrapped by `110.CommonApiResponseEnvelope`. | rule bundle, graph compatibility, authorization, redaction, version manifest, and derived-view refs when graph-backed. |
| empty analysis result | Success with empty result and envelope; no finding authority. | rule bundle, query target, result checksum. |
| stale derived view | Reject or label only as permitted by `090.GraphQueryResponseHandoffTo110` and `RuleGraphCompatibilityMatrix`. | derived-view state and compatibility refs. |
| graph compatibility mismatch | Owner error; no analysis output promotion. | compatibility matrix row and failure refs. |
| authorization or redaction mismatch | Owner or `110` authorization/redaction error; no result substitution. | authorization decision and redaction context refs. |
| mutation attempt | `ANALYSIS_MUTATION_FORBIDDEN`. | mutation-prohibition fixture ref. |

`AnalysisRuleBundle` execution summary output may reference `110.CommonApiResponseEnvelope` only by import.

### Analysis and lineage error codes

| Error code | Emitted when |
| --- | --- |
| `REGISTRY_ARTIFACT_INACTIVE` | Required analysis, lineage, enrichment, registry, classification, custom-property, or derived-edge artifact is not active. |
| `REGISTRY_ARTIFACT_OWNER_MISMATCH` | Registry artifact owner does not match the stable core behavior owner. |
| `REGISTRY_VOLATILITY_BOUNDARY_VIOLATION` | Registry metadata, lineage facet, analysis rule, threat-intel artifact, or derived-edge artifact attempts to become fact, identity, graph, source, completeness, package, or watermark authority without owner contract. |
| `ANALYSIS_MUTATION_FORBIDDEN` | Analysis, enrichment, lineage, or registry output attempts forbidden mutation. |
| `LINEAGE_FACET_POLICY_MISSING` | A lineage system, facet namespace, facet name, or activation scope has no active policy row. |
| `LINEAGE_FACET_SCHEMA_MUTABLE` | Facet schema URL is mutable or not immutable under policy. |
| `LINEAGE_FACET_CHECKSUM_MISMATCH` | Schema bytes or facet bytes do not match recorded checksum. |
| `LINEAGE_FACET_NAMESPACE_COLLISION` | Facet namespace collides and no active collision rule permits the collision. |
| `REGISTRY_AUTHORITY_FORBIDDEN` | Registry metadata attempts fact, graph, source authority, evidence, production approval, or another authority effect without exact owner interface. |
| `REGISTRY_CUSTOM_PROPERTY_SCHEMA_INVALID` | Custom-property schema omits required type, cardinality, bounds, default, redaction, attachment, authority, validation, activation, or lifecycle fields. |
| `REGISTRY_CLASSIFICATION_AUTHORITY_FORBIDDEN` | Classification or glossary label attempts authority beyond governance metadata. |
| `THREAT_INTEL_IDENTITY_FORBIDDEN` | Threat-intel indicator or enrichment attempts identity authority. |
| `THREAT_INTEL_PROFILE_MISSING` | Threat-intel enrichment execution has no active profile covering the input format and scope. |
| `THREAT_INTEL_DISTRIBUTION_UNMAPPED` | Distribution, sharing, or TLP-like value has no active mapping and the profile requires rejection. |
| `THREAT_INTEL_ARTIFACT_CHECKSUM_MISMATCH` | Threat-intel artifact bytes do not match the recorded checksum. |
| `DERIVED_GRAPH_EDGE_SUPPORTING_FACTS_REQUIRED` | Derived graph edge output lacks required supporting fact refs or support policy refs. |
| `DERIVED_GRAPH_EDGE_PROJECTION_FORBIDDEN` | Derived graph edge output attempts graph output without active `090` projection, semantics, eligibility, and apply refs. |
| `DERIVED_GRAPH_EDGE_REPLAY_MISMATCH` | Derived graph edge replay checksum, deterministic ID input set, supporting refs, or projection refs mismatch. |
| `ARTIFACT_CLASS_POLICY_MISSING` | Artifact substitution or class eligibility is evaluated and no active `ArtifactClassPolicy` row resolves for the request. |
| `ARTIFACT_CLASS_POLICY_AMBIGUOUS` | More than one maximal `ArtifactClassPolicy` row resolves for the same substitution request. |
| `ARTIFACT_SUBSTITUTION_FORBIDDEN` | Requested artifact substitution is not explicitly permitted for the requested effect scope. |
| `ARTIFACT_AUTHORITY_CLASS_MISMATCH` | Candidate substitute would widen authority class or owner authority. |
| `ARTIFACT_EVIDENCE_REF_FORBIDDEN` | Requested evidence ref class/kind pair is forbidden by `130.ArtifactClassPolicy` before evidence-ref creation. |

### AnalysisErrorRegistryFragment

This owner fragment feeds `110.GenerateErrorCodeRegistry`. `110` owns the generated caller-visible registry. This table must not render API output by itself. Every row is closed for severity, retry class, caller-visible fields, audit-visible fields, redaction, owner context, and fixture ref. Generated rows must use `110.StandardErrorCallerFields` and `110.StandardErrorAuditFields`; owner-specific detail remains inside `AnalysisErrorContext`.

| error_code | owner_spec | severity | retry_class | caller_visible_fields | audit_visible_fields | redaction_rule | owner_context_schema_ref | fixture_ref |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `REGISTRY_ARTIFACT_INACTIVE` | `130` | `blocked` | `retry_after_owner_repair` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `130.AnalysisErrorContext` | `error-registry-130-registry-artifact-inactive` |
| `REGISTRY_ARTIFACT_OWNER_MISMATCH` | `130` | `error` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `130.AnalysisErrorContext` | `error-registry-130-registry-artifact-owner-mismatch` |
| `REGISTRY_VOLATILITY_BOUNDARY_VIOLATION` | `130` | `security_error` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.security_boundary` | `130.AnalysisErrorContext` | `error-registry-130-registry-volatility-boundary-violation` |
| `ANALYSIS_MUTATION_FORBIDDEN` | `130` | `security_error` | `caller_correctable` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.security_boundary` | `130.AnalysisErrorContext` | `error-registry-130-analysis-mutation-forbidden` |
| `LINEAGE_FACET_SCHEMA_MUTABLE` | `130` | `blocked` | `caller_correctable` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `130.AnalysisErrorContext` | `error-registry-130-lineage-facet-schema-mutable` |
| `LINEAGE_FACET_CHECKSUM_MISMATCH` | `130` | `error` | `caller_correctable` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `130.AnalysisErrorContext` | `error-registry-130-lineage-facet-checksum-mismatch` |
| `REGISTRY_AUTHORITY_FORBIDDEN` | `130` | `security_error` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.security_boundary` | `130.AnalysisErrorContext` | `error-registry-130-registry-authority-forbidden` |
| `THREAT_INTEL_IDENTITY_FORBIDDEN` | `130` | `security_error` | `caller_correctable` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.security_boundary` | `130.AnalysisErrorContext` | `error-registry-130-threat-intel-identity-forbidden` |
| `LINEAGE_FACET_POLICY_MISSING` | `130` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `130.AnalysisErrorContext` | `error-registry-130-lineage-facet-policy-missing` |
| `LINEAGE_FACET_NAMESPACE_COLLISION` | `130` | `error` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `130.AnalysisErrorContext` | `error-registry-130-lineage-facet-namespace-collision` |
| `THREAT_INTEL_PROFILE_MISSING` | `130` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `130.AnalysisErrorContext` | `error-registry-130-threat-intel-profile-missing` |
| `THREAT_INTEL_DISTRIBUTION_UNMAPPED` | `130` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `130.AnalysisErrorContext` | `error-registry-130-threat-intel-distribution-unmapped` |
| `THREAT_INTEL_ARTIFACT_CHECKSUM_MISMATCH` | `130` | `error` | `caller_correctable` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `130.AnalysisErrorContext` | `error-registry-130-threat-intel-artifact-checksum-mismatch` |
| `REGISTRY_CUSTOM_PROPERTY_SCHEMA_INVALID` | `130` | `error` | `caller_correctable` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `130.AnalysisErrorContext` | `error-registry-130-registry-custom-property-schema-invalid` |
| `REGISTRY_CLASSIFICATION_AUTHORITY_FORBIDDEN` | `130` | `security_error` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.security_boundary` | `130.AnalysisErrorContext` | `error-registry-130-registry-classification-authority-forbidden` |
| `DERIVED_GRAPH_EDGE_SUPPORTING_FACTS_REQUIRED` | `130` | `blocked` | `caller_correctable` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `130.AnalysisErrorContext` | `error-registry-130-derived-graph-edge-supporting-facts-required` |
| `DERIVED_GRAPH_EDGE_PROJECTION_FORBIDDEN` | `130` | `security_error` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.security_boundary` | `130.AnalysisErrorContext` | `error-registry-130-derived-graph-edge-projection-forbidden` |
| `DERIVED_GRAPH_EDGE_REPLAY_MISMATCH` | `130` | `error` | `retry_after_owner_repair` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `130.AnalysisErrorContext` | `error-registry-130-derived-graph-edge-replay-mismatch` |
| `ARTIFACT_CLASS_POLICY_MISSING` | `130` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `130.AnalysisErrorContext` | `error-registry-130-artifact-class-policy-missing` |
| `ARTIFACT_CLASS_POLICY_AMBIGUOUS` | `130` | `error` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `130.AnalysisErrorContext` | `error-registry-130-artifact-class-policy-ambiguous` |
| `ARTIFACT_SUBSTITUTION_FORBIDDEN` | `130` | `security_error` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.security_boundary` | `130.AnalysisErrorContext` | `error-registry-130-artifact-substitution-forbidden` |
| `ARTIFACT_AUTHORITY_CLASS_MISMATCH` | `130` | `security_error` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.security_boundary` | `130.AnalysisErrorContext` | `error-registry-130-artifact-authority-class-mismatch` |
| `ARTIFACT_EVIDENCE_REF_FORBIDDEN` | `130` | `security_error` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.security_boundary` | `130.AnalysisErrorContext` | `error-registry-130-artifact-evidence-ref-forbidden` |

#### AnalysisErrorContext

`AnalysisErrorContext` is the owner context schema for `130` analysis, enrichment, lineage, derived-edge, and registry-governance registry rows. It satisfies `110.OwnerErrorContextMinimumSchema` and must not expose raw facet bytes, raw threat-intel values, private activation scope, private source binding values, registry payload bytes, artifact payload bytes, or backend/query text to callers.

| Field | Required | Rule |
| --- | ---: | --- |
| `context_schema_version` | Yes | Immutable `130` context schema version. |
| `owner_spec` | Yes | Must be `130`. |
| `error_code` | Yes | Must match the generated registry row. |
| `failure_class` | Yes | Closed token: `registry_artifact`, `volatility_boundary`, `analysis_mutation`, `lineage_facet`, `artifact_class_policy`, `registry_authority`, `threat_intel`, `custom_property`, `classification`, `derived_edge`, or `replay`. |
| `operation` | Yes | Analysis execution, analysis query import, derivation-rule routing, derived-edge validation, threat-intel enrichment, lineage facet mapping, artifact-class validation, registry activation, custom-property validation, classification validation, or replay validation. |
| `affected_record_type` | Yes | Analysis rule bundle, analysis rule, finding, metric, risk acceptance record, derived edge rule, lineage facet policy, threat-intel artifact, registry artifact, custom property schema, classification policy, or version manifest. |
| `field_path` | Yes | Exact field path when applicable; null for artifact-wide failures. |
| `artifact_refs` | Yes | Canonically sorted refs to analysis rule bundles, rule compatibility matrices, enrichment profiles, lineage facet mappings, artifact-class policies, registry governance artifacts, graph query refs, mutation-prohibition proofs, validation fixtures, package-set refs, or version manifests consulted by the error; empty only when no artifact was consulted. |
| `validation_refs` | Yes | Exact `120` analysis, lineage, enrichment, registry, and derived-edge fixture refs. |
| `redaction_classes` | Yes | Map every nested owner-context field to one `110.ErrorRedactionClassMatrix` class. Raw facet bytes, raw threat-intel values, private bindings, registry payload bytes, artifact payload bytes, backend-native query text, provider-native query text, and raw payload bytes must map to `always_forbidden`. |
| `blocking_reason` | Yes when generated row severity is `blocked` | Bounded reason; otherwise null or omitted. |
| `attempted_authority_effect` | No | Required when the error blocks fact, identity, source-authority, package, graph, watermark, or registry authority substitution. |
| `attempted_mutation_class` | No | Required when an analysis, lineage, enrichment, or registry operation attempted mutation. |
| `package_set_ref` | No | Required when the failing artifact was package-supplied. |

### RuleGraphCompatibilityMatrix fixture expectations

| Field | Required behavior |
| --- | --- |
| graph projection profile checksum | Exact active `090.GraphProjectionProfile` checksum. |
| graph edge semantics row refs | Exact rows for every queried edge type; missing row blocks rule activation. |
| traversal class set | Closed set of `090.GraphTraversalClass` values. `observed_connection_path` must be named for observed-connection path reads. |
| graph object output eligibility row refs | Exact eligibility rows for every returned object class and analysis context. |
| graph property refs | Exact graph property policy refs for every property read by the rule. |
| graph query translation profile ref | Exact active `090.GraphQueryTranslationProfile` ref and checksum. |
| derived-view lag policy ref | Exact active `090.DerivedViewLagPolicy` ref; stale behavior must be reject or label per `090` and `110`. |
| authorization/redaction refs | Required for every rule execution that reads graph or evidence data. |
| expected query checksum | Canonical checksum over the translated read-only graph query. |
| expected result checksum | Canonical checksum over the authorized, redacted result set. |
| read-only mutation proof | Fixture must prove no raw, silver, identity, gold, graph-delta, graph-serving, completeness, watermark, package, or source-authority mutation. |

Default graph compatibility rules:

| Object or traversal | Default analysis behavior |
| --- | --- |
| `observed_connection` detail query | Readable only when the matrix names the edge type and output eligibility permits analysis read. |
| `observed_connection_path` | Readable only when the matrix names the traversal class and query translation profile. |
| `generic_external_graph_payload` | Cannot produce findings, metrics, identity influence, or pathfinding input. |
| derived-view stale state | Rejected for compliance/audit analysis by default; otherwise labeled only when `110` permits. |
| mutation attempt | Fails with `ANALYSIS_MUTATION_FORBIDDEN` before any owner state changes. |
| `sql_text` under PostgreSQL relational profile | Rejected unless mapped into an active `090.GraphQueryTranslationProfile` and compatibility row; SQL DML/DDL mutation proof required. |
| AGE `ag_catalog.cypher()` or SQL/Cypher composition | Rejected unless mapped into active PostgreSQL AGE translation rows; mutating Cypher clauses, namespace DML/DDL, and AGE internal ID leakage proofs required. |
| prepared statement or query-plan text | Validation-only diagnostic material; must not execute or export unless translated and redacted through owner profiles. |

### RiskScoringBoundary validation

Numeric scoring is disabled by default. Attempts to emit authoritative numeric risk or exposure scores must fail with `ANALYSIS_MUTATION_FORBIDDEN` or remain `AnalysisMetric` only when `RiskScoringBoundary.numeric_scoring_authority = disabled`.

### AnalysisRegistryActivationControlledRowFieldRule

Every output-affecting `130` activation-controlled row family must use the exact `030.ActivationControlledRowField` column order and must materialize defaults before row ID, row checksum, row-set checksum, validation output checksum, replay checksum, and owner algorithm execution.

The rows below are shared field defaults. Row-family-specific tables may narrow these values. They must not omit any output-affecting field path.

| field_path | type | required | default | null_allowed | omit_allowed | bounds | array_semantics | duplicate_policy | canonical_sort_key | id_input | checksum_input | extension_policy | redaction_owner | version_manifest_requirement | missing_error | invalid_error |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `row_id` | `040.ScalarType.string` | `yes` | `none` | `no` | `no` | non-empty stable row ID scoped to owner row set | `n/a` | `reject` | `n/a` | `ordered:1` | `yes` | `closed` | `110` | selected row ref and checksum required when row affects output | `REGISTRY_ARTIFACT_INACTIVE` | `REGISTRY_ARTIFACT_OWNER_MISMATCH` |
| `row_version` | `040.ScalarType.string` | `yes` | `none` | `no` | `no` | immutable owner version | `n/a` | `reject` | `n/a` | `ordered:2` | `yes` | `closed` | `110` | row version and checksum required | `REGISTRY_ARTIFACT_INACTIVE` | `REGISTRY_ARTIFACT_OWNER_MISMATCH` |
| `validation_refs` | `array<030.ActivationControlledArtifactRef>` | `yes` | `none` | `no` | `no` | non-empty owner validation refs for production rows | `canonical_set` | `reject` | `artifact_ref` | `no` | `yes` | `closed` | `110` | every validation ref and checksum required | `REGISTRY_ARTIFACT_INACTIVE` | `REGISTRY_VOLATILITY_BOUNDARY_VIOLATION` |
| `activation_scope` | `030.ActivationScope` | `yes` | `none` | `no` | `no` | must cover request scope through `130.AnalysisRegistryScopeSelectorContext` | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | selector context and checksum required | `REGISTRY_ARTIFACT_INACTIVE` | `REGISTRY_ARTIFACT_OWNER_MISMATCH` |
| `lifecycle_status` | `030.LifecycleStatus` | `yes` | `none` | `no` | `no` | production selection requires `active` unless row explicitly emits deterministic block/no-op | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | lifecycle transition evidence required | `REGISTRY_ARTIFACT_INACTIVE` | `REGISTRY_ARTIFACT_INACTIVE` |
| `package_set_ref` | `030.ActivationControlledArtifactRef` | `conditional:package-supplied row` | `null` | `yes` | `no` | required when package-supplied and null otherwise | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | package-set ref/checksum required when package-supplied | `REGISTRY_ARTIFACT_INACTIVE` | `REGISTRY_VOLATILITY_BOUNDARY_VIOLATION` |
| `row_checksum` | `040.ScalarType.sha256` | `yes` | `derived:canonical row bytes after defaults` | `no` | `no` | lowercase SHA-256 hex | `n/a` | `reject` | `n/a` | `no` | `no` | `closed` | `110` | row checksum required for selected row | `REGISTRY_ARTIFACT_INACTIVE` | `REGISTRY_ARTIFACT_OWNER_MISMATCH` |
| `row_set_checksum` | `040.ScalarType.sha256` | `yes` | `derived:canonical row-set bytes` | `no` | `no` | lowercase SHA-256 hex | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | row-set checksum required for selection | `REGISTRY_ARTIFACT_INACTIVE` | `REGISTRY_ARTIFACT_OWNER_MISMATCH` |
| `version_manifest_ref` | `030.VersionManifest` | `conditional:row affects output or visibility` | `none` | `no` | `no` | required before output, visibility, replay, package activation, validation acceptance, or diagnostics | `n/a` | `reject` | `n/a` | `no` | `yes` | `closed` | `110` | manifest ref required | `REGISTRY_ARTIFACT_INACTIVE` | `REGISTRY_VOLATILITY_BOUNDARY_VIOLATION` |

A `130` row-family table that omits an output-affecting field must fail `ValidateActivationControlledRowSet` before activation. Bare string row refs, implementation-local defaults, unbounded maps, unbounded arrays, duplicate canonical-set members, mutable artifact refs, and missing row-set checksums are invalid.

### ActivationControlledRowSchemaPrecisionHandoff

The following closure ledger is total for `130` row families named in this spec. A row family not listed below must be `non_production` or must be added to this ledger before it can affect production output, validation acceptance, package-visible diagnostics, API-visible diagnostics, replay, redaction, visibility, or `030.VersionManifest`.

| row_family | production classification | closure_state | required field-precision owner |
| --- | --- | --- | --- |
| `DerivedGraphEdgeRule` | output_affecting | `full_row_schema` | Existing `DerivedGraphEdgeRule schema`, source-dataset closure handoff, effect routing, and `030.VersionManifest` rows. |
| `ThreatIntelEnrichmentProfile` | enrichment_affecting | `full_row_schema` | `ThreatIntelEnrichmentProfile schema`. |
| `ThreatIntelEnrichmentRecord` | enrichment_runtime_record | `runtime_state_record_schema` | `ThreatIntelEnrichmentRecord schema`. |
| `ThreatIntelDistributionMappingPolicy` | enrichment_visibility_policy | `full_row_schema` | `ThreatIntelDistributionMappingPolicy`. |
| `ThreatIntelArtifactRef` | immutable_artifact_identity | `full_row_schema` | `ThreatIntelArtifactRef`. |
| `LineageFacetMappingRow` | lineage_mapping_affecting | `full_row_schema` | `LineageFacetMappingRow schema`. |
| `ArtifactClassPolicy` | artifact_substitution_affecting | `full_row_schema` | `ArtifactClassPolicy row schema`. |
| `RegistryArtifactGovernance` | registry_activation_affecting | `full_row_schema` | `RegistryArtifactGovernance schema`. |
| `RegistryCustomPropertySchema` | registry_metadata_affecting | `full_row_schema` | `RegistryCustomPropertySchema`. |
| `RegistryClassificationPolicy` | registry_metadata_affecting | `full_row_schema` | `RegistryClassificationPolicy`. |
| `AnalysisFinding`, `AnalysisMetric`, `RiskAcceptanceRecord` | analysis_output | `runtime_state_record_schema` | Analysis output authority table plus `AnalysisReplayFieldSelectionRow`. |
| `AnalysisErrorRegistryFragment` | visible_diagnostic_affecting | `full_row_schema` | `AnalysisErrorRegistryFragment` and `AnalysisErrorContext`. |

No row family in this ledger may grant fact, identity, source-authority, source-completeness, graph, package, watermark, production-approval, remediation, validation-acceptance, cleanup, retraction, or absence authority unless another active owner spec exports an exact named interface and the selected refs appear in `030.VersionManifest`.

`ValidateSpecSet` must fail when a selected `130` row family contains `TODO:`, lacks one required field precision row, uses prose-only schemas, uses a bare string row ref, omits selected row refs or checksums from `030.VersionManifest`, omits package-set refs when package-supplied, omits validation refs, lacks lifecycle evidence, lacks redaction refs where raw-derived values can be exposed, or lacks generated error-registry refs for visible diagnostics.

### Acceptance Criteria

| ID | Criterion |
| --- | --- |
| `130-ACTIVATION-ROW-SCHEMA-PRECISION-AC-001` | `ValidateActivationControlledRowSet` accepts every selected `130` row family only when field precision, row checksum, row-set checksum, validation refs, lifecycle refs, package-set refs when package-supplied, redaction refs where required, generated error-registry refs when visible, and `030.VersionManifest` refs are concrete and non-`TODO`. |
| `130-THREAT-INTEL-ROW-PRECISION-AC-001` | Threat-intel profile, distribution mapping, and artifact-ref row families fail when source format, ref arrays, defaults, duplicate handling, checksum inputs, unknown value behavior, deprecated artifact behavior, redaction refs, or non-authority effects are missing or invalid. |
| `130-THREAT-INTEL-RECORD-SCHEMA-AC-001` | `ThreatIntelEnrichmentRecord` emits deterministic `record_id`, defaults empty context maps to `{}`, restricts missing labels, disables export when required, and fails if indicator equality, sightings, taxonomies, galaxies, templates, confidence labels, or distribution labels create identity or authority. |
| `130-DISTRIBUTION-POLICY-AC-001` | `ResolveThreatIntelDistribution` proves no-label restricted visibility, unknown-label rejection, preserved-unknown restricted visibility, most-restrictive conflict selection, export decision determinism, diagnostics, and checksum output. |
| `130-LINEAGE-FACET-ROW-SCHEMA-AC-002` | `LineageFacetMappingRow` fixtures prove immutable schema URL, schema checksum, facet checksum policy, collision rejection, raw-facet redaction, mapped field set closure, non-authority defaults, freshness non-completeness, and manifest inclusion. |
| `130-ARTIFACT-CLASS-POLICY-AC-001` | `EvaluateArtifactClassSubstitution` rejects missing, ambiguous, authority-widening, forbidden, class/kind mismatched, package-set-missing, unmanifested, and validation-missing substitutions and permits only explicit same-or-narrower validated substitutions. |
| `130-REGISTRY-CUSTOM-PROPERTY-SCHEMA-AC-002` | Registry custom-property fixtures prove namespace bounds, path bounds, scalar bounds, enum bounds, cardinality, null default rejection, explicit null allowance, omit behavior, attachment targets, redaction class, authority-effect rejection, replay exactness, and replay mismatch. |
| `130-REGISTRY-CLASSIFICATION-POLICY-AC-001` | Registry classification fixtures prove valid label attachment, unknown label rejection, invalid target rejection, conflict behavior, redaction narrowing only, metadata-only export, authority-effect rejection, glossary non-authority, and manifest inclusion. |
| `130-REGISTRY-GOVERNANCE-MANIFEST-AC-001` | Registry governance activation fails when owner refs, artifact checksum, lifecycle evidence, approval metadata, authority-grant interface refs, package-set refs when package-supplied, redaction refs, validation refs, or manifest refs are missing, stale, checksum-mismatched, or `TODO`. |
| `130-ERROR-FRAGMENT-TOTAL-AC-002` | Every `130.AnalysisErrorRegistryFragment` row, including artifact-class-policy errors, expands to exactly one `110.ErrorCodeRegistryRow` with standard caller fields, standard audit fields, owner context, redaction rule, fixture ref, generated registry checksum, and manifest inclusion. |
| `130-LINEAGE-FACET-ROW-SCHEMA-AC-001` | `120` lineage fixture families prove valid row activation, missing policy rejection, mutable schema rejection, checksum mismatch rejection, namespace collision rejection, raw facet redaction, freshness non-completeness, and facet-only non-evidence behavior. |
| `130-THREAT-INTEL-PROFILE-SCHEMA-AC-001` | `120` threat-intel fixture families prove active profile success, missing profile rejection, unknown taxonomy handling, artifact checksum mismatch, sighting non-completeness, and identity-forbidden behavior. |
| `130-THREAT-INTEL-DISTRIBUTION-AC-001` | `120` threat-intel distribution fixture families prove missing distribution label restricted visibility, conflicting distribution most-restrictive selection, unmapped distribution rejection when required, and export disablement. |
| `130-REGISTRY-GOVERNANCE-SCHEMA-AC-001` | `120` registry fixture families prove activation success, inactive artifact rejection, owner mismatch rejection, package-set-ref requirement, private binding redaction, and authority-grant rejection. |
| `130-REGISTRY-CUSTOM-PROPERTY-AC-001` | `120` registry fixture families prove custom-property bounds, type, cardinality, default, redaction, attachment-target, authority-forbidden, and replay behavior. |
| `130-DERIVED-GRAPH-EDGE-RULE-AC-001` | `120` derived-edge fixture families prove supporting facts required, `090`-routed graph output, direct graph mutation rejection, `080`-routed gold output, exact replay, replay mismatch, and explicit no-op behavior. |
| `130-SOURCE-DATASET-CATALOG-AC-001` | Derived graph edge rules with graph or gold output reject or no-op when source-dataset catalog refs, source-authority closure refs, supporting fact refs, absence derivation result refs, mutation-prohibition refs, or `VersionManifest` refs are missing. |
| `130-SOURCE-EFFECT-NONAUTH-AC-001` | Analysis and derived-edge content cannot become an alternate source-effect authority path; unauthorized support emits no graph mutation, no gold fact, no metric/finding authority, and no watermark. |
| `130-ANALYSIS-OUTPUT-AUTHORITY-TOTAL-AC-001` | `120` analysis fixture families prove every output class in `AnalysisOutputAuthorityMatrix` is non-authoritative by default and cannot mutate raw, silver, identity, gold, graph, completeness, watermark, package, or source authority state. |
| `130-ERROR-FRAGMENT-TOTAL-AC-001` | `120` error-registry fixture refs cover every `130.AnalysisErrorRegistryFragment` row and `110.GenerateErrorCodeRegistry` rejects missing, duplicate, unknown severity, unknown retry class, unredacted, legacy `code` caller fields, owner-specific top-level caller fields, or unfixtured rows. |
| `130-API-HANDOFF-AC-001` | Analysis API handoff fixtures cover analysis mutation rejection, graph compatibility mismatch, stale derived-view rejection or allowed stale display, empty read-only output, authorization/redaction mismatch, and lineage facet checksum mismatch. |
| `130-CLEANUP-AC-001` | No banned reference class remains. |
| `130-CLEANUP-AC-002` | Analysis, enrichment, lineage, and registry records still cannot mutate facts, graph state, completeness, watermarks, identity, package state, or source authority unless another active NLSpec grants a named interface. |
| `130-CLEANUP-AC-003` | Analysis rules remain read-only unless routed through an owning derivation interface. |
| `130-CLEANUP-AC-004` | Threat-intel enrichment remains context by default and cannot become identity, source completeness, source authority, gold fact, graph edge, absence, cleanup, retraction, coverage, or watermark authority by itself. |
| `130-LINEAGE-FACET-AC-001` | Lineage facet rows define schema URL immutability, schema bytes, checksum, collision behavior, raw-facet storage, mapped fields, and rejection behavior. |
| `130-RISK-SCORING-AC-001` | Numeric scoring is disabled by default and cannot emit authoritative risk scores without a future accepted scoring policy. |
| `130-ANALYSIS-REPLAY-AC-001` | Analysis replay exact match includes rule bundle refs, rule row refs, graph compatibility refs, query target refs, derived-view refs, authorization/redaction refs, output checksum, and `VersionManifest` ref. |
| `130-ANALYSIS-REPLAY-AC-002` | Rule bundle mismatch, graph compatibility mismatch, derived-view mismatch, authorization mismatch, or mutation attempt rejects production replay before output. |
| `130-ANALYSIS-REPLAY-AC-003` | Shadow-only comparison is non-production and may not mutate facts, graph state, completeness, watermarks, identity, package state, or source authority. |
| `130-PROVIDER-NATIVE-QUERY-IMPORT-AC-001` | SQL, SQL/Cypher, AGE `ag_catalog.cypher()` string, prepared statement text, or query-plan text without translation fails before execution; valid translated read-only import passes with expected query checksum and expected result checksum; SQL DML/DDL and AGE mutating-Cypher attempts fail before graph/backend execution. |
| `130-VOLATILITY-AC-001` | Registry classification creating source authority, analysis rule bundle manifest omission, lineage facet checksum mismatch, threat-intel identity authority attempt, and derived graph edge mutation outside `090` fail before production effect. |
| `130-VOLATILITY-AC-002` | Registry activation records include owner, lifecycle, checksum, validation refs, activation scope, and package-set ref when package-supplied. |
| `130-LIFECYCLE-AC-001` | Every analysis, enrichment, lineage, and registry activation-controlled artifact entering `active` status has generic artifact lifecycle transition evidence and owner-specific guard results. |
| `130-GRAPH-COMPAT-AC-001` | Analysis rule activation fails when graph profile, traversal class, output eligibility, query translation, property, derived-view lag, or authorization refs mismatch. |
| `130-GRAPH-COMPAT-AC-002` | `observed_connection` is readable only through a named detail query or `observed_connection_path` traversal class permitted by output eligibility. |
| `130-GENERIC-PAYLOAD-AC-001` | Generic external graph payload cannot produce analysis findings, metrics, pathfinding input, or identity influence. |
| `130-DERIVED-VIEW-STALE-AC-001` | Stale graph derived-view state rejects or labels according to `090` and `110`; compliance/audit analysis rejects by default. |
| `130-ANALYSIS-MUTATION-AC-001` | Any analysis attempt to mutate authoritative or graph-serving state fails before mutation. |
| `130-EVIDENCE-ARTIFACT-SUBSTITUTION-AC-001` | Lineage facets cannot substitute for table snapshot evidence or source evidence without an exact owner artifact-class row. |
| `130-EVIDENCE-ARTIFACT-SUBSTITUTION-AC-002` | Registry labels, validation reports, analysis findings, and metrics cannot substitute for fact authority, source evidence, production evidence, graph truth, or package activation. |
| `130-EVIDENCE-ARTIFACT-SUBSTITUTION-AC-003` | Evidence refs to analysis, lineage, registry, validation, or freshness artifacts validate both `130.ArtifactClassPolicy` and `040.EvidenceArtifactClassRegistry` before visibility. |

### Structured input registry acceptance criteria

| ID | Criterion |
| --- | --- |
| `130-STRUCTURED-INPUT-REGISTRY-AC-001` | Repository-authored registry label, approval metadata, or governance artifact cannot create fact authority, source authority, graph authority, source completeness, evidence authority, package activation, or production approval. |
| `130-STRUCTURED-INPUT-REGISTRY-AC-002` | Repository-authored analysis rule or registry artifact cannot run or activate unless active bundle refs, compatibility rows, validation refs, materialization refs when packaged, package-set refs when package-supplied, and manifest refs pass. |
| `130-STRUCTURED-INPUT-REGISTRY-AC-004` | Git-only registry candidates, template-only activation, stale CI, publication mismatch, sync-only analysis execution, and private leak attempts fail before analysis, registry, derived-edge, package, source-authority, graph, or fact mutation. |

## Definition of Done

| ID | Criterion |
| --- | --- |
| `130-SCOPE-ANALYSIS-AC-001` | Derived-edge exact activation scope, out-of-scope no mutation, registry artifact scope mismatch, ambiguity rejection, and private activation scope leak fixtures pass. |
| `130-SCOPE-ANALYSIS-AC-002` | Analysis and registry scoped selection includes selector checksums and context refs in replay and validation artifacts when output can be affected. |
| `130-DOD-ROW-PRECISION-AC-001` | No selected `130` row family has owner-local placeholder cells in field types, defaults, bounds, checksums, expected outputs, expected errors, validation refs, package-set refs, redaction refs, lifecycle refs, generated error-registry refs, or manifest refs. |
| `130-DOD-MUTATION-PROHIBITION-AC-001` | `120` mutation-prohibition proofs cover threat-intel, lineage, artifact-class, registry governance, custom-property, classification, analysis, derived-edge, and error-registry output classes. |
| `130-AC-001` | Analysis rules cannot mutate authoritative or graph-serving state. |
| `130-AC-002` | Threat-intel enrichment cannot create identity or fact authority without a separate active contract. |
| `130-AC-003` | External lineage facets with mutable schema URLs, missing schema bytes, checksum mismatches, or namespace collisions are rejected. |
| `130-AC-004` | Registry labels and custom properties remain governance metadata unless another active spec grants authority. |
| `130-AC-005` | Derived graph edges require persisted supporting facts and an active derivation/projection path. |
| `130-AC-006` | All lineage, threat-intel, registry, analysis-output, derived-edge, and error-registry rows required by this spec have no owner-local placeholder cells. |

## Open Questions

Open questions marked `unresolved_question` block authoritative status for the affected contract. A downstream implementation must not resolve an unresolved owner decision by inference.
