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

An `AnalysisRule` must not embed provider-native Gremlin, Cypher, AQL, SQL, SQL/Cypher composition, AGE `ag_catalog.cypher()` strings, prepared statement text, query-plan text, or other backend query text as production graph behavior unless `ValidateAnalysisQueryImport` maps the query into an active `090.GraphQueryTranslationProfile` row and `RuleGraphCompatibilityMatrix` proves read-only behavior, expected query checksum, expected result checksum, authorization/redaction behavior, and mutation prohibition. Provider-native query text is validation-only by default.

PostgreSQL and AGE compatibility rows must name `provider = postgresql` or `provider = postgresql_age` before a rule can target those profiles. Every PostgreSQL/AGE-compatible analysis rule must include an expected translated query checksum, expected result checksum, mutation-prohibition proof for SQL DML/DDL, mutation-prohibition proof for AGE mutating Cypher clauses, authorization refs, redaction refs, derived-view state refs, and `030.VersionManifest` refs. A rule embedding raw SQL, AGE Cypher, SQL/Cypher composition, prepared statement text, query-plan text, or provider-native query handles without those rows fails before analysis execution.

`observed_connection` is analysis-readable only when the compatibility matrix names `observed_connection_path` or a read-only detail query and the active output eligibility row permits that context. `generic_external_graph_payload` must not produce findings, metrics, identity influence, or pathfinding input.

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

`ThreatIntelEnrichmentProfile` is an activation-controlled artifact. It permits enrichment record emission only; it does not grant fact, identity, source-authority, source-completeness, graph, package, coverage, absence, cleanup, retraction, or watermark effects.

| Field | Required | Default or omission behavior | Rule |
| --- | ---: | --- | --- |
| `profile_id` | Yes | none | Stable profile ID. |
| `profile_version` | Yes | none | Immutable owner version. |
| `source_format` | Yes | none | Closed token declared by the profile row set; unknown formats fail with `THREAT_INTEL_PROFILE_MISSING`. |
| `immutable_artifact_refs` | Yes | none | Non-empty `ThreatIntelArtifactRef` refs and checksums. |
| `object_template_refs` | Required when object templates are used | `[]` only when unused | Exact immutable refs. Unknown template behavior is row-controlled and never coerces to a known template. |
| `taxonomy_refs` | Required when taxonomies are used | `[]` only when unused | Exact immutable refs. Unknown values are preserved or rejected by row, never coerced. |
| `galaxy_refs` | Required when galaxies are used | `[]` only when unused | Exact immutable refs. Unknown values are preserved or rejected by row, never coerced. |
| `sighting_policy` | Yes | `observability_only` | Sightings are observability signals only and must not become source completeness. |
| `distribution_policy_ref` | Yes | none | Active `ThreatIntelDistributionMappingPolicy` ref. Missing or unmapped distribution fails or restricts by table below. |
| `unknown_value_behavior` | Yes | `preserve_as_unknown_enrichment_context` | Closed enum: `preserve_as_unknown_enrichment_context`, `reject_profile_row`. No coercion is permitted. |
| `deprecated_artifact_behavior` | Yes | `reject_for_production` | Deprecated artifact use fails unless an active owner row grants validation-only behavior. |
| `allowed_output_record_classes` | Yes | `ThreatIntelEnrichmentRecord` only | No other production record class may be emitted by the profile. |
| `graph_effect` | Yes | `none` | Must be `none` for MVP. |
| `identity_effect` | Yes | `none` | Must be `none` for MVP. |
| `authority_effect` | Yes | `none` | Must be `none` for MVP. |
| `validation_refs` | Yes | none | Non-empty refs to `120` threat-intel positive, rejection, distribution, redaction, no-authority, and replay rows. |
| `activation_scope` | Yes | none | `030.ActivationScope`; scope matching never grants authority beyond the explicit allowed output effect. |
| `lifecycle_status` | Yes | none | Production use requires `active`. |

### ThreatIntelEnrichmentRecord schema

