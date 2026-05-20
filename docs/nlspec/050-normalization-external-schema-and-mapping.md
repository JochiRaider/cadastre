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
- `ObservationToOCSFMappingRow`
- `ObservationToOCSFMappingRowSet`
- `ResolveOCSFMapping`
- `SourceExtensionFieldRule`
- `SourceExtensionFieldRuleSet`
- `MappingArtifactLifecycleGuardRows`

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

`NormalizeObservation(parse_result, mapping_bundle, external_schema_profile) -> CadastreSilverObservation | MappingDiagnostic` must emit a Cadastre silver envelope that passes `040.CadastreSilverObservationSchema` and `040.ValidateCoreRecord` before persistence. `NormalizeObservation` must call `ResolveOCSFMapping` before emitting `normalized_fields`. `normalized_fields` must align to the active OCSF profile unless the observation type is declared `cadastre_only` by exactly one active `ObservationToOCSFMappingRow`.

`normalized_payload_checksum` must be computed from canonical OCSF class metadata plus `normalized_fields` canonical bytes. `external_schema_profile_id` may be null only when `observation_type` is declared `cadastre_only` by exactly one active row. `source_extension_fields` defaults to `{}`. Every non-empty source-extension path must match exactly one active `SourceExtensionFieldRule` in the selected `SourceExtensionFieldRuleSet`. `field_quality` must contain rows for any field with omission, redaction, parser quality, or source-quality state.

Cadastre-owned fields for omission, lineage, source identity, confidence, temporal semantics, identity inputs, and flow-role evidence must remain outside OCSF `normalized_fields`. OCSF fields must not override `040` omission states, `070` identity evidence, `080` temporal resolution, or `090` graph direction.

## External Schema Profile Contract

`ExternalSchemaProfile` must pin external schema name, version, source tag, source commit, compiler version, validator version, compiled artifact checksum, class allowlist, profile set, extension set, deprecated-field policy, and enum mapping behavior.

MVP OCSF baseline is `OCSF 1.8.0` as a candidate production baseline. The profile must remain non-active until `ExternalSchemaArtifactRef` records the exact source tag, source commit, compiler ID/version/checksum, validator ID/version/checksum, compiled artifact checksum, profile set, extension set, and class allowlist checksum. Use of OCSF `main`, development versions, or main-only fields is forbidden for production output.

Production OCSF profile status must be `active` only after `ExternalSchemaArtifactRef`, `ProfileResolutionManifest`, `OCSFBaseEventFieldPolicy`, `ExternalEnumMappingRule`, `OCSFProfileUpgradeReport` when applicable, and `ObservationTypeExternalMappingValidationMatrix` pass.

## OCSF Mapping Requirements

| Mapping element | Required behavior |
| --- | --- |
| Category/class/activity/type | Every MVP observation type must resolve to exactly one active `ObservationToOCSFMappingRow` or fail before normalized output. |
| Enum ID/name sibling | Must use `ExternalEnumMappingRule`; unknown values must not invent enum IDs. |
| `Other` enum values | Allowed only when compiled OCSF enum contains `Other` and active rule permits it. |
| `raw_data` and `unmapped` | Disabled by default unless `OCSFBaseEventFieldPolicy` allows a bounded use. |
| Observables and enrichments | Non-authoritative by default. |
| Deprecated fields | Rejected by default unless a non-expired waiver is recorded. |

### ExternalSchemaSignalAuthorityHandoff

`ObservationToOCSFMappingRow` must not grant source authority, coverage, staleness, control-result, absence, cleanup, retraction, graph expiry, or watermark permission.

If a mapping bundle intends an external schema field to be considered by authority logic, it must emit the normalized field as observation metadata only. A separate active `060.ExternalSchemaAuthoritySignalMappingRow` must name the external schema profile ref, field path, source dataset, fact type, predicate, requested effect, authority row ref, coverage row ref when applicable, staleness row ref, validation refs, activation scope, and lifecycle status.

`060.ExternalSchemaAuthoritySignalMappingRow` must also reference the selected `SourceAuthorityClosureMatrixRow` or deterministic block row, `SourceAuthorityProfileRow`, `CoverageDimensionProfile` when coverage-sensitive, `SourceStalenessPolicy`, `ControlResultMappingRow` when control-state output is requested, `AbsenceDerivationPolicy` when absence-sensitive output is requested, and `ProjectionWatermarkPolicy` when watermark is requested.

Missing `060.ExternalSchemaAuthoritySignalMappingRow` means the external schema signal remains non-authoritative. A mapping row or diagnostic that attempts to set authority, absence, cleanup, retraction, graph expiry, control pass/fail, or watermark effects must route the failure to imported `060.EXTERNAL_SCHEMA_AUTHORITY_FORBIDDEN` before the attempted authority effect. The normalized fields may remain persisted as observation metadata when the mapping itself is valid.

### GoldFactObjectBoundaryHandoff

`NormalizeObservation` and `ResolveOCSFMapping` must not emit `GoldFact` bytes, `GoldFact.subject_ref`, `GoldFact.object_value`, `EvidenceRef.artifact_id`, graph deltas, source authority decisions, absence outcomes, correction outputs, or watermark effects. Mapping output may supply observation metadata only. Gold object values may be derived only by `080.DeriveFacts` after `040` one-of validation, `060` authority validation, and `080` predicate-contract validation.

