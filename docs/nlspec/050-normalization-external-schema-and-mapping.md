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

- `CoreRecordValidationAlgorithm`
- `CadastreSilverObservationSchema`
- `ActivationControlledArtifactRef`

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

`NormalizeObservation(parse_result, mapping_bundle, external_schema_profile) -> CadastreSilverObservation | MappingDiagnostic` must emit a Cadastre silver envelope that passes `040.CadastreSilverObservationSchema` and `040.ValidateCoreRecord` before persistence. `normalized_fields` must align to the active OCSF profile unless the observation type is declared `cadastre_only` by an active mapping row.

`normalized_payload_checksum` must be computed from canonical OCSF class metadata plus `normalized_fields` canonical bytes. `external_schema_profile_id` may be null only when `observation_type` is declared `cadastre_only`. `source_extension_fields` defaults to `{}` and every path must have an active `SourceExtensionFieldRule`. `field_quality` must contain rows for any field with omission, redaction, parser quality, or source-quality state.

Cadastre-owned fields for omission, lineage, source identity, confidence, temporal semantics, identity inputs, and flow-role evidence must remain outside OCSF `normalized_fields`. OCSF fields must not override `040` omission states, `070` identity evidence, `080` temporal resolution, or `090` graph direction.

## External Schema Profile Contract

`ExternalSchemaProfile` must pin external schema name, version, source tag, source commit, compiler version, validator version, compiled artifact checksum, class allowlist, profile set, extension set, deprecated-field policy, and enum mapping behavior.

MVP OCSF baseline is `OCSF 1.8.0` as a candidate production baseline. The profile must remain non-active until `ExternalSchemaArtifactRef` records the exact source tag, source commit, compiler ID/version/checksum, validator ID/version/checksum, compiled artifact checksum, profile set, extension set, and class allowlist checksum. Use of OCSF `main`, development versions, or main-only fields is forbidden for production output.

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

Undeclared `source_extension_fields` must fail before production output with the owner-specific mapping error and the `040` schema extension-map validation result.

## Mapping Compiler Pipeline

```text
ValidateMappingBundle(bundle, project_manifest, compiler_pipeline):
1. Validate every mapping-related artifact ref through `030.ActivationControlledArtifactRef`.
2. Verify package-set inclusion when artifacts are package-supplied.
3. Canonicalize source roots and dependency lock refs.
4. Reject undeclared source roots, mutable dependency refs, and user-local config.
5. Resolve source schema import profiles and semantic overlays.
6. Compile mappings in declared phase order.
7. Validate external schema profile refs and compiled artifact checksums.
8. Reject any artifact that attempts to define a core field, core omission state, identity rule, temporal rule, graph rule, or source-authority rule.
9. Run MappingValidationRule rows with deterministic severities.
10. Run observation-type validation matrix cases.
11. Emit CanonicalValidationOutput with deterministic diagnostics and checksum.
12. Include all output-affecting mapping artifact refs in `VersionManifest` before silver output.
```

## CIM Projection Contract

CIM output is a lossy, deterministic projection. `CIMProjectionProfile` must define input record classes, field mapping rows, lossy fields, unsupported fields, redaction, default values, and `ProjectionLossManifest` output. CIM success must not imply no source information was lost.

## Mapping Activation Contract Details

### CadastreSilverObservation envelope field ownership matrix

| Envelope field | Field owner | Runtime owner |
| --- | --- | --- |
| `normalized_fields` | `040` shape, `050` OCSF validation | `050` |
| `source_extension_fields` | `040` shape, `050` activation | `050` |
| `observed_at` and `observed_time_quality` | `040` shape | `080` resolves fact time |
| `identity_inputs` | `040` shape | `070` turns inputs into identity evidence |
| `flow_role_evidence` | `040` shape | `090` consumes for graph direction when allowed |
| `redaction_summary` | `040` shape | `110` exposes or redacts |

OCSF `raw_data`, `unmapped`, `observables`, `enrichments`, `status`, `severity`, and `confidence` remain non-authoritative unless an active policy grants a bounded use.

### Mapping artifact volatility boundary

`050` stable text defines interfaces, validation requirements, compiler phase ordering, error behavior, and activation boundaries. Concrete parser profiles, mapping rows, external schema artifacts, enum rows, profile-resolution rows, source-extension rows, and compiler configuration are activation-controlled artifacts.