| Field | Required | Default or omission behavior | Rule |
| --- | ---: | --- | --- |
| `record_id` | Yes | none | Deterministic ID over profile ref, artifact refs, indicator context, distribution mapping, redaction refs, and canonical enrichment bytes. |
| `profile_ref` | Yes | none | Active `ThreatIntelEnrichmentProfile` ref. |
| `artifact_refs` | Yes | none | Non-empty immutable `ThreatIntelArtifactRef` refs and checksums. |
| `indicator_context` | No | `{}` | Enrichment context only; indicator equality must not create identity evidence. |
| `sighting_context` | No | `{}` | Observability only; must not satisfy source completeness or coverage. |
| `taxonomy_context` | No | `{}` | Raw unknown values preserved when profile permits. |
| `galaxy_context` | No | `{}` | Raw unknown values preserved when profile permits. |
| `object_template_context` | No | `{}` | Object-template values are bounded by the active profile. |
| `distribution_label` | No | `restricted_visibility` when missing | Missing label disables export. Conflicts select the most restrictive label. |
| `visibility_class` | Yes | Derived from distribution mapping | Must not expose values beyond the selected mapping and `110.RedactionPolicy`. |
| `redaction_policy_ref` | Yes | none | Required before raw or raw-derived threat-intel values are stored or exposed. |
| `authority_class` | Yes | `non_authoritative_analysis` | Other authority classes are forbidden unless imported by exact active owner interface. |
| `version_manifest_ref` | Yes for production-visible output | none | Must include profile, artifact, distribution, redaction, and validation refs. |

### ThreatIntelDistributionMappingPolicy

| Condition | Default behavior | Required diagnostic |
| --- | --- | --- |
| Unknown taxonomy, galaxy, category, type, or distribution value | Preserve raw value as unknown enrichment context or reject by explicit profile row. No coercion. | Owner diagnostic naming raw-value redaction class and profile row. |
| Missing distribution label | Restricted visibility; export disabled. | Distribution omission diagnostic. |
| Conflicting distribution labels | Most restrictive label wins. | Emit diagnostic naming all labels after redaction. |
| Sightings | Observability signal only; not source completeness. | No completeness, absence, cleanup, retraction, coverage, or watermark effect. |
| Indicator equality | Enrichment context only; not identity evidence or identity merge authority. | `THREAT_INTEL_IDENTITY_FORBIDDEN` for identity attempt. |
| Graph effect | `none`. | Graph output attempts fail with `REGISTRY_VOLATILITY_BOUNDARY_VIOLATION`. |
| Identity effect | `none`. | Identity attempts fail with `THREAT_INTEL_IDENTITY_FORBIDDEN`. |
| Authority effect | `none`. | Authority attempts fail with `REGISTRY_VOLATILITY_BOUNDARY_VIOLATION`. |

### ThreatIntelArtifactRef

| Field | Required rule |
| --- | --- |
| `artifact_ref` | Immutable artifact identity. Mutable refs, branches, tags, and `latest` are forbidden. |
| `artifact_format` | Closed token declared by active profile. |
| `artifact_version` | Immutable version or checksum-addressed version. |
| `artifact_bytes_checksum` | SHA-256 over exact artifact bytes. |
| `artifact_schema_ref` | Required when profile uses schema validation. |
| `redaction_policy_ref` | Required when raw or raw-derived values may be stored. |
| `lifecycle_status` | Production use requires `active`. |
| `validation_refs` | Non-empty refs to `120` checksum, deprecation, unknown-value, redaction, and no-authority rows. |

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

| Field | Required | Default or omission behavior | Rule |
| --- | ---: | --- | --- |
| `governance_id` | Yes | none | Stable ID for one registry governance artifact. |
| `owner_spec` | Yes | none | Stable owner spec that exports the behavior or vocabulary being governed. |
| `artifact_class` | Yes | none | Must match one closed `030.ActivationControlledArtifactRef.artifact_class`. |
| `artifact_checksum` | Yes | none | SHA-256 over exact registry artifact bytes or row set. |
| `lifecycle_status` | Yes | none | Production use requires `active`. |
| `governance_purpose` | Yes | none | Closed purpose token: `ownership`, `classification`, `glossary`, `custom_property`, `approval_metadata`, `artifact_boundary`, `registry_context`. |
| `classification_refs` | No | `[]` | Exact `RegistryClassificationPolicy` refs when labels affect redaction or export. |
| `glossary_refs` | No | `[]` | Governance metadata only. |
| `custom_property_schema_refs` | No | `[]` | Exact `RegistryCustomPropertySchema` refs for every custom property path. |
| `approval_refs` | Yes | none | Approval metadata refs; approval by itself does not grant production activation. |
| `activation_scope` | Yes | none | `030.ActivationScope`; scope matching never grants authority beyond the explicit allowed output effect. |
| `package_set_ref` | Required when package-supplied | null otherwise | Immutable `100.ProductionPackageSetManifest` ref. |
| `authority_grant_ref` | No | null | Non-null is valid only when another active owner spec exports the exact authority interface and validation refs pass. |
| `validation_refs` | Yes | none | Non-empty refs to `120` registry activation, authority rejection, custom-property, classification, redaction, package-set, and replay rows. |

