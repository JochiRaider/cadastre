---
doc_id: CADASTRE-NLSPEC-130
title: Analysis, Enrichment, and Registry Governance
doc_type: authoritative-nlspec
status: migration_active
generated_on: 2026-05-17
source_prd: PRD-Cadastre.md
source_prd_sha256: b17ac5d44618c43a57efe8ebd9b6c6e0bd8debc949b513368383c515c07f9748
---

# Analysis, Enrichment, and Registry Governance

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
| PRD-Cadastre.md | `RunDatasetIOContract` | lines 3630-3652 |
| PRD-Cadastre.md | `LineageFacetMappingPolicy` | lines 3653-3696 |
| PRD-Cadastre.md | `ArtifactClassPolicy` | lines 3697-3906 |
| PRD-Cadastre.md | `GraphPropertyEvidencePolicy` | lines 5651-5678 |
| PRD-Cadastre.md | `ThreatIntelEnrichmentProfile, ThreatIntelEnrichmentRecord, ThreatIntelDistributionMappingPolicy, and ThreatIntelArtifactRef` | lines 5679-5779 |
| PRD-Cadastre.md | `AnalysisRuleBundle, DerivationRuleBundle, and RuleGraphCompatibilityMatrix` | lines 8204-8389 |
| PRD-Cadastre.md | `AnalysisFinding, AnalysisMetric, and RiskAcceptanceRecord` | lines 8296-8359 |
| PRD-Cadastre.md | `DerivedGraphEdgeRule` | lines 8360-8389 |
| PRD-Cadastre.md | `RegistryArtifactGovernance, RegistryCustomPropertySchema, and RegistryClassificationPolicy` | lines 8666-8751 |
| PRD-Cadastre.md | `Analysis and Derivation Rule Interfaces` | lines 9991-10039 |
| PRD-Cadastre.md | `Analysis Output, Query Import, and Risk Acceptance Interfaces` | lines 10022-10039 |
| PRD-Cadastre.md | `Threat-Intel Enrichment Interface` | lines 10194-10214 |
| PRD-Cadastre.md | `Registry Governance Interface` | lines 10215-10231 |
| PRD-Cadastre.md | `Normalization, Enrichment, Lineage, and Registry Governance Defaults` | lines 10571-10596 |
| Decomposition plan | Current user prompt | Domain decomposition, disposition matrix, dependency model, gap ledger, and migration acceptance criteria. |

## Open Questions

Open questions marked `TODO:` block authoritative status for the affected contract. A downstream implementation must not resolve a `TODO:` by inference.
