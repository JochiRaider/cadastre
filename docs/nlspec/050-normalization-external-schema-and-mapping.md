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
- `030.ActivationScope`
- `030.ScopeSelectorContext`
- `030.ScopeSelectorCovers`
- `030.ActivationControlledRowSchema`
- `030.ActivationControlledRowField`
- `030.ActivationControlledRowRef`
- `030.ActivationControlledRowSetSchema`
- `020.ResolveSourceDatasetCatalogRow`
- `020.SourceDatasetCatalogRow`
- `080.GoldFactPredicateContractRow`
- `080.MVPGoldFactStructuredValueSchemaCatalog`

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
- `MVPStructuredGoldObjectMappingHandoff`
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

### CadastreOnlyMappingOutputPolicy

`cadastre_only` is an explicit row mode, not a fallback. `NormalizeObservation` must emit a null external schema profile only when `ResolveOCSFMapping` selects exactly one active `ObservationToOCSFMappingRow` with `row_mode = cadastre_only` for the normalized request scope and discriminator state.

| Output element | Required behavior for selected `cadastre_only` row |
| --- | --- |
| `external_schema_profile_id` | Must be `null`. A non-null value fails before persistence. |
| OCSF category, class, activity, and type metadata | Must not be emitted in `normalized_fields`, row diagnostics, graph handoff metadata, or authority handoff metadata. |
| `normalized_fields` | Must be `{}` for MVP. Non-empty values require a later owner patch that defines Cadastre-owned normalized field shape, checksums, and validation rows. |
| `normalized_payload_checksum` inputs | Must include selected mapping row ref, selected mapping row checksum, `observation_type`, `cadastre_only = true`, and canonical empty `normalized_fields` bytes. |
| Authority, identity, absence, temporal, graph, cleanup, retraction, and watermark effects | Must be `none`. A `cadastre_only` row does not grant any owner outside `050` an effect. |
| `source_extension_fields` | Must remain `{}` unless the selected row also references an active `SourceExtensionFieldRuleSet` that permits exact paths. |
| `VersionManifest` | Must include the selected `cadastre_only` row ref, selected row checksum, row-set ref, row-set checksum, validation refs, package-set refs when package-supplied, and lifecycle transition evidence refs. |

A missing OCSF mapping row must not infer `cadastre_only`. Missing row and selected `cadastre_only` row are observably different states and must produce different owner errors, validation rows, and manifest refs.

## External Schema Profile Contract

`ExternalSchemaProfile` must pin external schema name, version, source tag, source commit, compiler version, validator version, compiled artifact checksum, class allowlist, profile set, extension set, deprecated-field policy, and enum mapping behavior.

MVP OCSF baseline is `OCSF 1.8.0` as a candidate production baseline. The profile must remain non-active until `ExternalSchemaArtifactRef` records the exact source tag, source commit, compiler ID/version/checksum, validator ID/version/checksum, compiled artifact checksum, profile set, extension set, and class allowlist checksum. Use of OCSF `main`, development versions, or main-only fields is forbidden for production output.

Production OCSF profile status must be `active` only after `ExternalSchemaArtifactRef`, `ProfileResolutionManifest`, `OCSFBaseEventFieldPolicy`, `ExternalEnumMappingRule`, `OCSFProfileUpgradeReport` when applicable, and `ObservationTypeExternalMappingValidationMatrix` pass.

### MVP OCSF Production Baseline Decision

The MVP production candidate baseline is pinned to OCSF `1.8.0`.

| Baseline field | Required value |
| --- | --- |
| `schema_name` | `OCSF` |
| `schema_version` | `1.8.0` |
| `source_tag` | `1.8.0` |
| `source_commit` | `6fa6499a0f8c9f449d342816e90e5f687c224b0a` |
| `development_branch_policy` | OCSF `main`, development versions, and main-only fields are forbidden for production output. |
| `production_activation_default` | `blocked_validation` until every artifact and checksum below is concrete and non-`TODO`. |

Production OCSF-aligned output must remain inactive until the selected profile has concrete refs and checksums for source tag, source commit, compiler ID/version/checksum, validator ID/version/checksum, compiled artifact checksum, profile set checksum, extension set checksum, class allowlist checksum, `ProfileResolutionManifest` refs, active mapping row refs, validation refs, package-set refs when package-supplied, lifecycle transition evidence, and `030.VersionManifest` inclusion.

A row, fixture, validation report, package label, ADR, research report, branch name, repository path, or source prose that omits any required checksum must fail production activation with `OCSF_CANONICAL_VALIDATION_OUTPUT_INCOMPLETE`, `OCSF_ARTIFACT_MISMATCH`, or the more specific owner error.

### ExternalSchemaProfile row schema

`ExternalSchemaProfile` is the stable interface for a production-eligible external schema profile. Concrete profile rows are activation-controlled artifacts. A profile row may affect output only when lifecycle status is `active`, validation refs are non-empty, and every output-affecting ref appears in `030.VersionManifest`.

| Field | Required behavior |
| --- | --- |
| `external_schema_profile_id` | Stable ID scoped to `050`. |
| `schema_name` | `OCSF` for MVP. |
| `schema_version` | Exact version; default MVP candidate is `1.8.0`. |
| `schema_artifact_ref` | Active `ExternalSchemaArtifactRef`. |
| `profile_resolution_manifest_refs` | Non-empty for every emitted class or object path. |
| `class_allowlist_checksum` | Must match the active artifact ref. |
| `profile_set` | Explicit set; empty set is allowed only when the compiled artifact permits no profiles. |
| `extension_set` | Explicit set; empty set is allowed and checksummed. |
| `deprecated_field_policy_ref` | Active row or default reject policy. |
| `enum_rule_set_ref` | Active rule set when any enum path maps. |
| `base_event_field_policy_set_ref` | Active policy set. |
| `source_extension_rule_set_ref` | Active rule set or explicit empty rule set. |
| `upgrade_report_ref` | Required when replacing an active production profile. |
| `validation_refs` | Non-empty. |
| `activation_scope` | `030.ActivationScope`; validated through `050.MappingScopeSelectorContext` before the row may affect output. |
| `lifecycle_status` | Production use requires `active`. |

### MappingScopeSelectorContext

`MappingScopeSelectorContext` is the owner context family for `050` mapping, external schema profile, source extension, and mapping-tool activation scopes. It instantiates `030.ScopeSelectorContext`; it does not define selector equality, coverage, specificity, subset rules, or ambiguity behavior.

| Row family | Required request dimensions | Optional dimensions | Subset-allowed dimensions | Owner-local predicates kept outside scope matching |
| --- | --- | --- | --- | --- |
| `ExternalSchemaProfile` | `source_category`, `source_dataset`, `schema_name`, `schema_version` | `observation_type`, `mapping_bundle_ref` | none | compiled artifact validation, class allowlist, profile set, extension set, lifecycle status. |
| `ObservationToOCSFMappingRow` | `source_category`, `source_dataset`, `observation_type` | `schema_name`, `schema_version`, `mapping_bundle_ref` | none | mapping discriminator evaluation, OCSF category/class/activity/type validation, source-extension validation. |
| `MappingProjectManifest` | `source_category`, `source_dataset`, `mapping_project_ref` | `observation_type`, `schema_name`, `schema_version` | none | repository snapshot validation, compiler pipeline validation, plugin policy. |
| `SourceExtensionFieldRule` | `source_category`, `source_dataset`, `field_path` | `observation_type`, `mapping_bundle_ref` | none | namespace collision, secret scanning, OCSF reserved-name policy. |

`ResolveOCSFMapping` must normalize the request execution scope, resolve `source_dataset` through `020.ResolveSourceDatasetCatalogRow` before scope-filtered row selection when a row uses `source_dataset`, reject under-scoped requests through the owner error map, and keep only active mapping/profile rows whose `activation_scope` covers the request through `030.ScopeSelectorCovers` before mapping discriminator evaluation. Discriminator ordering and OCSF class validation remain owner-local and must run only after source-dataset catalog resolution and scope-filtered row selection. Missing, ambiguous, inactive, private-leaking, checksum-mismatched, unmanifested, or deterministically blocked source-dataset rows fail mapping activation or emit the owner diagnostic declared by the selected mapping execution mode. A valid normalized observation remains non-authoritative unless `060` later maps a specific signal to a specific requested effect.

Scope mismatch must remain distinguishable from missing discriminator input and ambiguous discriminator input. A mapping selection that fails scope coverage must not emit silver output.

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