`authority_grant_ref = null` by default. A non-null authority grant is valid only when another active owner spec exports the exact authority interface and validation refs pass. Otherwise activation fails with `REGISTRY_AUTHORITY_FORBIDDEN` or `REGISTRY_VOLATILITY_BOUNDARY_VIOLATION`.

### RegistryCustomPropertySchema

| Field | Required | Default or omission behavior | Rule |
| --- | ---: | --- | --- |
| `schema_id` | Yes | none | Stable custom-property schema ID. |
| `namespace` | Yes | none | Globally unique namespace in the active registry row set. |
| `property_path` | Yes | none | Dot-separated property path. Empty segments and wildcards are forbidden. |
| `scalar_type` | Yes | none | One `040.ScalarType` or bounded enum declared by this schema. |
| `cardinality` | Yes | `one` | Closed enum: `one`, `optional_one`, `array`, `map`. |
| `bounds` | Yes | none | Every string, array, map, object, and enum property must declare explicit minimum and maximum bounds or allowed value set. |
| `default_value` | Required when omitted value affects output | null when no default exists | Default must validate against scalar type and bounds. |
| `redaction_class` | Yes | none | One `110.RedactionDataClassMatrix` value or owner-declared subclass imported by `110`. |
| `allowed_attachment_targets` | Yes | none | Closed set of registry artifact classes; wildcards are forbidden. |
| `authority_effect` | Yes | `none` | Must be `none` unless another owner spec grants the exact authority interface. |
| `validation_refs` | Yes | none | Non-empty refs to bounds, unknown field, redaction, attachment-target, authority-forbidden, and replay rows. |
| `activation_scope` | Yes | none | `030.ActivationScope`; scope matching never grants authority beyond the explicit allowed output effect. |
| `lifecycle_status` | Yes | none | Production use requires `active`. |

### RegistryClassificationPolicy

| Field | Required | Default or omission behavior | Rule |
| --- | ---: | --- | --- |
| `policy_id` | Yes | none | Stable classification policy ID. |
| `label_namespace` | Yes | none | Globally unique namespace inside the active policy row set. |
| `allowed_labels` | Yes | none | Non-empty closed label set. Unknown labels fail before attachment. |
| `attachment_targets` | Yes | none | Closed set of registry artifact classes; wildcards are forbidden. |
| `redaction_effect` | Yes | `none` | May narrow visibility only through `110.RedactionPolicy`. |
| `export_effect` | Yes | `metadata_only` | Must not grant fact, source, graph, completeness, identity, package, approval, or watermark authority. |
| `authority_effect` | Yes | `none` | Non-`none` fails with `REGISTRY_CLASSIFICATION_AUTHORITY_FORBIDDEN` unless imported by exact owner interface. |
| `validation_refs` | Yes | none | Non-empty refs to label, attachment, redaction, authority-forbidden, and replay rows. |
| `activation_scope` | Yes | none | `030.ActivationScope`; scope matching never grants authority beyond the explicit allowed output effect. |
| `lifecycle_status` | Yes | none | Production use requires `active`. |

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

`LineageFacetMappingPolicy` is an activation-controlled row set. Production lineage effects require a `LineageFacetMappingRow` whose schema, checksum, namespace, collision behavior, redaction behavior, validation refs, activation scope, and lifecycle status validate before output.