OCSF objects may become `structured_value` only through an active `080.GoldFactPredicateContractRow.structured_value_schema_refs` entry and a matching `060` authority row when authority is required. OCSF `raw_data`, `unmapped`, observables, enrichments, status, severity, and confidence remain non-authoritative unless exact `050`, `060`, and `080` handoff rows permit a bounded interpretation.

`source_extension_fields` are never subject/object authority by themselves. A mapping bundle that attempts to set `object_kind`, `object_value.kind`, `GoldFact.subject_ref`, `EvidenceRef.artifact_id`, `absence_outcome`, source authority, correction output, graph output, or watermark output must fail before silver output or emit a mapping diagnostic using the most specific owner error.

### FlowRoleEvidenceMappingHandoff

`FlowRoleEvidence` is Cadastre-owned evidence outside OCSF `normalized_fields`. Mapping bundles may preserve OCSF endpoint fields as normalized observation metadata, but they must not synthesize `FlowRoleEvidence` from endpoint field order.

| Source direction condition | Mapping behavior | Graph handoff effect |
| --- | --- | --- |
| Explicit source payload direction evidence accepted by a `050` mapping rule | Emit a `FlowRoleEvidence` item with source path refs, confidence, ambiguity state, NAT/proxy/aggregation flags when present, and field-quality metadata. | `090` may consume the item only if it qualifies under the active graph projection row. |
| OCSF `src_endpoint` and `dst_endpoint` present without accepted explicit flow-role evidence | Preserve endpoints in `normalized_fields` when the OCSF mapping row permits them; emit no `FlowRoleEvidence` item. | `090` must emit no `observed_connection` direction from endpoint order. |
| Ambiguous endpoint role, NAT uncertainty, proxy uncertainty, sensor aggregation, or collection-point uncertainty | Emit no qualifying `FlowRoleEvidence`; record field-quality or diagnostic metadata. | `090` must no-op or emit `GRAPH_FLOW_ROLE_EVIDENCE_REQUIRED` when the projection attempt is observable. |
| DNS or DHCP observation with endpoint-like fields | Emit no `FlowRoleEvidence` by default. | No `observed_connection` edge unless a later active graph projection row and qualifying flow-role evidence explicitly permit it. |
| `cadastre_only` network-context row | May emit Cadastre-only context metadata when the row permits it. | No graph direction unless `090` consumes qualifying flow-role evidence. |

Absent or ambiguous source direction emits no qualifying `FlowRoleEvidence` item. It must not be converted into inferred graph direction, inferred reachability, service access, or source authority.

### ObservationToOCSFMappingRow schema

`ObservationToOCSFMappingRow` is the stable row interface for mapping one Cadastre observation subtype and discriminator state to one external schema output class. Concrete row instances are activation-controlled artifacts. A row instance may affect production output only through `030.ActivationControlledArtifactRef` with `artifact_class = observation_to_ocsf_mapping_row_set`.

| Field | Required | Default or omission behavior | Rule |
| --- | ---: | --- | --- |
| `row_id` | Yes | none | Stable unique ID within the row set. |
| `observation_type` | Yes | none | Must match `040.CadastreSilverObservationSchema.observation_type`. |
| `mapping_discriminator` | Yes | none | Canonical predicate over normalized parse fields, source metadata, and declared mapping-bundle inputs. It must not inspect graph, gold, identity, or API state. |
| `external_schema_profile_ref` | Required unless `cadastre_only` | null only when the row explicitly sets `cadastre_only = true` | Must reference an active `ExternalSchemaProfile`. |
| `ocsf_category_name` | Required for OCSF rows | null for `cadastre_only` | Must match the compiled artifact. |
| `ocsf_category_uid` | Required for OCSF rows | null for `cadastre_only` | Must match the compiled artifact. |
| `ocsf_class_name` | Required for OCSF rows | null for `cadastre_only` | Must match the compiled artifact and active class allowlist. |
| `ocsf_class_uid` | Required for OCSF rows | null for `cadastre_only` | Must match the compiled artifact and active class allowlist. |
| `ocsf_activity_id` | Required for OCSF rows | null for `cadastre_only` | Must match the compiled class activity enum. |
| `ocsf_activity_name` | Required for OCSF rows | null for `cadastre_only` | Must be the compiled name for `ocsf_activity_id`. |
| `ocsf_type_uid` | Required for OCSF rows | null for `cadastre_only` | Must match compiled type UID rules for class and activity. |
| `ocsf_type_name` | Required for OCSF rows | null for `cadastre_only` | Must be the compiled name for `ocsf_type_uid`. |
| `required_normalized_paths` | Yes | `[]` only for `cadastre_only` | Every listed path must be present after mapping unless field quality marks a permitted OCSF omission state. |
| `forbidden_normalized_paths` | No | `[]` | Emitting any listed path fails before output. |
| `enum_rule_refs` | Yes | `[]` only when no enum mapping affects output | Refs to active `ExternalEnumMappingRuleSet` rows. |
| `base_event_field_policy_ref` | Yes | none | Ref to an active `OCSFBaseEventFieldPolicySet`. |
| `source_extension_rule_set_ref` | Yes | empty rule set when omitted by explicit row policy | Ref to an active `SourceExtensionFieldRuleSet`; omitted means no source-extension fields may be emitted. |
| `cadastre_owned_field_policy_refs` | Yes | `[]` | Refs to owner policies for Cadastre-owned envelope fields used by the row. |
| `validation_fixture_refs` | Yes | none | Non-empty refs to positive and negative fixtures before activation. |
| `activation_scope` | Yes | none | Scope in which the row may affect output. |
| `lifecycle_status` | Yes | none | Imported from `030.LifecycleStatus`; production mapping requires `active`. |