Authority effects are owned by `060.ExternalSchemaAuthoritySignalMappingRow`; this section defines no authority, absence, cleanup, graph-expiry, control-state, or watermark behavior.

If a mapping bundle intends an external schema field to be considered by authority logic, it must emit the normalized field as observation metadata only. A separate active `060.ExternalSchemaAuthoritySignalMappingRow` must name the external schema profile ref, field path, source dataset, selected `020.SourceDatasetCatalogRow` ref/checksum, fact type, predicate, requested effect, authority row ref, coverage row ref when applicable, staleness row ref, validation refs, activation scope, and lifecycle status.

`060.ExternalSchemaAuthoritySignalMappingRow` must also reference the selected `SourceAuthorityClosureMatrixRow` or deterministic block row, `SourceAuthorityProfileRow`, `CoverageDimensionProfile` when coverage-sensitive, `SourceStalenessPolicy`, `ControlResultMappingRow` when control-state output is requested, `AbsenceDerivationPolicy` when absence-sensitive output is requested, `ProjectionWatermarkPolicy` when watermark is requested, and `030.VersionManifest` refs for the selected source-dataset catalog row and every consulted source-authority row.

Missing `060.ExternalSchemaAuthoritySignalMappingRow` means the external schema signal remains non-authoritative. A mapping row or diagnostic that attempts to set authority, absence, cleanup, retraction, graph expiry, control pass/fail, or watermark effects must route the failure to imported `060.EXTERNAL_SCHEMA_AUTHORITY_FORBIDDEN` before the attempted authority effect. The normalized fields may remain persisted as observation metadata when the mapping itself is valid.

OCSF field absence, OCSF object absence, status, severity, confidence, observables, enrichments, endpoint order, `raw_data`, `unmapped`, external enum values, or source-extension values must not set `040.FactAbsenceOutcome`, cleanup, retraction, graph expiry, control pass/fail, or watermark behavior without an exact active `060.SourceAuthorityClosureMatrixRow` or deterministic block row.

#### ExternalSchemaSignalAuthorityAttemptMatrix

Every attempt to use external-schema material as authority must resolve through this matrix before any authority effect can be requested from `060`.

| Attempt state | Required behavior | Required owner error or state |
| --- | --- | --- |
| No matching `060.ExternalSchemaAuthoritySignalMappingRow` | Preserve normalized field only; no authority effect, absence, cleanup, retraction, graph expiry, watermark, control pass/fail, or API authorized-negative output. | `EXTERNAL_SCHEMA_AUTHORITY_SIGNAL_ROW_MISSING` for attempted authority; otherwise no-op metadata. |
| Matching row inactive | Same as missing row; inactive rows do not weaken the non-authority default. | `EXTERNAL_SCHEMA_AUTHORITY_SIGNAL_ROW_INACTIVE`. |
| Matching deterministic block row | Emit no authority effect and require mutation-prohibition refs. | `EXTERNAL_SCHEMA_AUTHORITY_SIGNAL_BLOCKED`. |
| Row checksum, package-set, validation, or manifest mismatch | Reject the authority attempt before effect evaluation. | Most specific `050`, `060`, `100`, or `030` owner error. |
| External schema signal permitted only as non-authoritative normalized field | Persist only valid normalized observation metadata. | No authority error unless an effect is attempted. |
| Exact active `060` mapping row and exact closure row chain | Route the effect request to `060`; `050` still grants no authority by itself. | `060` owns the selected result. |

For every non-exact state above, the required output is no authority effect, no absence, no cleanup, no retraction, no graph expiry, no watermark, no control pass/fail, no remediation, and no API authorized-negative output. Mapping and normalization code must not annotate normalized fields as authority. A mapping row that attempts authority without exact `060` closure must emit an owner-specific error rather than a generic mapping diagnostic when such an owner-specific error exists.

Active `ObservationToOCSFMappingRow`, `ExternalSchemaProfile`, and `SourceExtensionFieldRule` selections that use `source_dataset` must include `source_dataset_catalog_row_ref` and `source_dataset_catalog_row_checksum` in their activation-controlled row refs and in `030.VersionManifest` when they affect output.

### SourceDatasetMappingCatalogHandoff

Every active `ObservationToOCSFMappingRow`, `ExternalSchemaProfile`, `SourceExtensionFieldRule`, and `MappingProjectManifest` that uses `source_dataset` must include the selected `020.SourceDatasetCatalogRow` ref/checksum, the source-dataset row-set ref/checksum, selector checksum when scoped, validation refs, package-set refs when package-supplied, and `030.VersionManifest` refs before mapping activation can emit silver output.

A missing, deterministically blocked, ambiguous, inactive, checksum-mismatched, private-leaking, unvalidated, package-set-missing, or unmanifested source-dataset row fails mapping activation before silver output when the row is required for mapping selection. The failure must use the most specific `020`, `030`, `050`, `100`, or `110` owner error. A mapping implementation must not infer `source_dataset` from OCSF category, class, activity, object path, source extension field, external schema profile, package name, module name, artifact filename, repository path, validation run, feed category, table path, or private binding.

| Source dataset or signal | Mapping output allowed | Authority and absence behavior |
| --- | --- | --- |
| `directory_inventory` | Observation metadata and resolver-input fields only. | Must not authorize `identity_membership_fact.member_of`, nonmembership, user deletion, group deletion, source absence, graph edge removal, cleanup, graph expiry, or gold output. |
| `source_history` | Observation metadata and supporting evidence refs only. | Must not authorize no-change proof, disappearance proof, negative output, cleanup, retraction, graph expiry, watermark, or compliance negative output without exact `060` retention and closure refs. |
| `network_flow` | Endpoint, time-window, sensor, and flow-role evidence fields may be preserved. | Must not infer absence, source/destination direction, graph edge direction, graph expiry, cleanup, or theoretical reachability from OCSF endpoint order or missing rows. Direction requires `050.FlowRoleEvidence` and graph rules in `090`. |

### GoldFactObjectBoundaryHandoff

`NormalizeObservation` and `ResolveOCSFMapping` must not emit `GoldFact` bytes, `GoldFact.subject_ref`, `GoldFact.object_value`, `EvidenceRef.artifact_id`, graph deltas, source authority decisions, absence outcomes, correction outputs, or watermark effects. Mapping output may supply observation metadata only. Gold object values may be derived only by `080.DeriveFacts` after `040` one-of validation, `060` authority validation, and `080` predicate-contract validation.

OCSF objects may become `structured_value` only through an active `080.GoldFactPredicateContractRow.structured_value_schema_refs` entry and a matching `060` authority row when authority is required. OCSF `raw_data`, `unmapped`, observables, enrichments, status, severity, and confidence remain non-authoritative unless exact `050`, `060`, and `080` handoff rows permit a bounded interpretation.

`source_extension_fields` are never subject/object authority by themselves. A mapping bundle that attempts to set `object_kind`, `object_value.kind`, `GoldFact.subject_ref`, `EvidenceRef.artifact_id`, `absence_outcome`, source authority, correction output, graph output, or watermark output must fail before silver output or emit a mapping diagnostic using the most specific owner error.

### MVPStructuredGoldObjectMappingHandoff

Mapping output may preserve inputs used later by `080.DeriveFacts`, but it must not construct gold object values. Every MVP structured gold object schema must route through `080` and exact authority handoffs.

