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

## Non-Authority Rule

Analysis, enrichment, lineage, and registry records must not mutate facts, graph state, completeness, watermarks, identity, package state, or source authority unless another active NLSpec grants a named interface.

## Analysis Rules

`AnalysisRuleBundle` contains read-only rules. A rule may query declared read models only when `RuleGraphCompatibilityMatrix` passes for the active graph projection profile, query class, node types, edge types, edge directions, graph properties, temporal fields, and expected result hashes.

`AnalysisFinding`, `AnalysisMetric`, and `RiskAcceptanceRecord` are workflow outputs. They must not represent remediation, risk reduction, fact retraction, graph edge removal, or source completeness change.

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

| Artifact class | Must not substitute for |
| --- | --- |
| static DAG artifact | executed run, validation, freshness, table snapshot, graph rebuild |
| executed run artifact | static DAG, source completeness, validation, table snapshot |
| freshness artifact | completeness, absence, authority, table snapshot |
| semantic artifact | identity, gold, graph, source authority |
| validation artifact | production evidence, source evidence, source completeness |
| lineage artifact | evidence, completeness, table snapshot, graph rebuild |
| table snapshot artifact | fact time, source time, graph rebuild correctness by itself |
| table commit artifact | fact time, source authority, source completeness by itself |
| graph rebuild artifact | source truth, identity truth, completeness, validation by itself |

### RegistryActivationPolicy

Registry governance artifacts may become active only when owner, domain, classification, glossary labels, lifecycle, checksum, approval, custom-property schema, and validation rows are present. Registry metadata remains governance metadata unless another active spec grants authority.

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

### Analysis and lineage error codes

| Error code | Emitted when |
| --- | --- |
| `ANALYSIS_MUTATION_FORBIDDEN` | Analysis, enrichment, lineage, or registry output attempts forbidden mutation. |
| `LINEAGE_FACET_SCHEMA_MUTABLE` | Facet schema URL is mutable or not immutable under policy. |
| `LINEAGE_FACET_CHECKSUM_MISMATCH` | Schema bytes or facet bytes do not match recorded checksum. |
| `REGISTRY_AUTHORITY_FORBIDDEN` | Registry metadata attempts fact, graph, source authority, evidence, or production approval authority. |
| `THREAT_INTEL_IDENTITY_FORBIDDEN` | Threat-intel indicator/enrichment attempts identity authority. |

### RuleGraphCompatibilityMatrix fixture expectations

| Field | Required behavior |
| --- | --- |
| required graph profile | Exact graph projection profile checksum. |
| required node/edge types | Closed set; missing type blocks rule activation. |
| expected query checksum | Canonical checksum over translated read-only graph query. |
| stale graph behavior | Reject or label according to `090` and `110`; compliance/audit queries reject by default. |
| read-only guarantee | Fixture must prove no raw, silver, identity, gold, graph-serving, completeness, watermark, or package mutation. |

### RiskScoringBoundary validation

Numeric scoring is disabled by default. Attempts to emit authoritative numeric risk or exposure scores must fail with `ANALYSIS_MUTATION_FORBIDDEN` or remain `AnalysisMetric` only when `RiskScoringBoundary.numeric_scoring_authority = disabled`.

### Acceptance Criteria

| ID | Criterion |
| --- | --- |
| `130-CLEANUP-AC-001` | No banned reference class remains. |
| `130-CLEANUP-AC-002` | Analysis, enrichment, lineage, and registry records still cannot mutate facts, graph state, completeness, watermarks, identity, package state, or source authority unless another active NLSpec grants a named interface. |
| `130-CLEANUP-AC-003` | Analysis rules remain read-only unless routed through an owning derivation interface. |
| `130-CLEANUP-AC-004` | Threat-intel enrichment remains context by default and cannot become identity, source completeness, source authority, gold fact, graph edge, absence, cleanup, retraction, coverage, or watermark authority by itself. |

| `130-LINEAGE-FACET-AC-001` | Lineage facet rows define schema URL immutability, schema bytes, checksum, collision behavior, raw-facet storage, mapped fields, and rejection behavior. |
| `130-RISK-SCORING-AC-001` | Numeric scoring is disabled by default and cannot emit authoritative risk scores without a future accepted scoring policy. |

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