Unknown fields in `ObservationToOCSFMappingRow` fail before row-set activation. Row-set canonical bytes sort rows by ascending lexical `row_id`.

### ResolveOCSFMapping algorithm

```text
ResolveOCSFMapping(observation, mapping_bundle, external_schema_profile):
1. Validate ExternalSchemaProfile, ExternalSchemaArtifactRef, ProfileResolutionManifest, ExternalEnumMappingRuleSet, OCSFBaseEventFieldPolicySet, SourceExtensionFieldRuleSet, and ObservationToOCSFMappingRowSet through 030.ActivationControlledArtifactRef.
2. Load active ObservationToOCSFMappingRow rows whose observation_type equals observation.observation_type.
3. Evaluate mapping_discriminator rows in ascending lexical row_id order.
4. If no row matches, emit MAP_OCSF_ROW_MISSING before normalized output.
5. If more than one row matches, emit MAP_OCSF_ROW_AMBIGUOUS before normalized output.
6. Validate category_uid, class_uid, activity_id, type_uid, and type_name against the compiled OCSF artifact.
7. Reject any class_uid outside the active class_allowlist with OCSF_CLASS_NOT_ALLOWED.
8. Validate required_normalized_paths and forbidden_normalized_paths.
9. Apply ExternalEnumMappingRule rows; unknown enum values must not invent enum IDs.
10. Validate every source_extension_fields path against exactly one active SourceExtensionFieldRule.
11. Emit CadastreSilverObservation only after 040.CadastreSilverObservationSchema passes.
```

Missing discriminator inputs fail before output. Authentication rows must emit `OCSF_ACTIVITY_DISCRIMINATOR_MISSING` when activity cannot be resolved to one compiled activity. Source-action or enum values not present in the compiled artifact must be preserved through declared diagnostics or declared source-extension fields; they must not create OCSF enum IDs.

### MVP OCSF class allowlist

The active MVP OCSF class allowlist must contain exactly the class UIDs used by active OCSF mapping rows. The default MVP class targets are closed to the following class UID set until a later profile upgrade activates additional classes.

| Class name | Class UID | Required mapping use |
| --- | ---: | --- |
| Device Inventory Info | `5001` | `inventory_observation` rows. |
| Software Inventory Info | `5020` | `software_inventory_observation` rows. |
| Vulnerability Finding | `2002` | `vulnerability_finding_observation` rows. |
| Authentication | `3002` | `authentication_observation` rows. |
| DNS Activity | `4003` | `dns_observation` rows. |
| DHCP Activity | `4004` | DHCP-discriminated `dhcp_ipam_observation` rows only. |
| Network Activity | `4001` | `network_activity_observation` rows. |

A `cadastre_only` row has no OCSF category, class, activity, or type and must not contribute to the class allowlist.

### MVPObservationMappingClosureChecklist

Concrete row instances are activation-controlled artifacts. This checklist defines the required closure content for each MVP observation family before an active row set can affect production silver output.

| Observation family | Required closure content |
| --- | --- |
| `inventory_observation` | Active row set ref, exact OCSF class `Device Inventory Info`, activity discriminator, compiled artifact ref, enum rule ref, base-event policy ref, object-path validation refs, and fixture refs. |
| `software_inventory_observation` | Active row set ref, exact class `Software Inventory Info`, `device` and `sbom` path coverage, and explicit forbidden deprecated `package` path fixture. |
| `vulnerability_finding_observation` | Active rows for create, update, and close source states; source scanner labels must not collapse to Cadastre absence without `060` authority. |
| `authentication_observation` | One active row per supported activity; missing discriminator must not default to an activity. |
| `dns_observation` | DNS Activity row coverage and explicit no graph direction handoff by default. |
| `dhcp_ipam_observation` | `assignment_source = dhcp` maps to DHCP Activity; `assignment_source = ipam` must be explicit `cadastre_only` or fail. |
| `network_activity_observation` | Network Activity rows; aggregate default is allowed only when the row explicitly declares aggregate-flow behavior; direction must route through `FlowRoleEvidence`. |
| `cadastre_only` | Exact active row required; null external profile is allowed only for that row. |

An observation type outside the active MVP row-family set must fail with `MAP_OCSF_ROW_MISSING` unless exactly one active `cadastre_only` row covers it. A missing mapping row must not default to `cadastre_only`.

`ObservationToOCSFMappingRowSet` activation requires non-empty `validation_refs`, fixture checksums, expected output checksums, lifecycle status `active`, and `030.VersionManifest` inclusion. Any `TODO` fixture checksum or expected checksum keeps the row set out of production scope.

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

`CIMProjectionProfile` is a lossy deterministic projection profile. CIM success must not imply that no source information was lost. Every CIM projection run must emit `ProjectionLossManifest` when fields are dropped, transformed, redacted, coalesced, or unsupported.

### ProjectionReplayEquivalenceHandoff

`050` owns projection-specific included and excluded fields for replay output class `080.ReplayEquivalencePolicy.output_class = export_projection`. `080` owns checksum algorithm and replay preflight ordering.