| Structured schema ref or evidence object | Permitted mapping output | Required downstream refs before gold or graph output | Forbidden shortcut |
| --- | --- | --- | --- |
| `080.struct.os_descriptor.v1` | Observation metadata in `normalized_fields` or declared `source_extension_fields`. | Exact `ObservationToOCSFMappingRow` or `cadastre_only` row, `ExternalSchemaProfile` and `ProfileResolutionManifest` when OCSF-derived, selected `020.SourceDatasetCatalogRow`, active `080` predicate row, active schema ref, and exact `060` authority row. | Direct `GoldFact.object_value` construction from OCSF device or OS objects. |
| `080.struct.host_lifecycle_state.v1` | Observation metadata only. | Exact mapping row refs, source-dataset catalog row, active `080` predicate row, active schema ref, and exact `060` authority/staleness rows. | Treating OCSF status, activity, severity, or field absence as lifecycle truth. |
| `080.struct.host_management_state.v1` | Observation metadata only. | Exact mapping row refs, source-dataset catalog row, active `080` predicate row, active schema ref, and exact `060` authority rows. | Treating management-plane labels or source-extension fields as canonical management state. |
| `080.struct.vulnerability_instance_key.v1` | Observation metadata only. | Exact mapping row refs, source-dataset catalog row, active `080` predicate row, active schema ref, exact `060` authority, coverage, staleness, and absence refs. | Treating scanner severity, confidence, or missing rows as vulnerability truth or absence. |
| `080.struct.software_instance_key.v1` | Observation metadata only. | Exact mapping row refs, source-dataset catalog row, active `080` predicate row, active schema ref, and exact `060` authority rows. | Treating arbitrary package strings as identity without schema and authority validation. |
| `080.struct.control_evaluation_key.v1` | Observation metadata only. | Exact mapping row refs, source-dataset catalog row, active `080` predicate row, active schema ref, exact `060.ControlResultMappingRowSet`, coverage, staleness, and authority refs. | Treating external control result strings as pass/fail/unknown without `060` mapping. |
| `080.struct.observed_exposure_key.v1` | Observation metadata only. | Exact mapping row refs, source-dataset catalog row, active `080` predicate row, active schema ref, and exact positive observed-exposure authority rows. | Treating observed exposure as theoretical reachability, service access, or identity-conditioned access. |
| `FlowRoleEvidence` for `gfp-mvp-flow-observed-connection-v1` | Cadastre-owned flow-role evidence with source path refs, ambiguity state, and field-quality metadata. | `090` projection may consume it only with `gfp-mvp-flow-observed-connection-v1`, both endpoint identities resolved, and graph profile/semantics/eligibility refs active. | Using OCSF endpoint order, provider field names, or source/destination labels alone as graph direction. |

If a structured object value requires a source path not expressible in active OCSF artifacts, the mapping bundle must use an exact `cadastre_only` mapping row or an active `SourceExtensionFieldRule`. The mapping row still emits observation metadata only. Gold object construction remains a later `080` responsibility after source authority and predicate validation.

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
| `row_version` | Yes | none | Immutable owner schema version included in row checksum and `VersionManifest`. |
| `row_mode` | Yes | none | Closed enum: `ocsf_mapped`, `cadastre_only`, or `deterministically_blocked`. Conditional required and forbidden fields are selected by this mode. |
| `observation_type` | Yes | none | Must match `040.CadastreSilverObservationSchema.observation_type`. |
| `mapping_discriminator` | Yes | none | Canonical predicate over normalized parse fields, source metadata, and declared mapping-bundle inputs. It must not inspect graph, gold, identity, or API state. |
| `source_dataset_catalog_row_ref` | Yes when row selection uses `source_dataset` | none | Structured `030.ActivationControlledRowRef` for the selected `020.SourceDatasetCatalogRow` or deterministic source-dataset block row. Bare strings fail. |
| `source_dataset_catalog_row_checksum` | Yes when row selection uses `source_dataset` | none | SHA-256 of the selected source-dataset row after defaults materialize. |
| `external_schema_profile_ref` | Required when `row_mode = ocsf_mapped` | null only when `row_mode = cadastre_only` or `deterministically_blocked` | Must reference an active `ExternalSchemaProfile`. |
| `external_schema_artifact_ref` | Required when `row_mode = ocsf_mapped` | null otherwise | Must reference the compiled OCSF artifact used for validation. |
| `profile_resolution_manifest_refs` | Required when `row_mode = ocsf_mapped` | `[]` otherwise | Canonical set of `ProfileResolutionManifest` refs for emitted class and object paths. |
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
| `activation_scope` | Yes | none | `030.ActivationScope`; validated through `050.MappingScopeSelectorContext` before the row may affect output. |
| `row_checksum` | Yes | none | SHA-256 over canonical row bytes after defaults materialize and excluding only `row_checksum`. |
| `lifecycle_status` | Yes | none | Imported from `030.LifecycleStatus`; production mapping requires `active`. |

Unknown fields in `ObservationToOCSFMappingRow` fail before row-set activation. Row-set canonical bytes sort rows by ascending lexical `row_id`. Row-set checksum must include every row checksum, row version, discriminator bytes, row mode, selected source-dataset refs, selected external-schema refs, policy refs, validation refs, activation scope, and lifecycle status.

`row_mode = ocsf_mapped` requires non-null OCSF category, class, activity, type, external schema profile, external schema artifact, profile-resolution manifest refs, enum refs when enum paths emit, base-event policy ref, source-extension rule-set ref or explicit empty rule set, validation fixture refs, activation scope, lifecycle status, and row checksum. `row_mode = cadastre_only` requires null external-schema refs, null OCSF category/class/activity/type values, `required_normalized_paths = []`, `forbidden_normalized_paths = []`, `normalized_fields = {}` by `CadastreOnlyMappingOutputPolicy`, validation fixture refs, activation scope, lifecycle status, and row checksum. `row_mode = deterministically_blocked` requires deterministic error or no-op code, mutation-prohibition refs, validation refs, activation scope, lifecycle status, and row checksum; it must emit no silver output.

### ResolveOCSFMapping algorithm

```text
ResolveOCSFMapping(observation, mapping_bundle, external_schema_profile):
1. Validate ExternalSchemaProfile, ExternalSchemaArtifactRef, ProfileResolutionManifest, ExternalEnumMappingRuleSet, OCSFBaseEventFieldPolicySet, SourceExtensionFieldRuleSet, ObservationToOCSFMappingRowSet, ObservationTypeExternalMappingValidationMatrix, CanonicalValidationOutput, package-set refs when package-supplied, lifecycle transition evidence, and 030.VersionManifest refs through 030.ActivationControlledArtifactRef and 030.ActivationControlledRowRef.
2. Normalize the request execution scope and reject under-scoped requests before row selection.
3. Resolve source_dataset through 020.ResolveSourceDatasetCatalogRow before scope-filtered row selection when any candidate row, profile, source-extension rule, or validation fixture filters by source_dataset.
4. Keep only active ObservationToOCSFMappingRow and profile rows whose activation_scope covers the request through 030.ScopeSelectorCovers, then load rows whose observation_type equals observation.observation_type.
5. Evaluate mapping_discriminator rows in ascending lexical row_id order only to produce deterministic diagnostics; row order must not break ambiguity.
6. If no row matches, emit MAP_OCSF_ROW_MISSING before normalized output.
7. If more than one row matches after discriminator evaluation, emit MAP_OCSF_ROW_AMBIGUOUS before normalized output.
8. If the selected row has row_mode = deterministically_blocked, emit the row's deterministic error or no-op, prove mutation prohibition, and emit no silver output.
9. If the selected row has row_mode = cadastre_only, apply CadastreOnlyMappingOutputPolicy, require selected row refs/checksums in VersionManifest, and emit no OCSF metadata.
10. If the selected row has row_mode = ocsf_mapped, validate category_uid, class_uid, activity_id, activity_name, type_uid, and type_name against the compiled OCSF artifact and active ProfileResolutionManifest refs.
11. Reject any class_uid outside the active class_allowlist with OCSF_CLASS_NOT_ALLOWED.
12. Validate required_normalized_paths and forbidden_normalized_paths.
13. Apply ExternalEnumMappingRule rows; unknown enum values must not invent enum IDs, Other requires compiled support plus active permission, deprecated enum values reject by default, and ID/name sibling mismatch rejects.
14. Apply OCSFBaseEventFieldPolicy rows; disabled base-event fields reject before output and non-authoritative metadata fields must not alter Cadastre authority, identity, temporal, graph, or absence state.
15. Validate every source_extension_fields path against exactly one active SourceExtensionFieldRule; missing, wildcard, reserved-name, missing redaction, secret-scan, or collision failures reject before output.
16. Reject artifact checksum mismatch, profile-set checksum mismatch, extension-set checksum mismatch, class-allowlist checksum mismatch, package-set omission, row-family TODO, validation fixture TODO, and VersionManifest omission before silver output.
17. Emit CadastreSilverObservation only after 040.CadastreSilverObservationSchema passes.
```

Missing discriminator inputs fail before output. Authentication rows must emit `OCSF_ACTIVITY_DISCRIMINATOR_MISSING` when activity cannot be resolved to one compiled activity. Source-action or enum values not present in the compiled artifact must be preserved through declared diagnostics or declared source-extension fields; they must not create OCSF enum IDs.

### StructuredInputRepositoryMappingHandoff

Repository-authored mapping bundles, external schema profiles, source schema imports, semantic overlays, enum rules, base-event policies, source-extension rules, mapping project manifests, and compiler pipeline configurations are inert until exact snapshot validation, materialization, package release handoff when package-supplied, package-set activation when required, and `030.VersionManifest` inclusion pass.