| Artifact or interface | Volatility class | Required handling |
| --- | --- | --- |
| `ParseRawBatch` interface | `stable_core_contract` | Must preserve evidence and emit no authoritative truth. |
| parser profile instance | `activation_controlled_artifact` | Must validate through `030.ActivationControlledArtifactRef`. |
| `NormalizeObservation` interface | `stable_core_contract` | Must emit only `040.CadastreSilverObservation` or diagnostics. |
| mapping bundle instance | `activation_controlled_artifact` | Must be active, scoped, checksummed, and manifest-recorded. |
| `ExternalSchemaProfile` | stable schema/interface plus activation-controlled instances | Interface is stable; concrete profile rows are activation-controlled. |
| `ExternalSchemaArtifactRef` | `activation_controlled_artifact` | Must carry exact artifact identity and checksum. |
| `ProfileResolutionManifest` | `activation_controlled_artifact` | Must be active for profile inheritance/resolution effects. |
| `ExternalEnumMappingRule` rows | `activation_controlled_artifact` | Must not invent enum IDs or restate OCSF behavior. |
| `OCSFBaseEventFieldPolicy` rows | `activation_controlled_artifact` | Must not grant Cadastre authority without owner contract. |
| `SourceExtensionFieldRule` rows | `activation_controlled_artifact` | Must declare extension paths, types, bounds, and redaction behavior. |
| `CIMProjectionProfile` rows | `activation_controlled_artifact` | Must remain projection-only. |
| `MappingProjectManifest` and concrete compiler config | `activation_controlled_artifact` | Must be included in validation and replay refs. |
| `MappingCompilerPipeline` phase-order contract | `stable_core_contract` | Concrete compiler config remains activation-controlled. |
| `CanonicalValidationOutput` | `runtime_state_record` | Records deterministic validation state and checksum. |

Concrete rows in `### Observation-to-OCSF mapping matrix` are activation-controlled row examples or blocking `TODO:` rows. They are not production-active rows unless represented by an active artifact ref.

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

| Cadastre observation type | OCSF category | OCSF class | Activity ID/name | Type UID/name | Required object paths | Enum rules | Disabled base-event fields | Validation fixture IDs | Blocked outputs while TODO | Status |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `inventory_observation` | TODO | TODO | TODO | TODO | TODO | `ExternalEnumMappingRule` | `raw_data`, `unmapped` by default | TODO | active normalized output, OCSF export, downstream gold candidates | blocking |
| `software_inventory_observation` | TODO | TODO | TODO | TODO | TODO | `ExternalEnumMappingRule` | `raw_data`, `unmapped` by default | TODO | active normalized output, OCSF export, downstream gold candidates | blocking |
| `vulnerability_finding_observation` | TODO | TODO | TODO | TODO | TODO | `ExternalEnumMappingRule` | `raw_data`, `unmapped` by default | TODO | active normalized output, OCSF export, vulnerability gold candidates | blocking |
| `authentication_observation` | TODO | TODO | TODO | TODO | TODO | `ExternalEnumMappingRule` | `raw_data`, `unmapped` by default | TODO | active normalized output, OCSF export, identity/gold candidates | blocking |
| `dns_observation` | TODO | TODO | TODO | TODO | TODO | `ExternalEnumMappingRule` | `raw_data`, `unmapped` by default | TODO | active normalized output, OCSF export, DNS gold candidates | blocking |
| `dhcp_ipam_observation` | TODO | TODO | TODO | TODO | TODO | `ExternalEnumMappingRule` | `raw_data`, `unmapped` by default | TODO | active normalized output, OCSF export, DHCP/IPAM gold candidates | blocking |
| `network_activity_observation` | TODO | TODO | TODO | TODO | TODO | `ExternalEnumMappingRule` | `raw_data`, `unmapped` by default | TODO | active normalized output, OCSF export, observed-connection graph candidates | blocking |
| `cadastre_only` | none | none | none | none | Cadastre envelope only | n/a | n/a | TODO | none when mapping row explicitly allows null profile | allowed only by mapping row |

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
| `source.<source_category>` | `source.<namespace>.<field_name>` where each segment is lowercase snake case and non-empty | scalar: string, boolean, uint64, int64, decimal string, timestamp, sha256; container: array or map of allowed scalar | string max 4096 scalar values; arrays max 1024 elements; maps max 4096 entries; canonical object max 65536 bytes | owner row must state caller-visible, audit-only, or redacted | reject path collision and type collision by default | reject secret-like value before output unless redaction policy permits audit-only storage | reject any path equal to or prefixed by an OCSF reserved base-event field unless policy permits non-authoritative metadata | `050` | `fixture-050-source-extension-declared` |
| undeclared namespace or path | n/a | n/a | n/a | n/a | n/a | n/a | n/a | `050` | `fixture-050-source-extension-undeclared` emits `SOURCE_EXTENSION_FIELD_UNDECLARED` |

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