| Output subset | Included replay fields | Excluded volatile fields | Required validation |
| --- | --- | --- | --- |
| `cim_projection` | projection profile ref, input record refs, mapping row refs, redaction policy refs, output checksum, `ProjectionLossManifest` checksum, `VersionManifest` ref | export job ID, request correlation ID, execution duration, display timestamp | exact replay, profile mismatch, redaction mismatch, checksum mismatch |
| `ocsf_export_projection` | external schema profile ref, profile resolution manifest ref, mapping row refs, input record refs, redaction policy refs, output checksum, loss manifest checksum, `VersionManifest` ref | export job ID, request correlation ID, execution duration, display timestamp | exact replay, schema profile mismatch, loss-manifest mismatch |
| `projection_loss_manifest` | projection profile ref, dropped/transformed/redacted/coalesced/unsupported field summaries, input checksum, manifest checksum, `VersionManifest` ref | display timestamp, UI sort order | exact replay and loss-manifest mismatch |

A missing projection replay row blocks replay output with `REPLAY_POLICY_ARTIFACT_MISSING` unless a more specific owner projection replay code is registered in `110`. Projection exporters must persist output checksums and loss-manifest checksums before replay output is accepted.

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
| `ObservationToOCSFMappingRowSet` | `activation_controlled_artifact` | Must resolve each emitted mapped observation to exactly one active row. |
| `ExternalEnumMappingRuleSet` | `activation_controlled_artifact` | Must not invent enum IDs or restate OCSF behavior. |
| `OCSFBaseEventFieldPolicySet` | `activation_controlled_artifact` | Must not grant Cadastre authority without owner contract. |
| `SourceExtensionFieldRuleSet` | `activation_controlled_artifact` | Must declare extension paths, types, bounds, and redaction behavior. |
| `CIMProjectionProfile` rows | `activation_controlled_artifact` | Must remain projection-only. |
| `MappingProjectManifest` and concrete compiler config | `activation_controlled_artifact` | Must be included in validation and replay refs. |
| `MappingCompilerPipeline` phase-order contract | `stable_core_contract` | Concrete compiler config remains activation-controlled. |
| `CanonicalValidationOutput` | `runtime_state_record` | Records deterministic validation state and checksum. |

Concrete rows in `### Observation-to-OCSF mapping matrix` are activation-controlled row examples or blocking `TODO:` rows. They are not production-active rows unless represented by an active artifact ref.

### MappingArtifactLifecycleGuardRows

`050` activation-controlled artifacts use `030.ActivationControlledArtifactLifecycleMachine.v1`. `050` owns only the mapping-specific guard rows below.

| Artifact class | Required guard before `validated` | Required guard before `active` | Quarantine trigger |
| --- | --- | --- | --- |
| parser profile | Parser fixtures pass, malformed/unsupported/duplicate cases pass, and no authoritative output attempt occurs. | Package set active, parser stage binding active, and validation refs non-expired. | Parser emits forbidden output or raw evidence leak. |
| mapping bundle | `CanonicalValidationOutput` passes, no core override occurs, and no undeclared source roots exist. | `ObservationToOCSFMappingRowSet` resolves every emitted mapped observation, required row-set refs are manifest-included, and the mapping package is in the active package set. | Core field override, undeclared extension field, ambiguous mapping row, missing mapping row, or OCSF artifact mismatch. |
| external schema profile/artifact | Artifact checksum, compiler, validator, class allowlist, profile set, and extension set validate. | `OCSFBaseEventFieldPolicySet`, enum rule set, profile resolution, and observation-type fixtures pass. | Schema artifact checksum mismatch or dev/main schema used in production. |
| observation-to-OCSF mapping row set | Row schema, discriminator totality, class allowlist, object-path policy, enum refs, base-event policy refs, source-extension refs, fixture refs, and activation scope validate. | Every active row has positive and negative fixtures and appears in `VersionManifest`. | Missing row, ambiguous row, class not allowed, forbidden field emitted, or object path missing. |
| source-extension rule set | Namespace, type, bounds, redaction, collision, and secret-scan validation pass. | Rule refs are present in mapping bundle and manifest; omitted rule set means empty rule set. | Raw secret exposure, wildcard namespace, undeclared field, or reserved-name collision. |
| CIM projection profile | Projection-loss manifest fixtures pass. | Projection remains non-authoritative. | CIM output attempts to become raw, silver, gold, or identity authority. |

A mapping artifact entering `active` status must have `030.LifecycleTransitionEvidence` for `030.ActivationControlledArtifactLifecycleMachine.v1` and the relevant mapping-specific guard rows.

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

The matrix below defines required row families for active `ObservationToOCSFMappingRowSet` artifacts. It does not embed production-active volatile rows in stable core text. A row family is complete only when concrete active row instances provide exact row IDs, discriminator predicates, OCSF category/class/activity/type values, object-path policy, enum refs, base-event policy refs, source-extension refs, fixture refs, activation scope, lifecycle status, and manifest refs.

| Cadastre observation type | Required row family |
| --- | --- |
| `inventory_observation` | Device Inventory Info `5001`. Include `Log` and `Collect` rows. Default to `Collect` only for snapshot/export inventory feeds. Use `Log` only when a log-origin discriminator is present. |
| `software_inventory_observation` | Software Inventory Info `5020`. Include `Log` and `Collect` rows. Require `device` and `sbom`; forbid deprecated `package` unless a non-expired waiver exists. |
| `vulnerability_finding_observation` | Vulnerability Finding `2002`. Include create, update, and close lifecycle rows. Default to update for current-state scanner/export feeds. `Create` and `Close` require explicit source lifecycle evidence. |
| `authentication_observation` | Authentication `3002`. Include one row per compiled OCSF 1.8.0 authentication activity. No default activity is allowed. Missing or ambiguous authentication activity fails before output. |
| `dns_observation` | DNS Activity `4003`. Include Query, Response, and Traffic rows. Response rows require response-code handling and explicit quality behavior when answers are empty. |
| `dhcp_ipam_observation` | DHCP Activity `4004` only when `assignment_source = dhcp`. Include rows for the compiled DHCP activity enum. `assignment_source = ipam` must not map to DHCP Activity. It must either map to an explicit `cadastre_only` row or fail with `MAPPING_OBSERVATION_TYPE_SPLIT_REQUIRED`. |
| `network_activity_observation` | Network Activity `4001`. Include Open, Close, Reset, Fail, Refuse, Traffic, and Listen rows. Default to Traffic only for aggregate flow feeds. Graph direction remains governed by `FlowRoleEvidence`, not OCSF endpoint order. |
| `cadastre_only` | No OCSF category, class, activity, or type. Null `external_schema_profile_id` is allowed only by an active row. |