A repository-authored mapping bundle must include `structured_input_repository_snapshot_ref`, `selected_path_refs`, and `file_manifest_checksum`. `ValidateMappingBundle` must reject the bundle when the snapshot ref is missing, stale, checksum-mismatched, mutable, outside the active repository profile, outside allowed path roots, or not covered by a current structured-input validation run.

`CanonicalValidationOutput` for repository-authored mapping validation must include snapshot ref, selected path manifest checksum, mapping project checksum, compiler pipeline checksum, external schema artifact refs, validation row refs, and normalized diagnostics. Pull request approval, branch update, merge, hook success, or current working tree validation must not satisfy `CanonicalValidationOutput`.

Repository-authored mapping validation must also validate `030.StructuredInputRepositoryTemplateContract` when the repository profile requires a template, `030.StructuredInputMaintenanceToolContract` and `030.StructuredInputMaintenanceToolInvocation` when mapping artifacts are generated, `030.StructuredInputRepositoryCIContract` when producer CI evidence is accepted, `100.StructuredInputPublicationManifest` when remote publication is consumed, `030.StructuredInputCandidateSyncRecord` when manifest sync imported the candidate, and `030.StructuredInputRepositoryGroup` when mappings depend on artifacts from another repository.

Template conformance, SDK/CLI diagnostics, generated artifact existence, producer CI success, publication manifest claims, and sync records must not select an OCSF mapping row, authorize a source-extension field, satisfy `CanonicalValidationOutput`, or activate a mapping bundle by themselves.

### MappingProjectManifest schema

`MappingProjectManifest` is the stable mapping-authoring project interface. Concrete manifest rows are activation-controlled artifacts and may be repository-authored only through `030.StructuredInputRepositorySnapshot`.

| Field | Required | Default or omission behavior | Rule |
| --- | ---: | --- | --- |
| `mapping_project_manifest_id` | Yes | none | Stable ID over canonical project bytes, snapshot refs, compiler options, dependency refs, plugin refs, and validation refs. |
| `project_root_ref` | Yes | none | Normalized selected path ref inside the validated repository snapshot or immutable package artifact. |
| `structured_input_repository_snapshot_ref` | Required when repository-authored | null only when the manifest is not repository-authored | Must match exact snapshot used for validation and materialization. |
| `repository_template_contract_ref` | Required when repository-authored and template required | null only when no template is required | Exact `030.StructuredInputRepositoryTemplateContract` ref. |
| `maintenance_tool_contract_refs` | Required when tool output affects mapping artifacts | `[]` only when no tool output affects validation or generated artifacts | Exact `030.StructuredInputMaintenanceToolContract` refs. |
| `maintenance_tool_invocation_refs` | Required when generated outputs are consumed | `[]` only when no generated output is consumed | Invocation refs must match tool contract, snapshot, input checksum, generated output checksum, and materialization result. |
| `generated_artifact_manifest_checksum` | Required when generated outputs exist | null only when no generated output exists | SHA-256 over canonical generated artifact manifest bytes. |
| `publication_manifest_ref` | Required when published by a remote maintenance repository | null otherwise | Exact `100.StructuredInputPublicationManifest` ref verified by `100`. |
| `candidate_sync_record_ref` | Required when imported by manifest sync | null otherwise | Sync ref is discovery/audit evidence only. |
| `declared_generated_output_roots` | No | `[]` | Generated outputs outside declared roots fail validation. |
| `generated_output_checksum` | Required when generated outputs exist | null otherwise | Byte-stable for the same snapshot, tool contract, compiler options, and dependency locks. |
| `template_validation_refs` | Required when template required | `[]` otherwise | Non-empty refs to template validation rows. |
| `producer_ci_validation_refs` | Required when producer CI evidence is accepted | `[]` otherwise | Non-empty refs bound to exact snapshot and file manifest checksum. |
| `source_roots` | Yes | none | Non-empty normalized paths under `project_root_ref`; undeclared source roots and path escapes fail. |
| `dependency_lock_refs` | Required when dependencies affect output or validation | `[]` only when no dependencies affect output or validation | Mutable dependency refs are forbidden. |
| `plugin_refs` | No | `[]` | Plugins must be package-supplied or activation-controlled and validation-covered. |
| `compiler_options` | Yes | `{}` only when compiler default set is explicitly checksummed | User-local config and undeclared workstation defaults are forbidden. |
| `generated_output_policy` | Yes | none | Declares generated artifact classes and forbidden output classes. |
| `artifact_class_outputs` | Yes | none | Closed artifact-class outputs expected from this project. |
| `private_binding_policy_ref` | Yes | none | Ref to redaction/private-binding rule set. |
| `canonical_project_checksum` | Yes | none | SHA-256 over normalized project inputs after defaults materialize. |
| `validation_refs` | Yes | none | Non-empty refs covering exact snapshot, mappings, private-binding rejection, and no-authority behavior. |
| `activation_scope` | Yes | none | Vendor-neutral scope. |
| `lifecycle_status` | Yes | none | Production use requires `active`. |

`ValidateMappingBundle` must run repository snapshot validation before compiling. It must reject branch names, tags, uncommitted working copies, mutable refs, user-local configuration, undeclared source roots, duplicate normalized paths, path escapes, core overrides, undeclared source-extension fields, ambiguous rows, missing rows, forbidden OCSF fields, and enum ambiguity before silver output.

Generated outputs must appear only under `declared_generated_output_roots`. For the same repository snapshot, tool contract, compiler options, dependency locks, and input checksums, generated mapping artifact bytes and `generated_output_checksum` must be byte-identical. SDK/CLI diagnostics must not substitute for `CanonicalValidationOutput`. Publication manifest claims must be verified by `100` before package release handoff or activation.

| Error code | Required use |
| --- | --- |
| `MAPPING_REPOSITORY_SNAPSHOT_MISSING` | Repository-authored mapping input lacks exact snapshot ref, selected path refs, or file manifest checksum. |
| `MAPPING_REPOSITORY_SNAPSHOT_MISMATCH` | Mapping validation, project manifest, compiler output, or materialization refs do not match the exact repository snapshot. |
| `MAPPING_REPOSITORY_PATH_FORBIDDEN` | Mapping project path is outside selected roots, invalid after normalization, duplicate, non-regular, or a path escape. |
| `MAPPING_VALIDATION_RUN_STALE` | Mapping validation run is not for the exact snapshot, path manifest, project checksum, compiler checksum, or schema artifact refs. |

### MVP OCSF Mapping Catalog Closure

MVP OCSF-aligned silver output requires a supporting catalog closure. The closure catalog is activation-controlled material and must not be inferred from research, examples, package labels, source prose, or missing rows.

| Required closure member | Requirement |
| --- | --- |
| `ExternalSchemaProfile` | Exactly one active OCSF `1.8.0` profile, or one deterministic block row that prevents OCSF-aligned silver output. |
| `ExternalSchemaArtifactRef` | Exactly one active ref containing source tag, source commit, compiler ID/version/checksum, validator ID/version/checksum, compiled artifact checksum, profile set checksum, extension set checksum, and class allowlist checksum. |
| observation mapping rows | Exactly one active `ObservationToOCSFMappingRow` or exactly one explicit `cadastre_only` row for every MVP observation family. Missing rows must not infer `cadastre_only`. |
| policy row sets | Active enum mapping, base-event field policy, source-extension field, profile-resolution manifest, and observation-type validation row sets. |
| validation bytes | Concrete non-`TODO` fixture checksums and expected output or expected error checksums for every active row. |
| authority handoff | No external schema authority effect is allowed without `060.ExternalSchemaAuthoritySignalMappingRowSet`. |

The required mapping closure status for each MVP observation family is exhaustive. The table declares the only row mode that may become active for each discriminator state; rows with `TODO:` fixture bytes or expected checksums remain blocked.

