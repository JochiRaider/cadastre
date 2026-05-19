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

Analysis, enrichment, lineage, and registry records must not mutate facts, graph state, completeness, watermarks, identity, package state, or source authority unless another active NLSpec grants a named interface.

## Analysis Rules

`AnalysisRuleBundle` contains read-only rules. A rule may query declared read models only when `RuleGraphCompatibilityMatrix` passes for the active graph projection profile, graph edge semantics row refs, traversal class set, graph object output eligibility row refs, graph property refs, query translation profile, derived-view lag policy, query class, node types, edge types, edge directions, temporal fields, authorization/redaction refs, expected query checksum, expected result checksum, and read-only mutation proof.

An `AnalysisRule` must not embed provider-native Gremlin, Cypher, AQL, or other backend query text as production graph behavior unless `ValidateAnalysisQueryImport` maps the query into an active `090.GraphQueryTranslationProfile` row and `RuleGraphCompatibilityMatrix` proves read-only behavior, expected result checksum, authorization/redaction behavior, and mutation prohibition. Provider-native query text is validation-only by default.

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

`DerivationRuleBundle` may emit `GoldFact` records only through the gold derivation interface owned by `080`. A derived graph edge rule may emit graph deltas only when the supporting facts, rule version, authority profile, completeness profile, deterministic ID inputs, and projection profile are persisted.

## Threat-Intel Enrichment

Threat-intel indicators, sightings, taxonomies, galaxies, object templates, confidence labels, distribution labels, sharing groups, and TLP-like classifications are enrichment context by default. They must be materialized through `ThreatIntelEnrichmentProfile`, `ThreatIntelEnrichmentRecord`, `ThreatIntelDistributionMappingPolicy`, and `ThreatIntelArtifactRef`.

They must not become identity, source completeness, source authority, gold fact, graph edge, graph serving, absence, cleanup, retraction, coverage, or watermark authority by themselves.

## Lineage and Artifact Boundary

`RunDatasetIOContract` and `LineageFacetMappingPolicy` may map external lineage facets to Cadastre lineage fields, diagnostic fields, non-authoritative metadata, or rejection. External lineage events and facets are non-authoritative by default.

`ArtifactClassPolicy` must prevent static DAG artifacts, executed-run artifacts, freshness artifacts, semantic artifacts, validation artifacts, lineage artifacts, table snapshot artifacts, table commit artifacts, and graph rebuild artifacts from substituting for one another.

## Registry Governance

`RegistryArtifactGovernance`, `RegistryCustomPropertySchema`, and `RegistryClassificationPolicy` manage owners, domains, classifications, glossary labels, policies, approval, lifecycle, custom properties, and checksums. Registry metadata must not define Cadastre fact authority, source authority, graph edge semantics, evidence refs, source completeness, or production approval by itself.

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
| `RegistryArtifactGovernance` | Owner, domain, lifecycle, checksum, approval, validation refs, activation scope present. | Registry metadata attempts production authority without owner spec. |

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

| Facet namespace | Schema URL immutability | Schema bytes | Checksum | Collision behavior | Raw-facet storage | Mapped fields | Rejection behavior |
| --- | --- | --- | --- | --- | --- | --- | --- |
| OpenLineage run facet | required immutable | required | SHA-256 over schema bytes and facet bytes | reject namespace collision by default | may be stored as non-authoritative metadata | run ID, job ref, dataset IO refs when policy maps them | `LINEAGE_FACET_SCHEMA_MUTABLE` or `LINEAGE_FACET_CHECKSUM_MISMATCH` |
| OpenLineage job facet | required immutable | required | SHA-256 over schema bytes and facet bytes | reject collision | non-authoritative metadata | job namespace/name metadata only | same |
| OpenLineage dataset facet | required immutable | required | SHA-256 over schema bytes and facet bytes | reject collision | non-authoritative metadata | dataset version refs only when owner policy maps them | same |
| custom facet | required immutable | required | SHA-256 over schema bytes and facet bytes | reject collision unless active policy grants namespace | raw facet may be stored as non-authoritative metadata | TODO owner-specific mapped fields required before production effect | `LINEAGE_FACET_SCHEMA_MUTABLE` or `LINEAGE_FACET_CHECKSUM_MISMATCH` |
| schema facet | required immutable | required | SHA-256 over schema bytes and facet bytes | reject collision | diagnostic only by default | schema diagnostic fields only | same |
| freshness facet | required immutable | required | SHA-256 over schema bytes and facet bytes | reject collision | diagnostic only | freshness diagnostic only; no completeness proof | same |
| source-code facet | required when used | required | SHA-256 over schema bytes and facet bytes | reject collision | non-authoritative metadata | source ref diagnostics only | same |
| parent-run facet | required when used | required | SHA-256 over schema bytes and facet bytes | reject collision | non-authoritative metadata | parent lineage metadata only | same |

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

