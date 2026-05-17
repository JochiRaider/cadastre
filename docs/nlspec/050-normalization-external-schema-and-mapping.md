---
doc_id: CADASTRE-NLSPEC-050
title: Normalization, External Schema, and Mapping
doc_type: authoritative-nlspec
status: migration_active
generated_on: 2026-05-17
source_prd: PRD-Cadastre.md
source_prd_sha256: b17ac5d44618c43a57efe8ebd9b6c6e0bd8debc949b513368383c515c07f9748
---

## Authority

This document is a generated Cadastre NLSpec candidate. It is `migration_active` until the migration ledger marks its source rows complete and `docs/nlspec/120-validation-fixtures-and-acceptance.md` records passing acceptance evidence.

This document owns the contracts listed in `Exports`. Other active Cadastre NLSpecs may import those contracts by exact name and must not restate them.

## Purpose

Define how parsed raw records become silver observations and how external schemas, OCSF, CIM, source schemas, semantic overlays, and mapping validation work.

## Explicit Non-Scope

- Gold fact authority.
- Identity merge decisions.
- Graph projection or graph apply behavior.
- Package trust beyond package-stage activation.

## Imports

- `RawRecord`
- `CadastreSilverObservation`
- `SourceExtensionFieldRuleShape`
- `PackageStageBinding`
- `ValidationMatrix`

## Exports

- `ParseRawBatch`
- `NormalizeObservation`
- `ValidateMappingBundle`
- `ExternalSchemaProfile`
- `ExternalSchemaArtifactRef`
- `ProfileResolutionManifest`
- `ExternalEnumMappingRule`
- `OCSFBaseEventFieldPolicy`
- `OCSFProfileUpgradeReport`
- `CIMProjectionProfile`
- `ProjectionLossManifest`
- `SourceSchemaImportProfile`
- `SemanticOverlayArtifact`
- `MappingProjectManifest`
- `MappingCompilerPipeline`
- `CanonicalValidationOutput`
- `MappingValidationRule`
- `ObservationTypeExternalMappingValidationMatrix`

## Parser Contract

`ParseRawBatch(raw_records, parser_profile) -> ParseResult[]` must preserve raw evidence refs and must not assert canonical truth, source absence, identity, gold facts, graph deltas, CIM outputs, health state, watermark state, or package state.

Parser output must classify each raw record as exactly one of:

| Variant | Required behavior |
| --- | --- |
| `parsed` | Emits typed parse fields and parse checksum. |
| `unsupported` | Emits deterministic unsupported diagnostic. |
| `malformed` | Emits deterministic malformed diagnostic and preserves raw ref. |
| `duplicate` | Emits duplicate classification only when raw import declared duplicate behavior. |
| `quarantined` | Emits quarantine reason and raw ref. |

## Silver Normalization Contract

`NormalizeObservation(parse_result, mapping_bundle, external_schema_profile) -> CadastreSilverObservation | MappingDiagnostic` must emit a Cadastre silver envelope. `normalized_fields` must align to the active OCSF profile unless the observation type is declared `cadastre_only` by an active mapping row.

Cadastre-owned fields for omission, lineage, source identity, confidence, temporal semantics, identity inputs, and flow-role evidence must remain outside OCSF `normalized_fields`.

## External Schema Profile Contract

`ExternalSchemaProfile` must pin external schema name, version, source tag, source commit, compiler version, validator version, compiled artifact checksum, class allowlist, profile set, extension set, deprecated-field policy, and enum mapping behavior.

Production OCSF profile status must be `active` only after `ExternalSchemaArtifactRef`, `ProfileResolutionManifest`, `OCSFBaseEventFieldPolicy`, `ExternalEnumMappingRule`, `OCSFProfileUpgradeReport` when applicable, and `ObservationTypeExternalMappingValidationMatrix` pass.

## OCSF Mapping Requirements

| Mapping element | Required behavior |
| --- | --- |
| Category/class/activity/type | Every MVP observation type must have an exact row or a `TODO:` blocker. |
| Enum ID/name sibling | Must use `ExternalEnumMappingRule`; unknown values must not invent enum IDs. |
| `Other` enum values | Allowed only when compiled OCSF enum contains `Other` and active rule permits it. |
| `raw_data` and `unmapped` | Disabled by default unless `OCSFBaseEventFieldPolicy` allows a bounded use. |
| Observables and enrichments | Non-authoritative by default. |
| Deprecated fields | Rejected by default unless a non-expired waiver is recorded. |