| Observation family and discriminator state | Required closure state | Required external target or block behavior | Production status until concrete refs exist |
| --- | --- | --- | --- |
| `inventory_observation` | `ocsf_mapped` | OCSF Device Inventory Info `5001`. | blocked until mapping row, compiled artifact, validation refs, and expected output checksum exist. |
| `software_inventory_observation` | `ocsf_mapped` | OCSF Software Inventory Info `5020`. | blocked until mapping row, profile-resolution refs, validation refs, and expected output checksum exist. |
| `vulnerability_finding_observation` | `ocsf_mapped` | OCSF Vulnerability Finding `2002`. | blocked until source lifecycle discriminator rows and validation refs exist. |
| `authentication_observation` with exact compiled activity discriminator | `ocsf_mapped` | OCSF Authentication `3002` with no default activity. | blocked for any missing or ambiguous activity discriminator. |
| `dns_observation` | `ocsf_mapped` | OCSF DNS Activity `4003`. | blocked until endpoint-order non-authority fixtures exist. |
| `dhcp_ipam_observation` where `assignment_source = dhcp` | `ocsf_mapped` | OCSF DHCP Activity `4004`. | blocked until DHCP discriminator fixtures exist. |
| `dhcp_ipam_observation` where `assignment_source = ipam` | `cadastre_only` or `deterministically_blocked` | Must not map to OCSF DHCP Activity. | blocked unless an explicit `cadastre_only` row or block row exists. |
| `network_activity_observation` | `ocsf_mapped` | OCSF Network Activity `4001`. | blocked until flow-role handoff and endpoint-order rejection fixtures exist. |
| every other MVP observation family | `cadastre_only` or `deterministically_blocked` | No OCSF class by inference. | blocked unless an explicit row exists. |

A row with a `TODO` checksum, missing validation ref, ambiguous row match, invalid activity discriminator, unknown enum without a permitted rule, undeclared source-extension field, forbidden base-event field authority, or compiled artifact checksum mismatch remains `blocked_validation` and must not affect production silver output.

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
1. Validate repository snapshot and selected paths through `030.ValidateStructuredInputRepositorySnapshot` when any input is repository-authored.
2. Validate every mapping-related artifact ref through `030.ActivationControlledArtifactRef`.
3. Verify package-set inclusion when artifacts are package-supplied.
4. Canonicalize source roots and dependency lock refs.
5. Reject undeclared source roots, path escapes, mutable dependency refs, uncommitted working copies, and user-local config.
6. Resolve source schema import profiles and semantic overlays.
7. Compile mappings in declared phase order.
8. Validate external schema profile refs and compiled artifact checksums.
9. Reject any artifact that attempts to define a core field, core omission state, identity rule, temporal rule, graph rule, or source-authority rule.
10. Run MappingValidationRule rows with deterministic severities.
11. Run observation-type validation matrix cases.
12. Emit CanonicalValidationOutput with deterministic diagnostics, snapshot refs when applicable, selected path manifest checksum, mapping project checksum, compiler pipeline checksum, and validation row refs.
13. Include all output-affecting mapping artifact refs and structured-input refs in `VersionManifest` before silver output.
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

### ValidateOCSFMappingActivation algorithm

```text
ValidateOCSFMappingActivation(mapping_row_set, external_schema_profile, artifact_refs, validation_matrix, version_manifest):
1. Validate every required artifact ref through `030.ActivationControlledArtifactRef`.
2. Validate lifecycle status `active` for the selected profile, artifact ref, profile-resolution manifests, enum rule set, base-event policy set, source-extension rule set, mapping row set, and validation matrix.
3. Validate the compiled artifact checksum, class allowlist checksum, profile set, extension set, compiler evidence, and validator evidence.
4. Validate that every active MVP observation family resolves to at least one required row family and that every emitted observation fixture resolves to exactly one row.
5. Validate that each active OCSF row has exact category UID, class UID, activity ID, type UID, required paths, forbidden paths, enum refs, base-event policy refs, source-extension refs, fixture refs, activation scope, lifecycle status, and manifest refs.
6. Validate that `cadastre_only` rows have null external schema values and that no missing row defaults to `cadastre_only`.
7. Validate that every enum path used by any active row has exactly one active enum mapping rule.
8. Validate that every source-extension path emitted by any success fixture has exactly one active source-extension rule.
9. Validate that positive, negative, non-authority, and direction fixtures have non-`TODO` fixture checksums and expected output or expected error checksums.
10. Reject activation with the most specific owner error when any condition fails.
```

`ValidateOCSFMappingActivation` must run before any mapping row set, external schema profile, enum rule set, base-event policy set, source-extension rule set, or observation-type validation matrix can enter production-active scope. A missing catalog path for concrete row instances remains an owner `TODO:` blocker and must not be inferred from stable prose.

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

### ProfileResolutionManifest schema

`ProfileResolutionManifest` records the compiled OCSF profile, inheritance, object-path, constraint, and enum resolution that governs emitted normalized fields. Concrete manifests are activation-controlled artifacts and must be checksummed over canonical manifest bytes.

| Field | Required behavior |
| --- | --- |
| `manifest_id` | Stable ID. |
| `external_schema_artifact_ref` | Exact active `ExternalSchemaArtifactRef`. |
| `schema_name` / `schema_version` | Must match the artifact ref. |
| `profile_set` / `extension_set` | Must match the compiled artifact and active profile. |
| `resolved_class_uid` | Required for event-class output. |
| `resolved_object_path` | Required for every emitted object path. |
| `inherited_field_set` | Canonically sorted resolved inherited fields. |
| `required_field_set` | Canonically sorted required fields. |
| `recommended_field_set` | Canonically sorted recommended fields. |
| `constraint_set` | Canonically sorted compiled constraints. |
| `enum_ref_set` | Canonically sorted enum refs used by mapping. |
| `deprecated_field_set` | Canonically sorted deprecated fields and waiver refs. |
| `manifest_checksum` | SHA-256 over canonical manifest bytes. |
| `validation_refs` | Non-empty. |
| `activation_scope` | `030.ActivationScope`; validated through `050.MappingScopeSelectorContext` before the row may affect output. |
| `lifecycle_status` | Production use requires `active`. |

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

`OCSFBaseEventFieldPolicySet` is an activation-controlled artifact. It instantiates field-path policy rows for OCSF base-event fields and must not grant Cadastre source authority.

| Field | Required behavior |
| --- | --- |
| `policy_set_id` | Stable row-set ID. |
| `policy_id` | Stable row ID. |
| `external_schema_profile_ref` | Exact active profile. |
| `field_path` | Exact OCSF base-event path. |
| `policy_class` | Closed enum: `disabled`, `non_authoritative_metadata`, `allowed_metadata_bounded`, `rejected_deprecated`, `waived_deprecated`. |
| `default_behavior` | Required. |
| `redaction_policy_ref` | Required when value may persist or display. |
| `authority_effect` | Must be `none`; any other value fails activation. |
| `validation_refs` | Non-empty. |
| `activation_scope` | `030.ActivationScope`; validated through `050.MappingScopeSelectorContext` before the row may affect output. |
| `lifecycle_status` | Production use requires `active`. |

`raw_data`, `raw_data_hash`, and `unmapped` are disabled by default. Observables, enrichments, severity, status, and confidence are non-authoritative by default. Any active policy that permits bounded metadata persistence must include redaction refs and validation fixtures proving no raw evidence, omission, identity, gold, graph, source-authority, or confidence-policy substitution.

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

`ExternalEnumMappingRuleSet` is an activation-controlled artifact. It contains total enum mapping rows for each enum path used by an active OCSF mapping row.

| Field | Required behavior |
| --- | --- |
| `rule_set_id` | Stable row-set ID. |
| `rule_id` | Stable row ID. |
| `external_schema_profile_ref` | Exact active profile. |
| `class_uid` / `object_path` | Exact compiled class or object context. |
| `enum_field_path` | Exact field path; wildcards are forbidden. |
| `compiled_enum_ref` | Exact compiled enum ID/name domain. |
| `source_value_selector` | Canonical selector over normalized parse fields or mapping inputs. |
| `known_value_map` | Total map for known source values claimed by the row. |
| `unknown_value_behavior` | Closed enum: `reject`, `emit_other_with_raw_preservation`, `cadastre_only`. |
| `other_value_policy` | Required when `Other` may be emitted. |
| `raw_value_preservation_path` | Required for `Other` or unknown diagnostics. |
| `deprecated_enum_behavior` | Default `reject`; waiver ref required otherwise. |
| `validation_refs` | Non-empty. |
| `activation_scope` | `030.ActivationScope`; validated through `050.MappingScopeSelectorContext` before the row may affect output. |
| `lifecycle_status` | Production use requires `active`. |

Unknown enum values must not invent enum IDs. `Other` may be emitted only when the compiled enum contains `Other`, the active rule permits `emit_other_with_raw_preservation`, and `raw_value_preservation_path` names a declared diagnostic or source-extension path permitted by the active source-extension policy.

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
| `artifact_refs` | Yes | Canonically sorted refs to mapping artifacts, external schema profiles, compiled schema artifacts, profile-resolution manifests, enum rule sets, base-event policy sets, source-extension rule sets, validation outputs, mapping row sets, fixtures, or version manifests consulted by the error; empty only when no artifact was consulted. |
| `mapping_artifact_ref` | No | Required when a mapping artifact was consulted. |
| `external_schema_profile_ref` | No | Required for OCSF or external schema failures. |
| `source_extension_rule_ref` | No | Required for source-extension failures. |
| `raw_value_ref` | No | Redacted ref or checksum only; raw source values must not be stored in caller context. |
| `validation_refs` | Yes | Exact `120` mapping fixture refs. |
| `redaction_classes` | Yes | Raw payload bytes, raw source values, credentials, private bindings, source-native identity values, and raw OCSF payload bytes must map to `always_forbidden`. |
| `blocking_reason` | Yes when generated row severity is `blocked` | Bounded reason; otherwise null or omitted. |