### OCSF object-path matrix

| Observation type | Required normalized paths |
| --- | --- |
| `inventory_observation` | OCSF base event fields plus `device`. |
| `software_inventory_observation` | OCSF base event fields plus `device` and `sbom`; deprecated `package` forbidden by default. |
| `vulnerability_finding_observation` | OCSF base event fields plus `finding_info`, `vulnerabilities`, and at least one affected-resource path such as `resources`. |
| `authentication_observation` | OCSF base event fields plus `user` and at least one of `service` or `dst_endpoint`. |
| `dns_observation` | OCSF base event fields plus `query`; response rows require response-code handling. |
| `dhcp_ipam_observation` with DHCP discriminator | OCSF base event fields plus DHCP endpoint context and `transaction_uid` when present. |
| `network_activity_observation` | OCSF base event fields plus endpoint paths satisfying the compiled network constraint. |
| `cadastre_only` | Cadastre envelope only. |

A required path may be absent only when the active row declares a field-quality behavior that the compiled profile permits. A forbidden path must not be emitted even when the source supplies the value.

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

`SourceExtensionFieldRuleSet` is an activation-controlled artifact. The default rule set is empty. An empty or missing rule set permits no `source_extension_fields` output.

| `SourceExtensionFieldRule` field | Required | Default or omission behavior | Rule |
| --- | ---: | --- | --- |
| `rule_id` | Yes | none | Stable row ID scoped to `050`. |
| `namespace` | Yes | none | Must be a declared namespace matching the path prefix. Wildcards are forbidden. |
| `field_path` | Yes | none | Must be exact; wildcard field paths are forbidden. |
| `type` | Yes | none | Closed scalar or container token permitted by `040.SourceExtensionFieldRuleShape` and narrowed by `050`. |
| `bounds` | Yes | none | Must state every applicable length, element, map-entry, numeric, and canonical-byte bound. |
| `redaction` | Yes | no default | One of `caller_visible`, `audit_only`, or `redacted`. |
| `collision_policy` | Yes | `reject_path_collision` and `reject_type_collision` | Must reject path and type collision unless a narrower owner policy exists. |
| `secret_scan_policy` | Yes | `reject_before_output` | Audit-only storage requires an active redaction policy. |
| `ocsf_reserved_name_policy` | Yes | reject | Reject when path equals or is prefixed by an OCSF reserved base-event field unless an active row permits non-authoritative metadata. |
| `permitted_observation_types` | Yes | `[]` | Empty means no observation type may emit the field. |
| `permitted_mapping_bundle_refs` | Yes | `[]` | Empty means no mapping bundle may emit the field. |
| `validation_fixture_refs` | Yes | none | Non-empty before activation. |
| `activation_scope` | Yes | none | Scope in which the rule may affect output. |
| `lifecycle_status` | Yes | none | Imported from `030.LifecycleStatus`; production mapping requires `active`. |

| Case | Required behavior |
| --- | --- |
| Missing rule set | Use empty rule set. No source-extension fields may be emitted. |
| Missing exact field rule | Fail with `SOURCE_EXTENSION_FIELD_UNDECLARED`. |
| Wildcard namespace | Forbidden. |
| Unknown namespace | Forbidden. |
| Path grammar | `source.<source_category>.<field_name>` or a stricter declared source-category grammar; segments must be lowercase snake case and non-empty. |
| Bounds | Strings max 4096 Unicode scalar values; arrays max 1024 elements; maps max 4096 entries; canonical objects max 65536 bytes unless the row narrows the bound. |
| Redaction | No implicit default; each rule must declare `caller_visible`, `audit_only`, or `redacted`. |
| Collision policy | Default `reject_path_collision` and `reject_type_collision`. |
| Secret scan | Default `reject_before_output`; audit-only storage requires an active redaction policy. |
| OCSF reserved-name collision | Default reject when path equals or is prefixed by an OCSF reserved base-event field. |

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

### MappingApiHandoff

`050` mapping, OCSF, external enum, and source-extension failures must be emitted as structured owner errors for `110.ErrorCodeRegistry`; they must not become source authority or raw source-value leakage.

| Mapping condition | API-visible state | Required behavior |
| --- | --- | --- |
| mapping row missing | `error` | Emit `MAP_OCSF_ROW_MISSING`; do not emit silver output for that observation. |
| mapping row ambiguous | `error` | Emit `MAP_OCSF_ROW_AMBIGUOUS`; do not select a row by implementation order. |
| source-extension secret-scan failure | `security_error` | Emit `SOURCE_EXTENSION_SECRET_SCAN_FAILED`; caller-visible value redacted. |
| OCSF forbidden field emitted | `security_error` | Emit `OCSF_FORBIDDEN_FIELD_EMITTED`; field path visible, value redacted. |
| external-schema non-authority failure | `error` | Must not render as pass, fail, authorized absence, graph mutation, remediation, cleanup, or watermark. |
| unknown or deprecated enum | `error` | Preserve raw value only through redacted evidence; do not invent enum IDs. |