## Source Extension Fields

Every `source_extension_fields` path must have an active `SourceExtensionFieldRule` before a production mapping bundle may emit it. The rule must define namespace, field path, type, bounds, redaction, collision policy, secret scan behavior, and OCSF-reserved-name collision behavior.

## Mapping Compiler Pipeline

```text
ValidateMappingBundle(bundle, project_manifest, compiler_pipeline):
1. Canonicalize source roots and dependency lock refs.
2. Reject undeclared source roots, mutable dependency refs, and user-local config.
3. Resolve source schema import profiles and semantic overlays.
4. Compile mappings in declared phase order.
5. Validate external schema profile refs and compiled artifact checksums.
6. Run MappingValidationRule rows with deterministic severities.
7. Run observation-type validation matrix cases.
8. Emit CanonicalValidationOutput with deterministic diagnostics and checksum.
```

## CIM Projection Contract

CIM output is a lossy, deterministic projection. `CIMProjectionProfile` must define input record classes, field mapping rows, lossy fields, unsupported fields, redaction, default values, and `ProjectionLossManifest` output. CIM success must not imply no source information was lost.

## Definition of Done

| ID | Criterion |
| --- | --- |
| `050-AC-001` | Every production mapping bundle emits deterministic `CanonicalValidationOutput`. |
| `050-AC-002` | OCSF-aligned observations validate against the exact compiled external schema artifact. |
| `050-AC-003` | Undeclared source extension fields fail before production output. |
| `050-AC-004` | CIM projection records every lossy or unsupported mapping and never becomes authoritative input. |
| `050-AC-005` | Every external mapping table is exhaustive for its declared MVP observation scope or carries a blocking `TODO:` row. |

## Source Traceability

| Source | Section or artifact | Location |
| --- | --- | --- |
| PRD-Cadastre.md | `CadastreSilverObservation` | lines 1806-1886 |
| PRD-Cadastre.md | `SourceExtensionFieldRule` | lines 1887-1952 |
| PRD-Cadastre.md | `ExternalSchemaProfile` | lines 3907-4007 |
| PRD-Cadastre.md | `SemanticOverlayArtifact` | lines 4008-4071 |
| PRD-Cadastre.md | `SourceSchemaImportProfile` | lines 4072-4141 |
| PRD-Cadastre.md | `CIMProjectionProfile` | lines 4142-4164 |
| PRD-Cadastre.md | `ExternalSchemaArtifactRef` | lines 5955-5983 |
| PRD-Cadastre.md | `ProfileResolutionManifest` | lines 5984-6016 |
| PRD-Cadastre.md | `ExternalEnumMappingRule` | lines 6065-6087 |
| PRD-Cadastre.md | `OCSFProfileUpgradeReport` | lines 6088-6120 |
| PRD-Cadastre.md | `OCSFBaseEventFieldPolicy` | lines 6121-6151 |
| PRD-Cadastre.md | `MappingValidationRule` | lines 6968-7024 |
| PRD-Cadastre.md | `MappingProjectManifest` | lines 8434-8469 |
| PRD-Cadastre.md | `ToolchainDependencyReview` | lines 8470-8502 |
| PRD-Cadastre.md | `ExternalToolCapabilityEvidence` | lines 8503-8534 |
| PRD-Cadastre.md | `ValidationScenario` | lines 8535-8613 |
| PRD-Cadastre.md | `CanonicalValidationOutput` | lines 8614-8665 |
| PRD-Cadastre.md | `Source Categories and Normalization` | lines 10647-11031 |
| PRD-Cadastre.md | `Parser Interface` | lines 9234-9257 |
| PRD-Cadastre.md | `Normalization Mapping Interface` | lines 9258-9396 |
| Decomposition plan | Current user prompt | Domain decomposition, disposition matrix, dependency model, gap ledger, and migration acceptance criteria. |

## Open Questions

Open questions marked `TODO:` block authoritative status for the affected contract. A downstream implementation must not resolve a `TODO:` by inference.