### MappingErrorNameCanonicalization

`050` is the sole owner for mapping and external-schema registry codes. The canonical codes are `MAP_OCSF_ROW_MISSING`, `MAP_OCSF_ROW_AMBIGUOUS`, and `OCSF_ARTIFACT_MISMATCH`. The aliases `OCSF_MAPPING_ROW_MISSING`, `OCSF_MAPPING_ROW_AMBIGUOUS`, and `OCSF_COMPILED_ARTIFACT_CHECKSUM_MISMATCH` are invalid input to `110.GenerateErrorCodeRegistry` and must fail validation before API, export, health, audit, or validation-visible output.

A mapping implementation must not emit both a canonical code and an alias for the same failure. Alias rejection uses `110.SharedRegistryAliasRejectionTable` and the validation rows in `120.ErrorCodeRegistryValidationMatrix`.

### ActivationControlledRowSchemaPrecisionHandoff

The following `050` row families can affect parsing, normalization, external schema validation, OCSF row selection, source-extension output, mapping validation, compiler output, projection loss, or canonical validation output. Each production-affecting row family must instantiate `030.ActivationControlledRowField` columns exactly. A selected row family with a missing concrete row, missing checksum, missing package-set ref when package-supplied, missing validation ref, missing `VersionManifest` ref, or `TODO:` fixture value is `blocked_validation` and must not emit production silver output.

#### OCSFClosureActivationControlledRowFieldTable

The table below is the closed `030.ActivationControlledRowField` table for OCSF and mapping closure members introduced by this patch. `field_path` prefixes name the row family. Owner-local row tables above may provide additional prose, but the behavior below governs missing, null, omission, checksum, and manifest handling for production activation.

| field_path | type | required | default | null_allowed | omit_allowed | bounds | array_semantics | duplicate_policy | canonical_sort_key | id_input | checksum_input | extension_policy | redaction_owner | version_manifest_requirement | missing_error | invalid_error |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `*.row_id` | `040.ScalarType.string` | yes | none | no | no | 1..256 Unicode scalar values | n/a | reject | n/a | ordered:1 | yes | closed | `110` | selected row ref and row checksum | `ACTIVATION_ROW_REQUIRED_FIELD_MISSING` | `ACTIVATION_ROW_FIELD_TYPE_INVALID` |
| `*.row_version` | `040.ScalarType.string` | yes | none | no | no | 1..64 Unicode scalar values | n/a | reject | n/a | ordered:2 | yes | closed | `110` | row schema version | `ACTIVATION_ROW_REQUIRED_FIELD_MISSING` | `ACTIVATION_ROW_FIELD_TYPE_INVALID` |
| `*.validation_refs` | `array<030.ActivationControlledRowRef>` | yes | none | no | no | 1..1024 refs for production rows | canonical_set | reject | `row_family,row_id` | no | yes | closed | `110` | every validation ref | `ACTIVATION_ROW_REQUIRED_FIELD_MISSING` | `ACTIVATION_ROW_REF_INVALID` |
| `*.activation_scope` | `030.ActivationScope` | yes | none | no | no | scope context declared by `050.MappingScopeSelectorContext` | n/a | reject | n/a | no | yes | closed | `110` | selector context and selector checksum | `ACTIVATION_ROW_REQUIRED_FIELD_MISSING` | `SCOPE_SELECTOR_INVALID` |
| `*.lifecycle_status` | `030.LifecycleStatus` | yes | none | no | no | production requires `active` | n/a | reject | n/a | no | yes | closed | `110` | lifecycle transition evidence refs | `ACTIVATION_ROW_REQUIRED_FIELD_MISSING` | `ACTIVATION_ROW_LIFECYCLE_INVALID` |
| `*.row_checksum` | `040.ScalarType.sha256_hex` | yes | derived:`040.CanonicalJSON` | no | no | SHA-256 hex | n/a | reject | n/a | derived | no | closed | `110` | row checksum | `ACTIVATION_ROW_CHECKSUM_MISSING` | `ACTIVATION_ROW_CHECKSUM_MISMATCH` |
| `ExternalSchemaProfile.schema_artifact_ref` | `030.ActivationControlledArtifactRef` | yes | none | no | no | artifact_class `external_schema_artifact_ref` | n/a | reject | n/a | ordered:3 | yes | closed | `110` | artifact ref and checksum | `OCSF_COMPILED_ARTIFACT_REF_MISSING` | `OCSF_ARTIFACT_MISMATCH` |
| `ExternalSchemaProfile.profile_resolution_manifest_refs` | `array<030.ActivationControlledArtifactRef>` | yes | none | no | no | 1..1024 refs for OCSF rows | canonical_set | reject | artifact ref | no | yes | closed | `110` | every manifest ref and checksum | `PROFILE_RESOLUTION_MANIFEST_MISSING` | `PROFILE_RESOLUTION_MANIFEST_INVALID` |
| `ExternalSchemaProfile.class_allowlist_checksum` | `040.ScalarType.sha256_hex` | yes | none | no | no | SHA-256 hex | n/a | reject | n/a | no | yes | closed | `110` | class allowlist checksum | `OCSF_CLASS_ALLOWLIST_CHECKSUM_MISSING` | `OCSF_CLASS_NOT_ALLOWED` |
| `ExternalSchemaArtifactRef.source_tag` | `040.ScalarType.string` | yes | `1.8.0` for MVP candidate rows | no | no | exact `1.8.0` for production MVP | n/a | reject | n/a | ordered:3 | yes | closed | `110` | artifact ref | `OCSF_SOURCE_TAG_MISSING` | `OCSF_ARTIFACT_MISMATCH` |
| `ExternalSchemaArtifactRef.source_commit` | `040.ScalarType.string` | yes | `6fa6499a0f8c9f449d342816e90e5f687c224b0a` for MVP candidate rows | no | no | 40 lowercase hex chars | n/a | reject | n/a | ordered:4 | yes | closed | `110` | artifact ref | `OCSF_SOURCE_COMMIT_MISSING` | `OCSF_ARTIFACT_MISMATCH` |
| `ExternalSchemaArtifactRef.compiled_artifact_checksum` | `040.ScalarType.sha256_hex` | yes | none | no | no | concrete non-`TODO` SHA-256 | n/a | reject | n/a | ordered:5 | yes | closed | `110` | compiled artifact checksum | `OCSF_COMPILED_ARTIFACT_CHECKSUM_MISSING` | `OCSF_ARTIFACT_MISMATCH` |
| `ExternalSchemaArtifactRef.compiler_ref` | `030.ActivationControlledArtifactRef` | yes | none | no | no | compiler ID/version/checksum concrete | n/a | reject | n/a | no | yes | closed | `110` | compiler ref and checksum | `OCSF_COMPILER_REF_MISSING` | `OCSF_ARTIFACT_MISMATCH` |
| `ExternalSchemaArtifactRef.validator_ref` | `030.ActivationControlledArtifactRef` | yes | none | no | no | validator ID/version/checksum concrete | n/a | reject | n/a | no | yes | closed | `110` | validator ref and checksum | `OCSF_VALIDATOR_REF_MISSING` | `OCSF_ARTIFACT_MISMATCH` |
| `ProfileResolutionManifest.required_field_set` | `array<040.ScalarType.field_path>` | yes | none | no | no | 0..4096 paths, exact compiled paths | canonical_set | reject | field path | no | yes | closed | `110` | manifest ref and checksum | `PROFILE_RESOLUTION_MANIFEST_MISSING` | `PROFILE_RESOLUTION_MANIFEST_INVALID` |
| `ObservationToOCSFMappingRow.row_mode` | owner enum | yes | none | no | no | `ocsf_mapped`, `cadastre_only`, `deterministically_blocked` | n/a | reject | n/a | ordered:3 | yes | closed | `110` | selected row ref and checksum | `MAP_OCSF_ROW_MISSING` | `MAP_OCSF_ROW_INVALID` |
| `ObservationToOCSFMappingRow.mapping_discriminator` | owner predicate bytes | yes | none | no | no | canonical predicate, max 65536 bytes | n/a | reject | n/a | ordered:4 | yes | closed | `110` | selected row ref and checksum | `OCSF_ACTIVITY_DISCRIMINATOR_MISSING` | `MAP_OCSF_ROW_INVALID` |
| `ObservationToOCSFMappingRow.ocsf_class_uid` | `040.ScalarType.uint64` | conditional:`row_mode = ocsf_mapped` | null otherwise | yes only when non-OCSF row mode | no | must be in active allowlist | n/a | reject | n/a | ordered:5 | yes | closed | `110` | selected row ref and class allowlist checksum | `MAP_OCSF_ROW_MISSING` | `OCSF_CLASS_NOT_ALLOWED` |
| `ObservationToOCSFMappingRow.required_normalized_paths` | `array<040.ScalarType.field_path>` | yes | `[]` for `cadastre_only` | no | no | 0..4096 exact paths | canonical_set | reject | field path | no | yes | closed | `110` | selected row ref and checksum | `MAP_REQUIRED_PATHS_MISSING` | `MAP_REQUIRED_PATH_INVALID` |
| `ObservationToOCSFMappingRow.forbidden_normalized_paths` | `array<040.ScalarType.field_path>` | no | `[]` | no | no | 0..4096 exact paths | canonical_set | reject | field path | no | yes | closed | `110` | selected row ref and checksum | none | `MAP_FORBIDDEN_PATH_EMITTED` |
| `ExternalEnumMappingRule.unknown_value_behavior` | owner enum | yes | `reject` | no | no | `reject`, `emit_other_with_raw_preservation`, `cadastre_only` | n/a | reject | n/a | no | yes | closed | `110` | enum rule ref and checksum | `EXTERNAL_ENUM_RULE_MISSING` | `EXTERNAL_ENUM_UNKNOWN` |
| `ExternalEnumMappingRule.known_value_map` | `map<string,canonical_object>` | yes | none | no | no | 0..4096 entries; total for claimed source values | canonical_set | reject | key lexical | no | yes | closed | `110` | enum rule ref and checksum | `EXTERNAL_ENUM_RULE_MISSING` | `EXTERNAL_ENUM_SIBLING_MISMATCH` |
| `OCSFBaseEventFieldPolicy.field_path` | `040.ScalarType.field_path` | yes | none | no | no | exact OCSF base-event path | n/a | reject | n/a | ordered:3 | yes | closed | `110` | policy row ref and checksum | `OCSF_BASE_FIELD_POLICY_MISSING` | `OCSF_BASE_FIELD_FORBIDDEN` |
| `OCSFBaseEventFieldPolicy.policy_class` | owner enum | yes | none | no | no | `disabled`, `non_authoritative_metadata`, `allowed_metadata_bounded`, `rejected_deprecated`, `waived_deprecated` | n/a | reject | n/a | ordered:4 | yes | closed | `110` | policy row ref and checksum | `OCSF_BASE_FIELD_POLICY_MISSING` | `OCSF_BASE_FIELD_FORBIDDEN` |
| `SourceExtensionFieldRule.field_path` | `040.ScalarType.field_path` | yes | none | no | no | `source.<source_category>.<field_name>`, max 512 | n/a | reject | n/a | ordered:3 | yes | closed | `110` | source-extension row ref and checksum | `SOURCE_EXTENSION_FIELD_UNDECLARED` | `SOURCE_EXTENSION_NAMESPACE_INVALID` |
| `SourceExtensionFieldRule.bounds` | `040.ScalarType.canonical_object` | yes | none | no | no | string 4096 chars, arrays 1024 elements, maps 4096 entries, object 65536 bytes unless narrowed | n/a | reject | n/a | no | yes | closed | `110` | source-extension row ref and checksum | `SOURCE_EXTENSION_RULE_INCOMPLETE` | `SOURCE_EXTENSION_BOUNDS_INVALID` |
| `ObservationTypeExternalMappingValidationMatrix.validation_row_id` | `040.ScalarType.string` | yes | none | no | no | exact row ID from `120` | n/a | reject | n/a | ordered:3 | yes | closed | `110` | validation row ref and checksum | `OCSF_VALIDATION_ROW_MISSING` | `OCSF_VALIDATION_ROW_INVALID` |
| `CanonicalValidationOutput.normalized_validation_output_checksum` | `040.ScalarType.sha256_hex` | yes | none | no | no | concrete non-`TODO` SHA-256 | n/a | reject | n/a | ordered:3 | yes | closed | `110` | validation output checksum | `OCSF_CANONICAL_VALIDATION_OUTPUT_INCOMPLETE` | `OCSF_CANONICAL_VALIDATION_OUTPUT_MISMATCH` |