| Facet namespace | Schema URL immutability | Schema bytes | Checksum | Collision behavior | Raw-facet storage | Mapped fields | Rejection behavior |
| --- | --- | --- | --- | --- | --- | --- | --- |
| OpenLineage run facet | required immutable | required | SHA-256 over schema bytes and canonical facet bytes | reject namespace collision by default | may be stored as non-authoritative metadata | run ID, job ref, dataset IO refs when policy maps them | `LINEAGE_FACET_SCHEMA_MUTABLE`, `LINEAGE_FACET_CHECKSUM_MISMATCH`, or `LINEAGE_FACET_POLICY_MISSING` |
| OpenLineage job facet | required immutable | required | SHA-256 over schema bytes and canonical facet bytes | reject collision | non-authoritative metadata | job namespace/name metadata only | same |
| OpenLineage dataset facet | required immutable | required | SHA-256 over schema bytes and canonical facet bytes | reject collision | non-authoritative metadata | dataset version refs only when owner policy maps them | same |
| custom facet | required immutable | required | SHA-256 over schema bytes and canonical facet bytes | reject collision unless active policy grants namespace | raw facet may be stored only as non-authoritative metadata | Custom facets are rejected for production effect by default. They may be stored only as non-authoritative metadata when the row declares immutable schema bytes, namespace ownership, bounded field types, collision behavior, redaction behavior, validation refs, activation scope, and lifecycle status. | `LINEAGE_FACET_POLICY_MISSING`, `LINEAGE_FACET_SCHEMA_MUTABLE`, `LINEAGE_FACET_NAMESPACE_COLLISION`, or `LINEAGE_FACET_CHECKSUM_MISMATCH` |
| schema facet | required immutable | required | SHA-256 over schema bytes and canonical facet bytes | reject collision | diagnostic only by default | schema diagnostic fields only | same |
| freshness facet | required immutable | required | SHA-256 over schema bytes and canonical facet bytes | reject collision | diagnostic only | freshness diagnostic only; no completeness proof | same |
| source-code facet | required when used | required | SHA-256 over schema bytes and canonical facet bytes | reject collision | non-authoritative metadata | source ref diagnostics only | same |
| parent-run facet | required when used | required | SHA-256 over schema bytes and canonical facet bytes | reject collision | non-authoritative metadata | parent lineage metadata only | same |

#### LineageFacetMappingRow schema

| Field | Required | Default or omission behavior | Rule |
| --- | ---: | --- | --- |
| `row_id` | Yes | none | Stable ID scoped to the policy row set. |
| `lineage_system` | Yes | none | Closed enum. MVP values: `openlineage`, `cadastre_internal_lineage`. Unknown systems fail with `LINEAGE_FACET_POLICY_MISSING`. |
| `facet_namespace` | Yes | none | Required and globally unique inside the active row set. Namespace collision fails with `LINEAGE_FACET_NAMESPACE_COLLISION` unless an active collision rule exists. |
| `facet_name` | Yes | none | Required exact facet name. Wildcards are forbidden. |
| `schema_url` | Yes | none | Required immutable URL or immutable schema ref. Branch names, mutable tags, `latest`, and missing schema refs fail with `LINEAGE_FACET_SCHEMA_MUTABLE`. |
| `schema_bytes_checksum` | Yes | none | SHA-256 over exact schema bytes. |
| `facet_bytes_checksum_policy` | Yes | none | SHA-256 over canonical facet bytes. |
| `collision_behavior` | No | `reject` | Closed enum: `reject`, `reject_unless_same_checksum`, `owner_explicit_namespace_override`. |
| `raw_facet_storage` | No | `store_as_non_authoritative_metadata` | Raw facet bytes must not become evidence, completeness proof, source authority, identity, fact, graph, package, or watermark state. |
| `mapped_field_set` | No | `[]` | Closed placement enum. Empty means raw non-authoritative metadata only. |
| `authority_class` | No | `non_authoritative_analysis` | `supporting_evidence` is allowed only when another owner imports it by exact authority interface. |
| `allowed_effects` | No | `[]` | Fact, identity, completeness, graph, package, or watermark effects are forbidden for MVP. |
| `redaction_policy_ref` | Required when raw facet bytes or raw values may be stored | none | Missing redaction policy rejects production storage. |
| `validation_refs` | Yes | none | Non-empty refs to `120` lineage facet schema, checksum, collision, redaction, no-authority, and replay rows. |
| `activation_scope` | Yes | none | `030.ActivationScope`; scope matching never grants authority beyond the explicit allowed output effect. |
| `lifecycle_status` | Yes | none | Production use requires `active`. |