Source-extension field values remain redacted unless a redaction policy explicitly permits display for the data class and caller.

### Mapping diagnostics and errors

| Error code | Emitted when |
| --- | --- |
| `MAPPING_ARTIFACT_INACTIVE` | A required mapping, parser, schema, enum, profile, extension, CIM, or compiler artifact is not active for the execution mode. |
| `MAPPING_ARTIFACT_SCOPE_MISMATCH` | A required mapping artifact does not cover the execution scope. |
| `MAPPING_ARTIFACT_CHECKSUM_MISMATCH` | A required mapping artifact checksum does not match the active ref or manifest. |
| `MAPPING_CORE_CONTRACT_OVERRIDE_FORBIDDEN` | A mapping artifact attempts to redefine core fields, omission states, identity semantics, temporal semantics, graph semantics, or source-authority semantics. |
| `OCSF_ARTIFACT_MISMATCH` | Active profile refs do not match the compiled artifact checksum, source tag, source commit, compiler, or validator evidence. |
| `OCSF_CLASS_NOT_ALLOWED` | A mapping emits an OCSF class outside the active class allowlist. |
| `MAP_OCSF_ROW_MISSING` | No active `ObservationToOCSFMappingRow` matches the observation type and discriminator. |
| `MAP_OCSF_ROW_AMBIGUOUS` | More than one active `ObservationToOCSFMappingRow` matches the observation type and discriminator. |
| `OCSF_ACTIVITY_DISCRIMINATOR_MISSING` | Required activity discriminator evidence is absent or ambiguous for a class with no default activity. |
| `OCSF_REQUIRED_OBJECT_PATH_MISSING` | A row's required OCSF object path is absent after mapping and no permitted field-quality state explains the absence. |
| `OCSF_FORBIDDEN_FIELD_EMITTED` | A mapping emits a forbidden OCSF path, deprecated field without waiver, disabled base-event field, or disallowed reserved field. |
| `MAPPING_OBSERVATION_TYPE_SPLIT_REQUIRED` | One Cadastre observation type contains source cases that cannot safely map to one OCSF class family without a discriminator split. |
| `SOURCE_EXTENSION_RULESET_MISSING` | A non-empty `source_extension_fields` output is attempted while the selected row has no active source-extension rule set. |
| `SOURCE_EXTENSION_NAMESPACE_INVALID` | A source-extension namespace is unknown, wildcarded, malformed, or outside the declared source category. |
| `SOURCE_EXTENSION_REDACTION_POLICY_MISSING` | A source-extension rule omits required redaction behavior. |
| `SOURCE_EXTENSION_SECRET_SCAN_FAILED` | A source-extension value matches the active secret scan rejection policy. |
| `SOURCE_EXTENSION_OCSF_RESERVED_COLLISION` | A source-extension path collides with an OCSF reserved base-event field path. |
| `EXTERNAL_ENUM_UNKNOWN` | Source enum value is not mapped and no unknown-value policy applies. |
| `EXTERNAL_ENUM_OTHER_NOT_PERMITTED` | Mapping attempts OCSF `Other` without an active rule preserving the raw value. |
| `EXTERNAL_ENUM_DEPRECATED` | Mapping emits a deprecated enum or field without a non-expired waiver. |
| `EXTERNAL_ENUM_SIBLING_MISMATCH` | Enum ID/name sibling values do not match the compiled artifact. |
| `SOURCE_EXTENSION_FIELD_UNDECLARED` | Mapping emits a source extension field without an active `SourceExtensionFieldRule`. |

### MappingErrorRegistryFragment

This owner fragment feeds `110.GenerateErrorCodeRegistry`. `110` owns the generated caller-visible registry. This table must not render API output by itself. Rows with `TODO:` cells block authoritative promotion and must be resolved by the owning domain before `110-ERROR-REGISTRY-TOTAL-AC-001` can pass.