#### OCSFClosureRowFamilyStatus

| row_family | production classification | precision status |
| --- | --- | --- |
| `ExternalSchemaProfile` | output_affecting | `full_row_schema` through `ExternalSchemaProfile row schema`, `MVP OCSF Production Baseline Decision`, and `OCSFClosureActivationControlledRowFieldTable`. |
| `ExternalSchemaArtifactRef` | output_affecting | `full_row_schema`; production remains blocked until concrete compiler, validator, compiled artifact, profile set, extension set, and class allowlist checksums exist. |
| `ProfileResolutionManifest` | output_affecting | `full_row_schema`; arrays are canonical sets with duplicate rejection and manifest refs. |
| `ObservationToOCSFMappingRow` | output_affecting | `full_row_schema`; row mode controls conditional required/null/forbidden fields. |
| `ExternalEnumMappingRule` | output_affecting | `full_row_schema`; unknown enum default is reject, `Other` requires compiled support plus raw preservation, deprecated enum default is reject. |
| `OCSFBaseEventFieldPolicy` | output_affecting | `full_row_schema`; `raw_data`, `raw_data_hash`, and `unmapped` default disabled; observables, enrichments, severity, status, and confidence remain non-authoritative. |
| `OCSFProfileUpgradeReport` | output_affecting for profile replacement | `full_row_schema`; replacement requires schema diff, replay, shadow, class allowlist, enum, deprecated-field, profile, extension, and golden-corpus refs. |
| `SourceExtensionFieldRule` | output_affecting | `full_row_schema`; missing or empty rule set emits no fields. |
| `SourceSchemaImportProfile` | output_affecting when importer output affects mapping | `blocked_validation` until this owner adds the same `030.ActivationControlledRowField` table for importer-specific fields. |
| `SemanticOverlayArtifact` | validation or authoring unless activated | `validation_only` unless a later owner patch promotes it with full row precision. |
| `MappingProjectManifest` | output_affecting for mapping validation | `full_row_schema` through its schema and structured-input repository handoff. |
| `MappingCompilerPipeline` | output_affecting for validation output | `blocked_validation` until concrete phase-order, diagnostics, toolchain, and checksum rows exist. |
| `MappingValidationRule` | output_affecting for promotion and diagnostics | `blocked_validation` until concrete rule rows and expected diagnostics exist. |
| `ObservationTypeExternalMappingValidationMatrix` | output_affecting for mapping activation | `full_row_schema` with blocked validation rows in `120` until fixture bytes are supplied. |
| `CanonicalValidationOutput` | output_affecting for promotion and replay | `full_row_schema`; `TODO:` checksum values block production. |

`profile_set`, `extension_set`, `profile_resolution_manifest_refs`, `enum_rule_refs`, `required_normalized_paths`, `forbidden_normalized_paths`, and `source_extension_rule_set_ref` use `canonical_set`, duplicate rejection, lexical or ref sort keys, checksum participation, owner error mapping, and `VersionManifest` requirements. `time`, diagnostic, and generated validation arrays use `ordered_sequence` only when this spec names the ordering rule.

A mapping bundle must fail before silver output when any selected `050` row family is `blocked_validation`, uses a bare string ref, accepts an undeclared extension path, omits row refs or row checksums from `030.VersionManifest`, or contains `TODO:` in production-affecting field type, bound, checksum, fixture checksum, expected output, expected error, or mutation-prohibition proof.

### Acceptance Criteria