### ArtifactClassPolicy substitution matrix

| Artifact class | Volatility class | Required owner spec | Must not substitute for |
| --- | --- | --- | --- |
| static DAG artifact | `activation_controlled_artifact` or `runtime_state_record` as owner declares | `130` unless imported owner is named | executed run, validation, freshness, table snapshot, graph rebuild |
| executed run artifact | `activation_controlled_artifact` or `runtime_state_record` as owner declares | `130` unless imported owner is named | static DAG, source completeness, validation, table snapshot |
| freshness artifact | `activation_controlled_artifact` or `runtime_state_record` as owner declares | `130` unless imported owner is named | completeness, absence, authority, table snapshot |
| semantic artifact | `activation_controlled_artifact` or `runtime_state_record` as owner declares | `130` unless imported owner is named | identity, gold, graph, source authority |
| validation artifact | `activation_controlled_artifact` or `runtime_state_record` as owner declares | `130` unless imported owner is named | production evidence, source evidence, source completeness |
| lineage artifact | `activation_controlled_artifact` or `runtime_state_record` as owner declares | `130` unless imported owner is named | evidence, completeness, table snapshot, graph rebuild |
| table snapshot artifact | `activation_controlled_artifact` or `runtime_state_record` as owner declares | `130` unless imported owner is named | fact time, source time, graph rebuild correctness by itself |
| table commit artifact | `activation_controlled_artifact` or `runtime_state_record` as owner declares | `130` unless imported owner is named | fact time, source authority, source completeness by itself |
| graph rebuild artifact | `activation_controlled_artifact` or `runtime_state_record` as owner declares | `130` unless imported owner is named | source truth, identity truth, completeness, validation by itself |
| structured input repository artifact | `runtime_state_record`, `activation_controlled_artifact`, or `validation_artifact` as owner declares | `030`, `100`, `110`, or `120` by contract | package artifact, validation acceptance, source evidence, owner row set, graph rebuild evidence, or production approval |
| evidence artifact class row set | `activation_controlled_artifact` | `040` for stable class/kind pairing; owner spec for class-specific rows | new `artifact_id.kind` values, source evidence authority, production evidence by itself |

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

#### AnalysisErrorContext

`AnalysisErrorContext` is the owner context schema for `130` analysis, enrichment, lineage, derived-edge, and registry-governance registry rows. It satisfies `110.OwnerErrorContextMinimumSchema` and must not expose raw facet bytes, raw threat-intel values, private activation scope, private source binding values, registry payload bytes, artifact payload bytes, or backend/query text to callers.

| Field | Required | Rule |
| --- | ---: | --- |
| `context_schema_version` | Yes | Immutable `130` context schema version. |
| `owner_spec` | Yes | Must be `130`. |
| `error_code` | Yes | Must match the generated registry row. |
| `failure_class` | Yes | Closed token: `registry_artifact`, `volatility_boundary`, `analysis_mutation`, `lineage_facet`, `registry_authority`, `threat_intel`, `custom_property`, `classification`, `derived_edge`, or `replay`. |
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
| `gremlin_text` under explicit non-default JanusGraph | Rejected unless translated into a declared `QueryGraph` class or validation-only import row; no production mutation; expected checksum required. |
| `sql_text` under PostgreSQL relational profile | Rejected unless mapped into an active `090.GraphQueryTranslationProfile` and compatibility row; SQL DML/DDL mutation proof required. |
| AGE `ag_catalog.cypher()` or SQL/Cypher composition | Rejected unless mapped into active PostgreSQL AGE translation rows; mutating Cypher clauses, namespace DML/DDL, and AGE internal ID leakage proofs required. |
| prepared statement or query-plan text | Validation-only diagnostic material; must not execute or export unless translated and redacted through owner profiles. |

### RiskScoringBoundary validation

Numeric scoring is disabled by default. Attempts to emit authoritative numeric risk or exposure scores must fail with `ANALYSIS_MUTATION_FORBIDDEN` or remain `AnalysisMetric` only when `RiskScoringBoundary.numeric_scoring_authority = disabled`.

### ActivationControlledRowSchemaPrecisionHandoff