| error_code | owner_spec | severity | retry_class | caller_visible_fields | audit_visible_fields | redaction_rule | owner_context_schema_ref | fixture_ref |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `MAPPING_ARTIFACT_INACTIVE` | `050` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `050.MappingErrorContext` | `error-registry-050-mapping-artifact-inactive` |
| `MAPPING_ARTIFACT_SCOPE_MISMATCH` | `050` | `error` | `caller_correctable` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `050.MappingErrorContext` | `error-registry-050-mapping-artifact-scope-mismatch` |
| `MAPPING_ARTIFACT_CHECKSUM_MISMATCH` | `050` | `error` | `retry_after_owner_repair` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `050.MappingErrorContext` | `error-registry-050-mapping-artifact-checksum-mismatch` |
| `MAPPING_CORE_CONTRACT_OVERRIDE_FORBIDDEN` | `050` | `security_error` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.security_boundary` | `050.MappingErrorContext` | `error-registry-050-mapping-core-contract-override-forbidden` |
| `OCSF_ARTIFACT_MISMATCH` | `050` | `error` | `retry_after_owner_repair` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `050.MappingErrorContext` | `error-registry-050-ocsf-artifact-mismatch` |
| `OCSF_CLASS_NOT_ALLOWED` | `050` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `050.MappingErrorContext` | `error-registry-050-ocsf-class-not-allowed` |
| `MAP_OCSF_ROW_MISSING` | `050` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `050.MappingErrorContext` | `error-registry-050-map-ocsf-row-missing` |
| `MAP_OCSF_ROW_AMBIGUOUS` | `050` | `error` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `050.MappingErrorContext` | `error-registry-050-map-ocsf-row-ambiguous` |
| `OCSF_ACTIVITY_DISCRIMINATOR_MISSING` | `050` | `error` | `caller_correctable` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `050.MappingErrorContext` | `error-registry-050-ocsf-activity-discriminator-missing` |
| `OCSF_REQUIRED_OBJECT_PATH_MISSING` | `050` | `error` | `caller_correctable` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `050.MappingErrorContext` | `error-registry-050-ocsf-required-object-path-missing` |
| `OCSF_FORBIDDEN_FIELD_EMITTED` | `050` | `security_error` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.security_boundary` | `050.MappingErrorContext` | `error-registry-050-ocsf-forbidden-field-emitted` |
| `MAPPING_OBSERVATION_TYPE_SPLIT_REQUIRED` | `050` | `error` | `caller_correctable` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `050.MappingErrorContext` | `error-registry-050-mapping-observation-type-split-required` |
| `SOURCE_EXTENSION_RULESET_MISSING` | `050` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `050.MappingErrorContext` | `error-registry-050-source-extension-ruleset-missing` |
| `SOURCE_EXTENSION_NAMESPACE_INVALID` | `050` | `security_error` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.security_boundary` | `050.MappingErrorContext` | `error-registry-050-source-extension-namespace-invalid` |
| `SOURCE_EXTENSION_REDACTION_POLICY_MISSING` | `050` | `security_error` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.security_boundary` | `050.MappingErrorContext` | `error-registry-050-source-extension-redaction-policy-missing` |
| `SOURCE_EXTENSION_SECRET_SCAN_FAILED` | `050` | `security_error` | `none` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.always_forbidden_sensitive_values` | `050.MappingErrorContext` | `error-registry-050-source-extension-secret-scan-failed` |
| `SOURCE_EXTENSION_OCSF_RESERVED_COLLISION` | `050` | `security_error` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.security_boundary` | `050.MappingErrorContext` | `error-registry-050-source-extension-ocsf-reserved-collision` |
| `EXTERNAL_ENUM_UNKNOWN` | `050` | `error` | `caller_correctable` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `050.MappingErrorContext` | `error-registry-050-external-enum-unknown` |
| `EXTERNAL_ENUM_OTHER_NOT_PERMITTED` | `050` | `error` | `caller_correctable` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `050.MappingErrorContext` | `error-registry-050-external-enum-other-not-permitted` |
| `EXTERNAL_ENUM_DEPRECATED` | `050` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `050.MappingErrorContext` | `error-registry-050-external-enum-deprecated` |
| `EXTERNAL_ENUM_SIBLING_MISMATCH` | `050` | `error` | `caller_correctable` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `050.MappingErrorContext` | `error-registry-050-external-enum-sibling-mismatch` |
| `SOURCE_EXTENSION_FIELD_UNDECLARED` | `050` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `050.MappingErrorContext` | `error-registry-050-source-extension-field-undeclared` |

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

### MappingErrorContext

`MappingErrorContext` is the owner context schema for `050` parser, mapping, OCSF, external enum, and source-extension registry rows.