| ID | Criterion |
| --- | --- |
| `050-SOURCE-DATASET-CATALOG-AC-001` | Mapping activation resolves `source_dataset` through `020.ResolveSourceDatasetCatalogRow` before scope-filtered row selection. |
| `050-SOURCE-DATASET-CATALOG-AC-002` | Missing, ambiguous, inactive, checksum-mismatched, private-leaking, or deterministically blocked source-dataset rows emit no authority, absence, cleanup, retraction, graph expiry, control pass/fail, or watermark effect. |
| `050-EXTERNAL-SCHEMA-NONAUTH-AC-001` | OCSF field absence, endpoint order, status, severity, confidence, observables, enrichments, `raw_data`, `unmapped`, and external enum values cannot become source-effect authority without exact `060` closure rows. |
| `050-CLEANUP-AC-001` | No banned reference class remains. |
| `050-CLEANUP-AC-002` | `normalized_fields` remains governed by active OCSF profile unless an observation type is declared `cadastre_only`. |
| `050-CLEANUP-AC-003` | OCSF raw, unmapped, observable, enrichment, status, severity, and confidence fields remain non-authoritative unless explicitly governed by policy. |
| `050-SCHEMA-PATCH-AC-001` | Every normalized output validates against `040.CadastreSilverObservationSchema`. |
| `050-SCHEMA-PATCH-AC-002` | Undeclared `source_extension_fields` fail before production output. |
| `050-SCHEMA-PATCH-AC-003` | `cadastre_only` observations are the only silver observations allowed to have null `external_schema_profile_id`. |
| `050-SCHEMA-PATCH-AC-004` | OCSF fields cannot create identity, gold, graph, authority, or omission semantics outside the owning specs. |
| `050-CLEANUP-AC-004` | Mapping validation remains deterministic and byte-stable through `CanonicalValidationOutput`. |
| `050-OCSF-BASELINE-AC-001` | Active production output uses compiled OCSF `1.8.0` for the MVP profile; OCSF `main`, development versions, and main-only fields fail production activation before silver output. |
| `050-SOURCE-EXTENSION-AC-001` | Undeclared, namespace-invalid, unbounded, secret-scan-failing, or OCSF-reserved-colliding source extension fields fail before production output. |
| `050-VOLATILITY-AC-001` | Every production mapping-related artifact validates through `030.ActivationControlledArtifactRef` before silver output. |
| `050-VOLATILITY-AC-002` | OCSF artifact checksum mismatch, inactive enum rule sets, omitted source-extension row refs, and mapping core overrides fail before production output. |
| `050-VOLATILITY-AC-003` | `cadastre_only` mapping with null schema profile is allowed only by an active mapping row. |
| `050-OCSF-MAP-AC-001` | Every active MVP mapping fixture resolves to exactly one `ObservationToOCSFMappingRow` or fails before silver output with `MAP_OCSF_ROW_MISSING` or `MAP_OCSF_ROW_AMBIGUOUS`. |
| `050-OCSF-MAP-AC-002` | Every active row's category UID, class UID, activity ID, type UID, required object paths, and forbidden paths match the compiled OCSF `1.8.0` artifact and are enforced before silver output. |
| `050-OCSF-MAP-AC-003` | The active class allowlist contains exactly the class UIDs used by active mapping rows. |
| `050-OCSF-MAP-AC-004` | Unknown enum and source-action values do not invent OCSF enum IDs; `Other` enum use requires compiled support, active permission, and raw-value preservation. |
| `050-OCSF-MAP-AC-005` | Authentication has no default activity. |
| `050-OCSF-MAP-AC-006` | IPAM-only assignment rows do not map to OCSF DHCP Activity. |
| `050-EXTERNAL-SCHEMA-AUTHORITY-HANDOFF-AC-001` | A mapping bundle that attempts to use OCSF class, status, severity, confidence, observables, enrichments, endpoint order, field presence, or field absence as authority without an active `060.ExternalSchemaAuthoritySignalMappingRow` fails before authority, absence, cleanup, graph expiry, retraction, control-state, or watermark effect. |
| `050-EXTERNAL-SCHEMA-AUTHORITY-HANDOFF-AC-002` | Valid normalized metadata remains persistable when the authority effect is blocked, provided the mapping itself is valid and emits no forbidden authority output. |
| `050-OCSF-MAP-AC-007` | `Other` enum output is permitted only when the compiled enum contains `Other`, the active enum rule permits it, and raw-value preservation is declared. |
| `050-SOURCE-EXT-AC-001` | Every emitted `source_extension_fields` path has exactly one active `SourceExtensionFieldRule`; zero or multiple matching rules fail before silver output. |
| `050-SOURCE-EXT-AC-002` | Missing or empty `SourceExtensionFieldRuleSet` permits no `source_extension_fields` output. |
| `050-SOURCE-EXT-AC-003` | Wildcards, namespace errors, reserved-name collisions, missing redaction, and secret-scan failures reject before output. |
| `050-BASE-FIELD-AC-001` | Disabled base-event fields, including `raw_data`, `raw_data_hash`, and `unmapped`, fail before output unless an active bounded policy permits non-authoritative metadata persistence with required redaction refs. |
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

### Structured input mapping repository errors

| Error code | Required use |
| --- | --- |
| `MAPPING_REPOSITORY_TEMPLATE_MISMATCH` | Mapping project layout, source roots, generated output roots, or artifact class outputs do not match the active repository template contract. |
| `MAPPING_REPOSITORY_TOOL_OUTPUT_MISMATCH` | Generated artifact bytes, generated artifact manifest checksum, tool invocation checksum, or canonical validation output checksum does not match the tool contract. |
| `MAPPING_REPOSITORY_CI_STALE` | Producer CI validation is stale or not exact-snapshot-bound. |
| `MAPPING_REPOSITORY_PUBLICATION_MANIFEST_MISMATCH` | Publication manifest digest, package type, validation refs, materialization refs, or compatibility claims do not match mapping artifacts. |
| `MAPPING_REPOSITORY_SYNC_NONAUTHORITY` | Candidate sync record is used to activate or select mapping behavior. |

### Structured input mapping acceptance criteria

| ID | Criterion |
| --- | --- |
| `050-STRUCTURED-INPUT-MAPPING-AC-001` | Valid repository-authored mapping emits deterministic `CanonicalValidationOutput` containing exact snapshot refs, selected path manifest checksum, mapping project checksum, compiler pipeline checksum, external schema refs, validation refs, and output checksum. |
| `050-STRUCTURED-INPUT-MAPPING-AC-002` | Merge, PR approval, branch update, hook success, or validation over current working tree emits no production silver output without materialized package-set activation when package-supplied. |
| `050-STRUCTURED-INPUT-MAPPING-AC-003` | Branch, tag, current working tree, stale validation, missing snapshot ref, and path escape fail before silver output with the most specific mapping or structured-input error. |
| `050-STRUCTURED-INPUT-MAPPING-AC-004` | Undeclared generated output roots, generated artifact drift, stale producer CI, publication manifest mismatch, branch-tip validation substitution, and template-only mapping selection fail before silver output. |
| `050-STRUCTURED-INPUT-MAPPING-AC-005` | CLI-generated mapping output is accepted only when tool contract, invocation checksum, generated output checksum, materialization refs, package refs when supplied, and `VersionManifest` refs match. |

## Definition of Done

| ID | Criterion |
| --- | --- |
| `050-SCOPE-MAPPING-AC-001` | Mapping activation scope mismatch, private activation scope leak, exact scope match, duplicate selector dimension rejection, and selected mapping scope context manifest-inclusion fixtures pass before silver output. |
| `050-SCOPE-MAPPING-AC-002` | `ResolveOCSFMapping` distinguishes scope mismatch from missing discriminator and ambiguous discriminator without emitting silver output for scope failures. |
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
| `050-TODO-MVP-OCSF-MAPPING-ROW-CATALOG` | TODO: Provide active `ObservationToOCSFMappingRowSet` instances or exact `cadastre_only` rows for every MVP observation family, with discriminator predicates, class/activity/type values, object-path policy, enum refs, base-event policy refs, source-extension refs, fixture refs, validation refs, lifecycle status, activation scope, and `VersionManifest` refs. | Production silver `normalized_fields` for MVP observation families. | `050` mapping governance plus `100` package-set activation and `120` OCSF fixture refs. | `MAP_OCSF_ROW_MISSING` or `MAP_OCSF_ROW_AMBIGUOUS` before silver output. |
| `050-TODO-EXTERNAL-SCHEMA-ARTIFACT-CHECKSUMS` | TODO: Provide active `ExternalSchemaArtifactRef`, compiled OCSF 1.8.0 artifact checksum, compiler and validator refs, `ProfileResolutionManifest` refs, enum rule set refs, base-event policy set refs, source-extension rule-set refs, and expected output checksums. | Production OCSF-aligned `normalized_fields`. | `050` plus `120` fixture validation. | `ExternalSchemaProfile` remains non-active. |