### Mapping diagnostics and errors

| Error code | Emitted when |
| --- | --- |
| `MAPPING_ARTIFACT_INACTIVE` | A required mapping, parser, schema, enum, profile, extension, CIM, or compiler artifact is not active for the execution mode. |
| `MAPPING_ARTIFACT_SCOPE_MISMATCH` | A required mapping artifact does not cover the execution scope. |
| `MAPPING_ARTIFACT_CHECKSUM_MISMATCH` | A required mapping artifact checksum does not match the active ref or manifest. |
| `MAPPING_CORE_CONTRACT_OVERRIDE_FORBIDDEN` | A mapping artifact attempts to redefine core fields, omission states, identity semantics, temporal semantics, graph semantics, or source-authority semantics. |
| `OCSF_ARTIFACT_MISMATCH` | Active profile refs do not match the compiled artifact checksum, source tag, source commit, compiler, or validator evidence. |
| `OCSF_CLASS_NOT_ALLOWED` | A mapping emits an OCSF class outside the active class allowlist. |
| `EXTERNAL_ENUM_UNKNOWN` | Source enum value is not mapped and no unknown-value policy applies. |
| `EXTERNAL_ENUM_OTHER_NOT_PERMITTED` | Mapping attempts OCSF `Other` without an active rule preserving the raw value. |
| `EXTERNAL_ENUM_DEPRECATED` | Mapping emits a deprecated enum or field without a non-expired waiver. |
| `EXTERNAL_ENUM_SIBLING_MISMATCH` | Enum ID/name sibling values do not match the compiled artifact. |
| `SOURCE_EXTENSION_FIELD_UNDECLARED` | Mapping emits a source extension field without an active `SourceExtensionFieldRule`. |

### OCSFProfileUpgradeReport evidence requirements

| Evidence | Required behavior |
| --- | --- |
| schema diff | Compare current and candidate compiled artifacts and record checksum. |
| golden corpus replay | Replay every active observation mapping fixture and compare expected output checksums. |
| shadow run | Produce non-production output only and record drift summary. |
| enum change review | Review new, removed, renamed, deprecated, and sibling enum changes. |
| deprecated field review | List deprecated fields and waiver decisions. |
| class allowlist change | Record added/removed classes and affected observation types. |
| extension set change | Record added/removed extensions, namespace collisions, and reserved-name effects. |

### Acceptance Criteria

| ID | Criterion |
| --- | --- |
| `050-CLEANUP-AC-001` | No banned reference class remains. |
| `050-CLEANUP-AC-002` | `normalized_fields` remains governed by active OCSF profile unless an observation type is declared `cadastre_only`. |
| `050-CLEANUP-AC-003` | OCSF raw, unmapped, observable, enrichment, status, severity, and confidence fields remain non-authoritative unless explicitly governed by policy. |
| `050-SCHEMA-PATCH-AC-001` | Every normalized output validates against `040.CadastreSilverObservationSchema`. |
| `050-SCHEMA-PATCH-AC-002` | Undeclared `source_extension_fields` fail before production output. |
| `050-SCHEMA-PATCH-AC-003` | `cadastre_only` observations are the only silver observations allowed to have null `external_schema_profile_id`. |
| `050-SCHEMA-PATCH-AC-004` | OCSF fields cannot create identity, gold, graph, authority, or omission semantics outside the owning specs. |
| `050-CLEANUP-AC-004` | Mapping validation remains deterministic and byte-stable through `CanonicalValidationOutput`. |

| `050-OCSF-BASELINE-AC-001` | MVP OCSF production activation is blocked until exact `OCSF 1.8.0` artifact identity, compiled artifact checksum, class allowlist, profile set, extension set, enum rules, and validation fixtures are recorded. |
| `050-SOURCE-EXTENSION-AC-001` | Undeclared, namespace-invalid, unbounded, secret-scan-failing, or OCSF-reserved-colliding source extension fields fail before production output. |
| `050-VOLATILITY-AC-001` | Every production mapping-related artifact validates through `030.ActivationControlledArtifactRef` before silver output. |
| `050-VOLATILITY-AC-002` | OCSF artifact checksum mismatch, inactive enum rule sets, omitted source-extension row refs, and mapping core overrides fail before production output. |
| `050-VOLATILITY-AC-003` | `cadastre_only` mapping with null schema profile is allowed only by an active mapping row. |

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