| Field | Required | Rule |
| --- | ---: | --- |
| `context_schema_version` | Yes | Immutable `050` context schema version. |
| `owner_spec` | Yes | Must be `050`. |
| `error_code` | Yes | Must match the generated registry row. |
| `failure_class` | Yes | Closed token: `mapping_artifact`, `external_schema`, `ocsf_mapping`, `external_enum`, `source_extension`, or `core_override`. |
| `operation` | Yes | Parser, normalization, mapping resolution, source-extension validation, enum mapping, or mapping-bundle validation operation. |
| `affected_record_type` | Yes | Mapping artifact, observation, mapping row, source-extension rule, enum rule, or validation output type. |
| `field_path` | Yes | Exact normalized field, source-extension path, OCSF path, enum path, or null for artifact-wide failures. |
| `mapping_artifact_ref` | No | Required when a mapping artifact was consulted. |
| `external_schema_profile_ref` | No | Required for OCSF or external schema failures. |
| `source_extension_rule_ref` | No | Required for source-extension failures. |
| `raw_value_ref` | No | Redacted ref or checksum only; raw source values must not be stored in caller context. |
| `validation_refs` | Yes | Exact `120` mapping fixture refs. |
| `redaction_classes` | Yes | Raw payload bytes, raw source values, credentials, private bindings, source-native identity values, and raw OCSF payload bytes must map to `always_forbidden`. |

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
| `050-OCSF-MAP-AC-001` | Every active MVP observation mapping resolves to exactly one `ObservationToOCSFMappingRow` or fails before normalized output. |
| `050-OCSF-MAP-AC-002` | Every active row's category UID, class UID, activity ID, and type UID match the compiled OCSF 1.8.0 artifact recorded by `ExternalSchemaArtifactRef`. |
| `050-OCSF-MAP-AC-003` | The active class allowlist contains exactly the class UIDs used by active mapping rows. |
| `050-OCSF-MAP-AC-004` | Unknown enum and source-action values do not invent OCSF enum IDs. |
| `050-OCSF-MAP-AC-005` | Authentication has no default activity. |
| `050-OCSF-MAP-AC-006` | IPAM-only assignment rows do not map to OCSF DHCP Activity. |
| `050-EXTERNAL-SCHEMA-AUTHORITY-HANDOFF-AC-001` | A mapping bundle that attempts to use OCSF class, status, severity, confidence, observables, enrichments, endpoint order, field presence, or field absence as authority without an active `060.ExternalSchemaAuthoritySignalMappingRow` fails before authority, absence, cleanup, graph expiry, retraction, control-state, or watermark effect. |
| `050-EXTERNAL-SCHEMA-AUTHORITY-HANDOFF-AC-002` | Valid normalized metadata remains persistable when the authority effect is blocked, provided the mapping itself is valid and emits no forbidden authority output. |
| `050-SOURCE-EXT-AC-002` | The default `SourceExtensionFieldRuleSet` is empty. |
| `050-SOURCE-EXT-AC-003` | Wildcard source-extension paths are rejected. |
| `050-BASE-FIELD-AC-001` | `raw_data`, `raw_data_hash`, and `unmapped` remain absent unless an explicit bounded policy permits them. |
| `050-NONAUTH-AC-001` | OCSF observables, enrichments, status, severity, confidence, and endpoint ordering cannot create identity, source authority, omission, gold, temporal, graph, or reachability effects. |
| `050-LIFECYCLE-AC-001` | Every `050` activation-controlled artifact entering `active` status has a `030.LifecycleTransitionEvidence` ref for the generic artifact lifecycle and mapping-specific guard rows. |
| `050-PROJECTION-REPLAY-AC-001` | Exact projection replay matches projection profile, mapping refs, redaction refs, output checksum, loss-manifest checksum, and `VersionManifest` ref. |
| `050-PROJECTION-REPLAY-AC-002` | Projection profile mismatch, loss-manifest mismatch, redaction-policy mismatch, or output checksum mismatch blocks replay before output. |
| `050-PROJECTION-REPLAY-AC-003` | Differences only in export job ID, request correlation ID, execution duration, or display timestamp are excluded volatile-field differences. |
| `050-FLOW-ROLE-HANDOFF-AC-001` | Explicit source direction evidence accepted by a mapping rule emits `FlowRoleEvidence` with evidence refs and field-quality metadata. |
| `050-FLOW-ROLE-HANDOFF-AC-002` | OCSF endpoint fields without explicit flow-role evidence emit no `FlowRoleEvidence` item and cannot determine graph direction. |
| `050-FLOW-ROLE-HANDOFF-AC-003` | Ambiguous endpoint roles, NAT/proxy uncertainty, aggregation uncertainty, DNS, and DHCP defaults emit no observed-connection direction. |
| `050-FLOW-ROLE-HANDOFF-AC-004` | `cadastre_only` network-context rows remain metadata unless `090` consumes qualifying flow-role evidence. |
| `050-MVP-OCSF-CLOSURE-AC-001` | Every active MVP observation type resolves to exactly one active mapping row or exact `cadastre_only` row, and every row has compiled artifact refs, policy refs, validation refs, and non-`TODO` expected output checksums. |
| `050-GOLD-OBJECT-BOUNDARY-AC-001` | OCSF object fields do not become `GoldFact.object_value.structured_value` without an active `080` structured schema ref and matching authority handoff when authority is required. |
| `050-GOLD-OBJECT-BOUNDARY-AC-002` | OCSF `raw_data` and `unmapped` are not `EvidenceRef` artifacts and cannot set `EvidenceRef.artifact_id`. |
| `050-GOLD-OBJECT-BOUNDARY-AC-003` | Observables, enrichments, source-extension fields, and endpoint fields are not `GoldFact.subject_ref` or `GoldFact.object_value` references by mapping-side inference. |
| `050-GOLD-OBJECT-BOUNDARY-AC-004` | A mapping attempt to set `GoldFact.object_value`, `GoldFact.subject_ref`, `object_kind`, source authority, absence outcome, correction output, graph output, or watermark output fails before silver output or emits the most specific owner diagnostic. |

## Definition of Done

| ID | Criterion |
| --- | --- |
| `050-AC-001` | Every production mapping bundle emits deterministic `CanonicalValidationOutput`. |
| `050-AC-002` | OCSF-aligned observations validate against the exact compiled external schema artifact. |
| `050-AC-003` | Undeclared source extension fields fail before production output. |
| `050-AC-004` | CIM projection records every lossy or unsupported mapping and never becomes authoritative input. |
| `050-AC-005` | Every active external mapping row set is exhaustive for its declared MVP observation scope and every non-active unresolved mapping remains a blocking owner `TODO:` outside production output. |
| `050-AC-006` | Every emitted `source_extension_fields` path has exactly one active source-extension rule, and the default rule set emits no fields. |

## Open Questions

Open questions marked `TODO:` block authoritative status for the affected contract. A downstream implementation must not resolve a `TODO:` by inference.

| ID | Question | Blocking scope | Required owner decision | Default until resolved |
| --- | --- | --- | --- | --- |
| `050-TODO-ERROR-FRAGMENT-COMPLETION` | TODO: Complete all `050` owner error fragments using `110.ErrorCodeRegistryRow` fields with non-`TODO` severity, retry class, caller-visible fields, audit fields, redaction rule, and validation fixture refs. | Error registry generation and API/export visibility. | `050` plus `110.GenerateErrorCodeRegistry` validation. | `110.GenerateErrorCodeRegistry` fails with `ERROR_REGISTRY_OWNER_FRAGMENT_INCOMPLETE`. |