The following `130` row families can affect analysis execution, derived graph edge output, threat-intel enrichment, lineage facet mapping, artifact-class substitution, registry governance, custom property validation, or classification policy. Each output-affecting family must use a complete `030.ActivationControlledRowField` table before production selection. Until the required table is present and non-`TODO`, `ValidateSpecSet` must classify the family as `blocked_validation`.

| row_family | production classification | required precision status |
| --- | --- | --- |
| `DerivedGraphEdgeRule` | output_affecting | Closed for source-effect routing by the existing schema plus required source-dataset catalog refs, source-authority closure matrix refs, absence derivation result refs, mutation-prohibition refs, structured row refs, allowed output effect, unsupported behavior, deterministic ID inputs, validation refs, activation scope, lifecycle status, and manifest inclusion. Remaining threat-intel, lineage, registry, and custom-property precision work remains blocked until separately closed. |
| `ThreatIntelEnrichmentProfile` | enrichment_affecting | TODO: add full field precision for permitted formats, indicator normalization, sighting semantics, visibility policy, output restrictions, ref arrays, and validation refs. |
| `ThreatIntelEnrichmentRecord` | enrichment_runtime_record | TODO: declare runtime-state field precision or full row precision for indicator, sighting, taxonomy, galaxy, object-template, confidence, distribution, and checksum behavior. |
| `RegistryArtifactGovernance` | output_affecting for registry activation | TODO: add full field precision for owner, domain, classification, glossary, policy, approval, lifecycle, and checksum metadata. |
| `RegistryCustomPropertySchema` | output_affecting | TODO: add null/omit rules, cardinality, array/map semantics, default value behavior, bounds, redaction owner, extension policy, and invalid errors. |
| `RegistryClassificationPolicy` | output_affecting for governance metadata | TODO: add full field precision for label vocabularies, non-authority defaults, permitted uses, redaction, and invalid errors. |
| `LineageFacetMappingRow` | output_affecting when lineage facets are mapped | TODO: add full field precision for immutable schema refs, schema checksum, facet bytes checksum policy, collision behavior, raw facet storage, mapped field sets, authority class, and allowed effects. |
| `ArtifactClassPolicy` | output_affecting for artifact substitution checks | TODO: add full field precision for artifact class, permitted substitutes, forbidden substitutes, authority class, validation refs, and error mapping. |

`DerivedGraphEdgeRule.required_*_refs` must use structured `030.ActivationControlledRowRef` or `030.ActivationControlledArtifactRef` objects. `allowed_output_effect` and `unsupported_behavior` must be closed owner enums with explicit defaults and checksum inclusion. Threat-intel profile ref arrays must use `canonical_set` semantics and duplicate rejection.

Analysis, enrichment, lineage, registry activation, or derived-edge routing must fail before output or mutation when any selected `130` row family remains `TODO:`, attempts hidden authority grant, uses a bare string row ref, or omits selected row refs from `030.VersionManifest`.

### Acceptance Criteria

| ID | Criterion |
| --- | --- |
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
| `130-PROVIDER-NATIVE-QUERY-IMPORT-AC-001` | Gremlin, SQL, SQL/Cypher, AGE `ag_catalog.cypher()` string, prepared statement text, or query-plan text without translation fails before execution; valid translated read-only import passes with expected query checksum and expected result checksum; SQL DML/DDL and AGE mutating-Cypher attempts fail before graph/backend execution. |
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
| `130-AC-001` | Analysis rules cannot mutate authoritative or graph-serving state. |
| `130-AC-002` | Threat-intel enrichment cannot create identity or fact authority without a separate active contract. |
| `130-AC-003` | External lineage facets with mutable schema URLs, missing schema bytes, checksum mismatches, or namespace collisions are rejected. |
| `130-AC-004` | Registry labels and custom properties remain governance metadata unless another active spec grants authority. |
| `130-AC-005` | Derived graph edges require persisted supporting facts and an active derivation/projection path. |
| `130-AC-006` | All lineage, threat-intel, registry, analysis-output, derived-edge, and error-registry rows required by this spec have no owner-local placeholder cells. |

## Open Questions

Open questions marked `unresolved_question` block authoritative status for the affected contract. A downstream implementation must not resolve an unresolved owner decision by inference.