### RegistryActivationPolicy

Registry governance artifacts require `030.ActivationControlledArtifactRef`. Registry governance artifacts may become active only when owner, domain, classification, glossary labels, lifecycle, checksum, approval, custom-property schema, validation rows, activation scope, and package-set ref when package-supplied are present. Registry metadata remains governance metadata unless another active spec grants authority. Registry metadata cannot grant authority unless an owner stable core contract grants the specific behavior.

### Analysis output authority table

| Output class | Default authority | Permitted downstream use | Prohibited mutations | Owner of exception | Validation fixture |
| --- | --- | --- | --- | --- | --- |
| `AnalysisFinding` | non-authoritative | UI, audit, workflow context | raw, silver, identity, gold, graph, completeness, watermark | future accepted spec | `analysis-finding-non-authority` |
| `AnalysisMetric` | non-authoritative | dashboards and reports | facts, risk authority, package state | future scoring contract | `analysis-metric-non-authority` |
| `RiskAcceptanceRecord` | workflow metadata | audit and workflow | remediation, retraction, risk reduction proof | future workflow spec | `risk-acceptance-no-remediation` |
| `ThreatIntelEnrichmentRecord` | enrichment context | context, filtering, analysis | identity, source authority, graph edge | future owner spec | `threat-intel-no-identity` |
| `RegistryArtifactGovernance` | governance metadata | ownership, approval, labels | fact, graph, source authority | owner spec | `registry-label-no-fact-authority` |

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
| `REGISTRY_ARTIFACT_INACTIVE` | Required analysis, lineage, enrichment, registry, classification, or custom-property artifact is not active. |
| `REGISTRY_ARTIFACT_OWNER_MISMATCH` | Registry artifact owner does not match the stable core behavior owner. |
| `REGISTRY_VOLATILITY_BOUNDARY_VIOLATION` | Registry metadata, lineage facet, analysis rule, or threat-intel artifact attempts to become fact, identity, graph, source, completeness, package, or watermark authority without owner contract. |
| `ANALYSIS_MUTATION_FORBIDDEN` | Analysis, enrichment, lineage, or registry output attempts forbidden mutation. |
| `LINEAGE_FACET_SCHEMA_MUTABLE` | Facet schema URL is mutable or not immutable under policy. |
| `LINEAGE_FACET_CHECKSUM_MISMATCH` | Schema bytes or facet bytes do not match recorded checksum. |
| `REGISTRY_AUTHORITY_FORBIDDEN` | Registry metadata attempts fact, graph, source authority, evidence, or production approval authority. |
| `THREAT_INTEL_IDENTITY_FORBIDDEN` | Threat-intel indicator/enrichment attempts identity authority. |

### AnalysisErrorRegistryFragment

This owner fragment feeds `110.GenerateErrorCodeRegistry`. `110` owns the generated caller-visible registry. This table must not render API output by itself. Rows with `TODO:` cells block authoritative promotion and must be resolved by the owning domain before `110-ERROR-REGISTRY-TOTAL-AC-001` can pass.

