---
doc_id: CADASTRE-NLSPEC-130
title: Analysis, Enrichment, and Registry Governance
doc_type: candidate-nlspec
status: migration_active
generated_on: 2026-05-17
source_prd: docs/archive/PRD-Cadastre.revised-draft.md
source_prd_sha256: 99437d5ec12d52752a0003577ac37f8a6c6f1221ac3ae3b7cce713b003aeae55
---

## Authority

This document is a generated Cadastre NLSpec candidate. It is `migration_active` until the migration ledger marks its source rows complete and `docs/nlspec/120-validation-fixtures-and-acceptance.md` records passing acceptance evidence.

This document owns the contracts listed in `Exports`. Other active Cadastre NLSpecs may import those contracts by exact name and must not restate them.

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

## Migration finalization contracts

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
| TODO | required immutable | required | SHA-256 required | reject by default | raw facet may be stored as non-authoritative metadata | TODO | reject with lineage error |

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
| `AnalysisFinding` | non-authoritative | UI, audit, workflow context | raw, silver, identity, gold, graph, completeness, watermark | future accepted spec | TODO |
| `AnalysisMetric` | non-authoritative | dashboards and reports | facts, risk authority, package state | future scoring contract | TODO |
| `RiskAcceptanceRecord` | workflow metadata | audit and workflow | remediation, retraction, risk reduction proof | future workflow spec | TODO |
| `ThreatIntelEnrichmentRecord` | enrichment context | context, filtering, analysis | identity, source authority, graph edge | future owner spec | TODO |
| `RegistryArtifactGovernance` | governance metadata | ownership, approval, labels | fact, graph, source authority | owner spec | TODO |

### External lineage facet table

| External facet | Cadastre placement | Authority class | Required checksum | Rejected conditions |
| --- | --- | --- | --- | --- |
| run/job/dataset facet | `RunDatasetIOContract` metadata | non-authoritative lineage | schema bytes and facet bytes | mutable schema URL, missing bytes, checksum mismatch |
| custom facet | raw facet storage or mapped diagnostic field | non-authoritative metadata | schema and raw facet | namespace collision, missing policy row |
| schema facet | schema diagnostic only unless owner maps | non-authoritative metadata | schema bytes | mismatch, stale, missing owner |
| freshness facet | freshness diagnostic only | non-authoritative | facet bytes | use as completeness proof |

### Patch acceptance criteria

| ID | Criterion |
| --- | --- |
| `130-PATCH-AC-001` | Analysis rules cannot mutate authoritative or graph-serving state. |
| `130-PATCH-AC-002` | Risk/exposure scores are either explicitly non-authoritative or governed by an accepted scoring contract. |
| `130-PATCH-AC-003` | External lineage facets with mutable schema URLs, missing schema bytes, checksum mismatches, or namespace collisions are rejected. |
| `130-PATCH-AC-004` | Registry metadata remains governance metadata unless another active spec grants authority. |

## Definition of Done

| ID | Criterion |
| --- | --- |
| `130-AC-001` | Analysis rules cannot mutate authoritative or graph-serving state. |
| `130-AC-002` | Threat-intel enrichment cannot create identity or fact authority without a separate active contract. |
| `130-AC-003` | External lineage facets with mutable schema URLs, missing schema bytes, checksum mismatches, or namespace collisions are rejected. |
| `130-AC-004` | Registry labels and custom properties remain governance metadata unless another active spec grants authority. |
| `130-AC-005` | Derived graph edges require persisted supporting facts and an active derivation/projection path. |

## Source Traceability

| Source | Section or artifact | Location |
| --- | --- | --- |
| docs/archive/PRD-Cadastre.revised-draft.md | `RunDatasetIOContract` | lines 3630-3652 |
| docs/archive/PRD-Cadastre.revised-draft.md | `LineageFacetMappingPolicy` | lines 3653-3696 |
| docs/archive/PRD-Cadastre.revised-draft.md | `ArtifactClassPolicy` | lines 3697-3906 |
| docs/archive/PRD-Cadastre.revised-draft.md | `GraphPropertyEvidencePolicy` | lines 5651-5678 |
| docs/archive/PRD-Cadastre.revised-draft.md | `ThreatIntelEnrichmentProfile, ThreatIntelEnrichmentRecord, ThreatIntelDistributionMappingPolicy, and ThreatIntelArtifactRef` | lines 5679-5779 |
| docs/archive/PRD-Cadastre.revised-draft.md | `AnalysisRuleBundle, DerivationRuleBundle, and RuleGraphCompatibilityMatrix` | lines 8204-8389 |
| docs/archive/PRD-Cadastre.revised-draft.md | `AnalysisFinding, AnalysisMetric, and RiskAcceptanceRecord` | lines 8296-8359 |
| docs/archive/PRD-Cadastre.revised-draft.md | `DerivedGraphEdgeRule` | lines 8360-8389 |
| docs/archive/PRD-Cadastre.revised-draft.md | `RegistryArtifactGovernance, RegistryCustomPropertySchema, and RegistryClassificationPolicy` | lines 8666-8751 |
| docs/archive/PRD-Cadastre.revised-draft.md | `Analysis and Derivation Rule Interfaces` | lines 9991-10039 |
| docs/archive/PRD-Cadastre.revised-draft.md | `Analysis Output, Query Import, and Risk Acceptance Interfaces` | lines 10022-10039 |
| docs/archive/PRD-Cadastre.revised-draft.md | `Threat-Intel Enrichment Interface` | lines 10194-10214 |
| docs/archive/PRD-Cadastre.revised-draft.md | `Registry Governance Interface` | lines 10215-10231 |
| docs/archive/PRD-Cadastre.revised-draft.md | `Normalization, Enrichment, Lineage, and Registry Governance Defaults` | lines 10571-10596 |
| Decomposition plan | Current user prompt | Domain decomposition, disposition matrix, dependency model, gap ledger, and migration acceptance criteria. |

## Open Questions

Open questions marked `TODO:` block authoritative status for the affected contract. A downstream implementation must not resolve a `TODO:` by inference.
