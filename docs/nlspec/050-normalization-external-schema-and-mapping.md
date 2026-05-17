---
doc_id: CADASTRE-NLSPEC-050
title: Normalization, External Schema, and Mapping
doc_type: nlspec
status: candidate
---

## Authority

This document owns the contracts listed in `Exports`. Other Cadastre NLSpecs may import those contracts by exact name and must not restate them. This document has implementation authority only after the document registry marks it `authoritative` and its acceptance criteria pass.

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

## Mapping Activation Contract Details

### ExternalSchemaArtifactRef field table

| Field | Required | Rule |
| --- | ---: | --- |
| `schema_name` | Yes | External schema family, default `OCSF` for MVP mapped observations. |
| `version` | Yes | Exact external schema version. |
| `source_tag` | Yes | Immutable tag or release identifier. |
| `source_commit` | Yes | Exact source commit when available. |
| `compiler_id`, `compiler_version`, `compiler_checksum` | Yes | Required before production validation. |
| `validator_id`, `validator_version`, `validator_checksum` | Yes | Required before production validation. |
| `compiled_artifact_checksum` | Yes | SHA-256 over compiled artifact bytes. |
| `profile_set` | Yes | Declared profiles only. |
| `extension_set` | Yes | Declared extensions only; empty set is explicit. |
| `class_allowlist` | Yes | Every active observation mapping must use an allowed class. |
| `expiration` | Yes | Non-expired for production activation. |

### Observation-to-OCSF mapping matrix

The following rows define the active MVP mapping scope at MVP mapping scope. Missing exact class, activity, type, object path, or fixture IDs are blocking rows, not implementation discretion.

| Cadastre observation type | OCSF category | OCSF class | Activity ID/name | Type UID/name | Required object paths | Enum rules | Disabled base-event fields | Validation fixture IDs | Status |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `inventory_observation` | TODO | TODO | TODO | TODO | TODO | `ExternalEnumMappingRule` | `raw_data`, `unmapped` by default | TODO | blocking |
| `software_inventory_observation` | TODO | TODO | TODO | TODO | TODO | `ExternalEnumMappingRule` | `raw_data`, `unmapped` by default | TODO | blocking |
| `vulnerability_finding_observation` | TODO | TODO | TODO | TODO | TODO | `ExternalEnumMappingRule` | `raw_data`, `unmapped` by default | TODO | blocking |
| `authentication_observation` | TODO | TODO | TODO | TODO | TODO | `ExternalEnumMappingRule` | `raw_data`, `unmapped` by default | TODO | blocking |
| `dns_observation` | TODO | TODO | TODO | TODO | TODO | `ExternalEnumMappingRule` | `raw_data`, `unmapped` by default | TODO | blocking |
| `dhcp_ipam_observation` | TODO | TODO | TODO | TODO | TODO | `ExternalEnumMappingRule` | `raw_data`, `unmapped` by default | TODO | blocking |
| `network_activity_observation` | TODO | TODO | TODO | TODO | TODO | `ExternalEnumMappingRule` | `raw_data`, `unmapped` by default | TODO | blocking |
| `cadastre_only` | none | none | none | none | Cadastre envelope only | n/a | n/a | TODO | allowed only by mapping row |

### OCSFBaseEventFieldPolicy

| Base-event field | Default | Required behavior |
| --- | --- | --- |
| `raw_data` | disabled | Raw payload remains in `RawRecord`; enabling requires explicit bounded policy and redaction rows. |
| `raw_data_hash` | disabled | Payload hash authority remains Cadastre-owned; mapping may preserve as non-authoritative metadata only. |
| `unmapped` | disabled | Unknown fields must use declared source extension fields or fail. |
| `observables` | non-authoritative | Extraction hints only; no graph node or identity authority. |
| `enrichments` | non-authoritative | Context only; `130` governs enrichment. |
| `severity`, `status`, `confidence` | non-authoritative | Must not become Cadastre assertion, compliance, or risk authority by default. |
| `metadata`, `message`, timezone fields | allowed as normalized metadata only | Must not override Cadastre timestamps or evidence refs. |

### ExternalEnumMappingRule

| Case | Required behavior | Validation failure code |
| --- | --- | --- |
| Enum ID/name sibling present and known | Emit both values exactly as compiled artifact permits. | none |
| Raw value unknown | Preserve raw value in declared extension or diagnostic; do not invent ID. | `EXTERNAL_ENUM_UNKNOWN` when production mapping lacks policy. |
| `Other` exists | Use only when active rule permits and raw value is preserved. | `EXTERNAL_ENUM_OTHER_NOT_PERMITTED` |
| Deprecated enum | Reject unless non-expired waiver exists. | `EXTERNAL_ENUM_DEPRECATED` |
| Name/ID mismatch | Reject. | `EXTERNAL_ENUM_SIBLING_MISMATCH` |

### Source extension field matrix

| Namespace | Field path | Type | Bounds | Redaction | Collision policy | Secret-scan behavior | OCSF-reserved-name behavior | Owner | Validation fixture |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| TODO | TODO | TODO | TODO | TODO | reject collision by default | reject secret-like value by default | reject by default | `050` | TODO |

### MappingValidationRule severity defaults

| Rule class | Default severity | Production demotion constraint |
| --- | --- | --- |
| undeclared source extension | `error` | Blocks active and canary. |
| unknown OCSF enum | `error` | Blocks active. |
| deprecated OCSF field | `error` | Blocks unless waiver active. |
| OCSF artifact mismatch | `error` | Blocks all production output. |
| CIM projection loss without manifest | `error` | Blocks export. |
| documentation-only schema evidence | `error` | Blocks production mapping. |

External schema docs, OCSF `main` branch, dev fields, and uncompiled artifacts cannot authorize production mappings.

### Acceptance Criteria

| ID | Criterion |
| --- | --- |
| `050-CLEANUP-AC-001` | No banned reference class remains. |
| `050-CLEANUP-AC-002` | `normalized_fields` remains governed by active OCSF profile unless an observation type is declared `cadastre_only`. |
| `050-CLEANUP-AC-003` | OCSF raw, unmapped, observable, enrichment, status, severity, and confidence fields remain non-authoritative unless explicitly governed by policy. |
| `050-CLEANUP-AC-004` | Mapping validation remains deterministic and byte-stable through `CanonicalValidationOutput`. |

## Definition of Done

| ID | Criterion |
| --- | --- |
| `050-AC-001` | Every production mapping bundle emits deterministic `CanonicalValidationOutput`. |
| `050-AC-002` | OCSF-aligned observations validate against the exact compiled external schema artifact. |
| `050-AC-003` | Undeclared source extension fields fail before production output. |
| `050-AC-004` | CIM projection records every lossy or unsupported mapping and never becomes authoritative input. |
| `050-AC-005` | Every external mapping table is exhaustive for its declared MVP observation scope or carries a blocking `TODO:` row. |


## Open Questions

Open questions marked `TODO:` block authoritative status for the affected contract. A downstream implementation must not resolve a `TODO:` by inference.