| error_code | owner_spec | default_severity | default_retry_class | caller_visible_fields | audit_visible_fields | redaction_rule | owner_context_schema_ref | fixture_family |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `REGISTRY_ARTIFACT_INACTIVE` | `130` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `130.AnalysisErrorContext` | `analysis-error-registry-artifact-inactive` |
| `REGISTRY_ARTIFACT_OWNER_MISMATCH` | `130` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `130.AnalysisErrorContext` | `analysis-error-registry-artifact-owner-mismatch` |
| `REGISTRY_VOLATILITY_BOUNDARY_VIOLATION` | `130` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `130.AnalysisErrorContext` | `analysis-error-registry-volatility-boundary-violation` |
| `ANALYSIS_MUTATION_FORBIDDEN` | `130` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `130.AnalysisErrorContext` | `analysis-error-analysis-mutation-forbidden` |
| `LINEAGE_FACET_SCHEMA_MUTABLE` | `130` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `130.AnalysisErrorContext` | `analysis-error-lineage-facet-schema-mutable` |
| `LINEAGE_FACET_CHECKSUM_MISMATCH` | `130` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `130.AnalysisErrorContext` | `analysis-error-lineage-facet-checksum-mismatch` |
| `REGISTRY_AUTHORITY_FORBIDDEN` | `130` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `130.AnalysisErrorContext` | `analysis-error-registry-authority-forbidden` |
| `THREAT_INTEL_IDENTITY_FORBIDDEN` | `130` | TODO: owner-confirm severity | TODO: owner-confirm retry class | TODO: caller field set | TODO: audit field set | TODO: redaction rule | `130.AnalysisErrorContext` | `analysis-error-threat-intel-identity-forbidden` |

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
| `gremlin_text` under JanusGraph default | Rejected unless translated into a declared `QueryGraph` class or validation-only import row; no production mutation; expected checksum required. |

### RiskScoringBoundary validation

Numeric scoring is disabled by default. Attempts to emit authoritative numeric risk or exposure scores must fail with `ANALYSIS_MUTATION_FORBIDDEN` or remain `AnalysisMetric` only when `RiskScoringBoundary.numeric_scoring_authority = disabled`.

### Acceptance Criteria

| ID | Criterion |
| --- | --- |
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
| `130-PROVIDER-NATIVE-QUERY-IMPORT-AC-001` | Gremlin analysis import without translation fails before execution; valid translated read-only import passes with expected result checksum; mutation attempts fail before graph/backend execution. |
| `130-VOLATILITY-AC-001` | Registry classification creating source authority, analysis rule bundle manifest omission, lineage facet checksum mismatch, threat-intel identity authority attempt, and derived graph edge mutation outside `090` fail before production effect. |
| `130-VOLATILITY-AC-002` | Registry activation records include owner, lifecycle, checksum, validation refs, activation scope, and package-set ref when package-supplied. |
| `130-LIFECYCLE-AC-001` | Every analysis, enrichment, lineage, and registry activation-controlled artifact entering `active` status has generic artifact lifecycle transition evidence and owner-specific guard results. |
| `130-GRAPH-COMPAT-AC-001` | Analysis rule activation fails when graph profile, traversal class, output eligibility, query translation, property, derived-view lag, or authorization refs mismatch. |
| `130-GRAPH-COMPAT-AC-002` | `observed_connection` is readable only through a named detail query or `observed_connection_path` traversal class permitted by output eligibility. |
| `130-GENERIC-PAYLOAD-AC-001` | Generic external graph payload cannot produce analysis findings, metrics, pathfinding input, or identity influence. |
| `130-DERIVED-VIEW-STALE-AC-001` | Stale graph derived-view state rejects or labels according to `090` and `110`; compliance/audit analysis rejects by default. |
| `130-ANALYSIS-MUTATION-AC-001` | Any analysis attempt to mutate authoritative or graph-serving state fails before mutation. |

## Definition of Done

| ID | Criterion |
| --- | --- |
| `130-AC-001` | Analysis rules cannot mutate authoritative or graph-serving state. |
| `130-AC-002` | Threat-intel enrichment cannot create identity or fact authority without a separate active contract. |
| `130-AC-003` | External lineage facets with mutable schema URLs, missing schema bytes, checksum mismatches, or namespace collisions are rejected. |
| `130-AC-004` | Registry labels and custom properties remain governance metadata unless another active spec grants authority. |
| `130-AC-005` | Derived graph edges require persisted supporting facts and an active derivation/projection path. |

## Open Questions

Open questions marked `TODO:` block authoritative status for the affected contract. A downstream implementation must not resolve a `TODO:` by inference.
