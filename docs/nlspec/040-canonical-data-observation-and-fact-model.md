---
doc_id: CADASTRE-NLSPEC-040
title: Canonical Data, Observation, and Fact Model
doc_type: nlspec
status: candidate
---

## Authority

This document owns the contracts listed in `Exports`. Other Cadastre NLSpecs may import those contracts by exact name and must not restate them. This document has implementation authority only after the document registry marks it `authoritative` and its acceptance criteria pass.

## Purpose

Own Cadastre core record shapes, scalar rules, identifiers, omission states, evidence references, and canonical serialization.

## Explicit Non-Scope

- Source absence authorization algorithm.
- Resolver algorithm.
- OCSF mapping rows.
- Graph projection algorithms.
- Package lifecycle.

## Imports

- `AuthorityClass`
- `VersionManifest`
- `ActivationControlledArtifactRef`
- `030.ScopeSelector`
- `030.ActivationScope`
- `030.NormalizeScopeSelector`
- `030.ActivationControlledRowSchema`
- `030.ActivationControlledRowField`

## Exports

- `ScalarType`
- `CanonicalJSON`
- `OmissionState`
- `FactAbsenceOutcome`
- `RawRecord`
- `CadastreSilverObservation`
- `SourceExtensionFieldRuleShape`
- `CanonicalEntity`
- `SourceAsset`
- `Identifier`
- `GoldFact`
- `EvidenceRef`
- `GraphNodeDeltaShape`
- `GraphEdgeDeltaShape`
- `CoreRecordSchema`
- `CoreRecordFieldRow`
- `CommonRecordHeader`
- `CoreRecordIdPolicy`
- `CoreRecordChecksumPolicy`
- `CoreRecordValidationAlgorithm`
- `CoreRecordErrorCodeSet`
- `ComputeRawRecordId`
- `ComputeGoldFactKeyId`
- `ComputeGoldFactId`
- `RawRecordSchema`
- `CadastreSilverObservationSchema`
- `CanonicalEntitySchema`
- `SourceAssetSchema`
- `IdentifierSchema`
- `GoldFactSchema`
- `EvidenceRefSchema`
- `CoreOneOfRegistry`
- `GoldFactSubjectRefKindRegistry`
- `GoldFactObjectValueKindRegistry`
- `GoldFactPredicateCatalogShapeHandoff`
- `EvidenceArtifactIdKindRegistry`
- `EvidenceArtifactClassRegistry`
- `ComputeEvidenceRefId`

### Exported schema aliases

The exported `*Schema` names are exact aliases to the corresponding row family in `CoreRecordSchema`. They do not define second schemas, field defaults, nullability, ID inputs, checksum inputs, or extension behavior.

| Exported schema name | Exact owner table or registry |
| --- | --- |
| `RawRecordSchema` | `CoreRecordSchema` rows for `RawRecord`. |
| `CadastreSilverObservationSchema` | `CoreRecordSchema` rows for `CadastreSilverObservation`. |
| `CanonicalEntitySchema` | `CoreRecordSchema` rows for `CanonicalEntity`. |
| `SourceAssetSchema` | `CoreRecordSchema` rows for `SourceAsset`. |
| `IdentifierSchema` | `CoreRecordSchema` rows for `Identifier`. |
| `GoldFactSchema` | `CoreRecordSchema` rows for `GoldFact`, including `CoreOneOfRegistry` constraints. |
| `EvidenceRefSchema` | `CoreRecordSchema` rows for `EvidenceRef`, including `EvidenceArtifactIdKindRegistry` and `EvidenceArtifactClassRegistry` constraints. |

A downstream spec may import a schema alias by exact name and route to it. A downstream spec must not restate the field schema, default, null behavior, ID input, checksum input, or extension behavior owned by this file.

## Canonical Serialization

All records owned by this spec must serialize through canonical JSON when their bytes affect IDs, checksums, replay equivalence, package activation, validation output, graph delta identity, or audit evidence.

Canonical JSON rules:

| Rule | Required behavior |
| --- | --- |
| Object key order | Lexical Unicode code point order. |
| Omitted field | Excluded from serialized bytes unless field default is explicitly materialized. |
| Null value | Distinct from omission and serialized as JSON `null` only when the schema allows null. |
| Timestamp | RFC3339 UTC with `Z`; no offset-preserving variants. |
| Decimal | Base-10 string when precision affects output; binary floating point is forbidden for checksummed values. |
| Arrays | Input order is preserved unless the field declares canonical sort keys. |
| Unknown field | Rejected unless the record declares an extension field map. |

### ScopeSelectorCanonicalSerializationHandoff

`030.ScopeSelector` and `030.ActivationScope` use `040.CanonicalJSON` and `040.ScalarType` bounds for canonical bytes, selector checksums, unknown-field rejection, null and omission behavior, arrays, maps, scalar normalization, and SHA-256 values.

`040` does not define selector schema, selector equality, selector coverage, selector specificity, subset eligibility, owner contexts, row resolution, ambiguity behavior, or selector error mapping. Those behaviors are owned only by `030`.

Any `040`-owned schema row containing `activation_scope` must validate that field as `030.ActivationScope`. Canonical bytes for the field are computed by `040.CanonicalJSON` only after `030.NormalizeScopeSelector` materializes selector defaults, sorts dimensions, sorts dimension values, rejects private bindings, and computes or verifies `selector_checksum`.

### ActivationControlledRowCanonicalizationHandoff

`030.ActivationControlledRowSchema` imports `040.CanonicalJSON` and `040.ScalarType` for activation-controlled row bytes, scalar normalization, row checksums, row-set checksums, selector checksum participation, unknown-field rejection, null behavior, omission behavior, array behavior, map behavior, timestamp behavior, and decimal behavior.

`040` does not own activation-controlled row field schemas, row-set envelopes, row refs, row selection, lifecycle state, package membership, owner algorithms, owner enum values, owner unions, owner missing errors, or owner invalid errors. Those contracts remain owned by `030` for generic mechanics and by the owner spec for domain row semantics.

Any activation-controlled row field table that names a `040.ScalarType` must use the scalar representation, default bound, canonicalization rule, and error behavior defined by this spec. An owner row table may narrow a `040.ScalarType` bound. It must not widen a `040.ScalarType` default bound unless this spec adds the wider bound first.

Activation-controlled row checksums and row-set checksums must use the same `040.CanonicalJSON` behavior for nulls, omissions, arrays, maps, decimals, timestamps, strings, object key order, and unknown fields that core records use. A row schema must not define a second JSON dialect.

Core records remain governed only by `CoreRecordSchema`. Activation-controlled rows must not be promoted into `CoreRecordSchema` unless the row is an actual `040` core record exported by this spec. A row-catalog validator and a core-record validator may share canonical JSON and scalar normalization, but they must remain separate validation surfaces and must emit owner-specific row or core errors.

## Omission Semantics

Omission states must be explicit. Optionality, nullable schema declarations, absent source field, OCSF field absence, CIM field absence, or source row absence must not decide Cadastre omission semantics by themselves.

| Omission state | Meaning |
| --- | --- |
| `observed_present` | Source evidence asserted a value. |
| `observed_empty` | Source evidence asserted an empty value. |
| `not_provided` | Source did not provide a value and no stronger interpretation is authorized. |
| `not_applicable` | Active profile states the field does not apply. |
| `permission_limited` | Source or supplier visibility did not permit observation. |
| `source_unavailable` | Source or required feed evidence was unavailable. |
| `unsupported` | Source or parser cannot represent the field. |
| `malformed` | Source value was present but invalid for the target field. |
| `redacted` | Value exists but must not be exposed to caller or downstream projection. |

### FactAbsenceOutcome

`FactAbsenceOutcome` is the closed fact-level outcome vocabulary for `GoldFact.absence_outcome`. It is distinct from `OmissionState` and from caller-visible labels owned by `110`.

| Token | Meaning | May imply authorized negative fact | Required producer |
| --- | --- | ---: | --- |
| `authorized_absent` | Exact authority, completeness, coverage, and staleness gates authorize fact absence. | Yes | `060.DeriveAbsenceOrUnknown` |
| `authorized_not_observed` | Exact rows authorize not-observed-within-complete-scope without object deletion semantics. | Only if predicate permits it | `060.DeriveAbsenceOrUnknown` |
| `unknown` | Absence cannot be determined. | No | `060.DeriveAbsenceOrUnknown` |
| `not_applicable` | The fact or control does not apply under an active row. | No | `060.DeriveAbsenceOrUnknown` or `060.ControlResultMappingRow` |
| `source_stale` | Source evidence is stale under active staleness policy. | No | `060.SourceStalenessPolicy` through `060` |
| `source_unavailable` | Required source or feed evidence is unavailable. | No | `060` |
| `scope_unavailable` | Required scope is unavailable. | No | `060` |
| `permission_limited` | Permission or visibility limits block absence. | No | `060` |
| `partial_known_gap` | Known coverage or completeness gap blocks absence. | No | `060` |
| `partial_unknown_gap` | Unknown completeness gap blocks absence. | No | `060` |
| `not_checked` | Required control or evaluation was not checked. | No | `060.ControlResultMappingRow` |
| `no_op` | Requested effect produced no fact-level outcome. | No | `060` |
| `error` | Deterministic owner error blocked the outcome. | No | owner error through `060` |
| `ambiguous` | More than one equally specific row or state matched. | No | `060` |

`FactAbsenceOutcome` must not be inferred from `OmissionState`. Parser, normalizer, resolver, graph projection, API, and analysis stages must not set `GoldFact.absence_outcome`. `GoldFact.absence_outcome = null` means no fact-level absence outcome was produced; it must not mean `unknown`.

#### SourceHistoryNoChangeProofBoundary

`no_change_proof`, `unchanged`, `not_changed`, `no_change`, and equivalent labels are not `FactAbsenceOutcome` tokens. `GoldFact.absence_outcome` with any of those values must fail schema validation before persistence.

A source-history no-change proof may become a `GoldFact` only when an active `080.GoldFactPredicateContractRow` defines the exact predicate and an active `060.AbsenceDerivationResult` authorizes the relevant absence outcome for that predicate, source dataset, scope, and requested effect. Otherwise source-history no-result, outside-window, stale-history, permission-limited-history, and unavailable-history states remain `unknown`, diagnostic-only, or owner no-op, and must not create a fact, correction, graph handoff, watermark, compliance negative output, or authorized-negative API label.

### FactAbsenceOutcomeClosureHandoff

A non-null `GoldFact.absence_outcome` is schema-valid only when the producing run's `VersionManifest` contains the `060.AbsenceDerivationResult` ref and every consulted source-authority closure ref required by `030.VersionManifestCompletenessMatrix`.

`040` does not evaluate source authority. `040` validates only that `absence_outcome` is null or one closed `FactAbsenceOutcome` token and that producer ownership is `060` for non-null values.

`authorized_absent` and `authorized_not_observed` must not be rendered, projected, exported, or audited as authorized negative output unless `060.AbsenceDerivationResult.absence_authorized = true` and the requested effect, predicate, and scope permit the output.

### CoreStateApiHandoff

`040` owns core state tokens and record fields. `110` owns caller-visible labels. API handoff must preserve assertion state and absence outcome as separate owner contexts.

| Core state | Required API handoff | Forbidden interpretation |
| --- | --- | --- |
| `GoldFact.assertion_state = conflicted` | Maps to `110.SourceStateLabel.conflicted` with assertion-state owner context. | Must not be treated as authorized absence, ambiguity, pass/fail, cleanup, graph expiry, retraction, or source deletion. |
| `GoldFact.absence_outcome = ambiguous` | Maps to `110.SourceStateLabel.ambiguous` with absence-outcome owner context. | Must not be treated as conflict, pass/fail, authorized absence, cleanup, graph expiry, or source deletion. |
| `GoldFact.assertion_state = unknown` | Maps to `110.SourceStateLabel.unknown` with assertion-state owner context. | Must not be collapsed with absence-outcome unknown in evidence drillback or audit. |
| `GoldFact.absence_outcome = unknown` | Maps to `110.SourceStateLabel.unknown` with absence-outcome owner context. | Must not imply unknown assertion state or authorized absence. |
| `GoldFact.assertion_state = no_op` or no-op correction evidence | No output change plus audit evidence unless `080` permits diagnostic output. | Must not be a source state label by default and must not create graph, compliance, absence, or remediation output. |

Record serializers and API renderers must preserve `assertion_state` and `absence_outcome` separately in evidence drillback and audit paths.

## Core Record Schema Registry

`040` is the sole normative registry for the field schemas of `RawRecord`, `CadastreSilverObservation`, `CanonicalEntity`, `SourceAsset`, `Identifier`, `GoldFact`, `EvidenceRef`, `GraphNodeDeltaShape`, and `GraphEdgeDeltaShape`. Downstream specs may import these schemas by exact name and must not restate field-level schema authority.

### ScalarType registry

`ScalarType` is a closed registry. A field that names a scalar type not listed in this table must fail with `CORE_FIELD_TYPE_INVALID` before ID or checksum computation.

| Scalar type | Representation | Default bound |
| --- | --- | --- |
| `cadastre_id` | `<prefix>_<64 lowercase hex>` unless the owning policy explicitly defines a different stable prefix. | Maximum 96 Unicode scalar values. |
| `external_ref_id` | String reference to a non-Cadastre artifact. | Maximum 512 Unicode scalar values. |
| `enum_token` | Lowercase snake-case token from a closed owner enum. | Maximum 128 Unicode scalar values. |
| `string` | Unicode string normalized to NFC before canonical serialization. | Maximum 4096 Unicode scalar values unless the field row overrides. |
| `field_path` | Dot-separated canonical path with no empty segment. | Maximum 512 Unicode scalar values. |
| `timestamp_utc` | RFC3339 UTC timestamp with `Z`. | Offsets other than `Z` are forbidden. |
| `sha256_hex` | 64 lowercase hexadecimal characters. | Exact length 64. |
| `uint64` | JSON integer from `0` through `18446744073709551615`. | Field may set a lower maximum. |
| `int64` | JSON integer in signed 64-bit range. | Field may set narrower bounds. |
| `decimal_string` | Base-10 string, no binary float. | Field declares precision and range. |
| `boolean` | JSON boolean. | Null is forbidden unless the field row permits it. |
| `canonical_object` | JSON object serialized by `CanonicalJSON`. | Maximum 65536 canonical UTF-8 bytes unless the field row overrides. |
| `array<T>` | JSON array. | Maximum 1024 elements unless the field row overrides; sort rule must be declared when order affects output. |
| `map<string,T>` | JSON object keyed by canonical strings. | Maximum 4096 entries unless the field row overrides; keys sort lexically. |
| `one_of` | Tagged union with `kind` plus exactly one matching value member. | Unknown `kind` is rejected. |

### DecimalPrecisionPolicy

Decimal values that affect IDs, checksums, confidence, graph scoring, or validation output must use canonical base-10 strings. Binary floating point is forbidden.

| Decimal class | Range | Canonical scale | Accepted input normalization | Rounding behavior | Invalid error |
| --- | --- | ---: | --- | --- | --- |
| `confidence_0_1` | `0.000000` through `1.000000` inclusive | 6 fractional digits | `0`, `0.0`, and `0.000000` normalize to `0.000000`; `1`, `1.0`, and `1.000000` normalize to `1.000000`. | No rounding is performed; inputs with more than 6 fractional digits fail. | `CORE_DECIMAL_PRECISION_INVALID` |

Persisted records must contain the canonical six-fractional-digit string. A persisted value such as `1`, `1.0`, or `1.00000` fails checksum validation because it is not canonical.

### CoreEnumRegistryOwnership

Every `enum_token` field must be governed by exactly one closed enum owner before authoritative promotion. `040` owns only the enum-token scalar grammar unless the row below assigns the enum to `040`.

Closed closure-status values for this table are:

```text
closed_in_040
closed_in_owner_spec
activation_controlled_row_set_required
blocked_validation
inactive_deferred
```

| Enum family | Owner | Applies to | Closure status |
| --- | --- | --- | --- |
| core record tokens and schema record types | `040` | `CommonRecordHeader.record_type` and core schema selectors | `closed_in_040` |
| omission states | `040` | `OmissionState` | `closed_in_040` |
| fact absence outcomes | `040`, production derivation by `060` | `FactAbsenceOutcome` and `GoldFact.absence_outcome` | `closed_in_040` for token set; `closed_in_owner_spec` for production derivation |
| assertion states | `040`, transition behavior by `080` | `GoldFact.assertion_state` and correction transitions | `closed_in_040` for default state tokens; `closed_in_owner_spec` for transitions |
| raw import and payload enums | `020`, scalar grammar by `040` | `read_target_kind`, `payload_hash_algorithm`, `payload_format`, `duplicate_status`, `quarantine_status`, `evidence_visibility` | `closed_in_owner_spec` |
| silver observation enums | `050`, scalar grammar by `040` | observation type, source category, field quality, external profile state | `closed_in_owner_spec` |
| identity enums | `070`, scalar grammar by `040` | identity decision, evidence role, review state, selector mechanism | `closed_in_owner_spec` |
| temporal and correction enums | `080`, scalar grammar by `040` | temporal quality, correction class, replay result, bitemporal query mode | `closed_in_owner_spec` |
| graph enums | `090`, scalar grammar by `040` | graph node type, edge type, operation, traversal class, assertion visibility | `closed_in_owner_spec` |
| API, package, validation, and analysis enums | `100`, `110`, `120`, `130`, scalar grammar by `040` | status, health, error, validation, and analysis tokens | `closed_in_owner_spec` |

An owner enum family marked `closed_in_owner_spec` must resolve through the named owner before authoritative promotion. No enum family row may use an unrecognized closure status.

### CoreOneOfRegistry

Each `one_of` field must use a tagged object with `kind` plus exactly one member named by the registry row. Unknown `kind`, missing value member, multiple value members, or null value member where not permitted fails with `CORE_ONE_OF_INVALID`.

| Union | Owning spec | Allowed `kind` values | Value member rule | Null/omission behavior | ID/checksum behavior | Invalid-kind error | Closure state |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `GoldFact.subject_ref` | `040`, predicate permission imported from `080` | `canonical_entity_ref`; `source_asset_ref`; `identifier_ref` | Exactly one value member defined by `GoldFactSubjectRefKindRegistry`. | Null and omission forbidden. | Complete one-of object included in gold fact key and record checksum. | `CORE_ONE_OF_INVALID` | closed_in_040 |
| `GoldFact.object_value` | `040`, predicate permission imported from `080` | `canonical_entity_ref`; `source_asset_ref`; `identifier_ref`; `string_value`; `enum_value`; `boolean_value`; `int64_value`; `uint64_value`; `decimal_value`; `timestamp_value`; `structured_value`; `null_value` | Exactly one value member defined by `GoldFactObjectValueKindRegistry`. | Null and omission forbidden at the `object_value` field. `kind = null_value` is allowed only when `080.GoldFactPredicateContractRow.null_object_policy = allowed`. | Complete one-of object included in gold fact key and record checksum. | `CORE_ONE_OF_INVALID` | closed_in_040 |
| `GraphNodeDeltaShape.source_object_ref` | `040`, behavior imported by `090` | `canonical_entity_ref`; `structural_synthetic_ref` inactive until `090` activates structural output; `generic_external_graph_ref` forbidden in MVP. | Exactly one value member matching `kind`; MVP projected canonical nodes must use `canonical_entity_ref`. | Null and omission forbidden. `structural_synthetic_ref` may be used only when a future active `090` structural row permits it. | Included in graph delta checksum; graph ID input is imported from `090`. | `CORE_ONE_OF_INVALID` | closed_mvp_graph |
| `GraphEdgeDeltaShape.source_object_ref` | `040`, behavior imported by `090` | `gold_fact_ref` only for MVP `observed_connection`; structural and generic external graph refs are forbidden in MVP. | Exactly one value member matching `kind`; MVP observed-connection edges must use `gold_fact_ref`. | Null and omission forbidden. | Included in graph delta checksum; graph ID input is imported from `090`. | `CORE_ONE_OF_INVALID` | closed_mvp_graph |
| `EvidenceRef.artifact_id` | `040`, class permission imported by `020`, `030`, `100`, and `130` | `cadastre_record_ref`; `activation_artifact_ref`; `lakehouse_artifact_ref`; `external_artifact_ref` | Exactly one value member defined by `EvidenceArtifactIdKindRegistry`; raw payload bytes are forbidden. | Null and omission forbidden. | Complete one-of object included in evidence-ref ID and record checksum. | `CORE_ONE_OF_INVALID` | closed_in_040 |

### GoldFactSubjectRefKindRegistry

`GoldFact.subject_ref.kind` is a closed token set owned by `040`. Predicate-specific permission to use a subject kind is owned by `080.GoldFactPredicateContractRow`.

| `subject_ref.kind` | Required value member | Member type | Required behavior |
| --- | --- | --- | --- |
| `canonical_entity_ref` | `canonical_entity_id` | `cadastre_id` | The referenced record must be an eligible `CanonicalEntity` under `070` when the selected predicate contract permits this kind. |
| `source_asset_ref` | `source_asset_id` | `cadastre_id` | The referenced record must be an eligible `SourceAsset` under `070` when the selected predicate contract permits this kind. |
| `identifier_ref` | `identifier_id` | `cadastre_id` | The referenced record must be an eligible `Identifier` under `070` when the selected predicate contract permits this kind. |

`subject_ref.kind` must be one of the values in this registry. The one-of object must contain `kind` and exactly one matching value member. Null, omission, multiple value members, unknown kinds, mismatched members, and null value members fail with `CORE_ONE_OF_INVALID` before `gold_fact_key_id` computation.

Raw records, silver observations, evidence refs, graph node IDs, graph edge IDs, backend IDs, package IDs, validation artifacts, lineage artifacts, telemetry IDs, external schema objects, OCSF objects, and source-native strings are not valid `GoldFact.subject_ref` kinds.

### GoldFactObjectValueKindRegistry

`GoldFact.object_value.kind` is a closed token set owned by `040`. Predicate-specific permission, null-object permission, and structured-value schema permission are owned by `080.GoldFactPredicateContractRow`.

| `object_value.kind` | Required value member | Member type | Required behavior |
| --- | --- | --- | --- |
| `canonical_entity_ref` | `canonical_entity_id` | `cadastre_id` | Identity-like object value carried by canonical entity reference. |
| `source_asset_ref` | `source_asset_id` | `cadastre_id` | Identity-like object value carried by source asset reference. |
| `identifier_ref` | `identifier_id` | `cadastre_id` | Identity-like object value carried by typed identifier reference. |
| `string_value` | `value` | `string` | Bounded string value. It must not carry an identity-like object when `080.identity_like_string_policy = reject`. |
| `enum_value` | `value` | `enum_token` | Closed owner enum value selected by the active predicate contract. |
| `boolean_value` | `value` | `boolean` | JSON boolean. |
| `int64_value` | `value` | `int64` | Signed 64-bit integer. |
| `uint64_value` | `value` | `uint64` | Unsigned 64-bit integer. |
| `decimal_value` | `decimal_value` | `canonical_object` | Object containing `decimal_class` and canonical `value`; binary floating point is forbidden. |
| `timestamp_value` | `value` | `timestamp_utc` | RFC3339 UTC timestamp with `Z`. |
| `structured_value` | `structured_value` | `canonical_object` | Object containing `schema_ref` and `value`; `schema_ref` must reference an active owner-declared object-value schema row. |
| `null_value` | `null_value` | JSON null | Explicit null object sentinel; allowed only when `080.GoldFactPredicateContractRow.null_object_policy = allowed`. |

`GoldFact.object_kind` must equal `GoldFact.object_value.kind` before `gold_fact_key_id` computation. The one-of object must contain `kind` and exactly one matching value member. Null, omission, multiple value members, unknown kinds, mismatched members, null value members other than the `null_value` sentinel, and `object_kind` mismatch fail before ID computation.

Identity-like object values, including IP addresses, hostnames, DNS names, PTR names, provider keys, mapped targets, graph keys, source-native identities, and selector-only values, must use one of the reference kinds when an active predicate contract treats the value as identity-like. They must not be encoded as `string_value` to bypass resolver governance.

`structured_value.schema_ref` must reference an active owner-declared object-value schema row and must not serve as an unbounded escape hatch. OCSF objects, observables, raw data, unmapped fields, lineage facets, detection results, registry labels, graph backend objects, and external taxonomies are not object authority unless an owning spec derives a bounded value through the gold derivation interface.

### GoldFactPredicateCatalogShapeHandoff

The MVP predicate catalog owned by `080` must not add subject or object shapes beyond the closed registries in this spec.

| Handoff rule | Required behavior |
| --- | --- |
| Subject kind set | `080.GoldFactPredicateContractRow.allowed_subject_ref_kinds` must be a non-empty subset of `GoldFactSubjectRefKindRegistry`. |
| Object value kind set | `080.GoldFactPredicateContractRow.allowed_object_value_kinds` must be a non-empty subset of `GoldFactObjectValueKindRegistry`. |
| New kind creation | Forbidden. A predicate catalog row that names a subject kind or object value kind outside the `040` registries fails before activation. |
| Entity-type eligibility | Validated by `070` and the selected `080` predicate row after the core one-of shape passes. |
| Identifier-type eligibility | Validated by `070` and the selected `080` predicate row after the core one-of shape passes. |
| Structured values | `object_value.kind = structured_value` must include `schema_ref`. The schema ref must be active, checksum-valid, package-set-valid when package-supplied, and manifest-included under `080`. |
| Object-kind equality | `GoldFact.object_kind` must equal `GoldFact.object_value.kind` before `ComputeGoldFactKeyId`. |
| Key ID input | `ComputeGoldFactKeyId` uses the complete canonical subject object and complete canonical object/value object after predicate-catalog validation. Serializing only the inner value members is forbidden. |

`040` validates core shape and canonical serialization. `080` may narrow the permitted combinations for MVP fact semantics. `080` must not widen `040` registries, redefine the one-of shape, or define a second key serialization.

### EvidenceArtifactIdKindRegistry

`EvidenceRef.artifact_id.kind` is a closed token set owned by `040`. Artifact-class eligibility is governed by `EvidenceArtifactClassRegistry` and owner handoffs from `020`, `030`, `100`, and `130`.

| `artifact_id.kind` | Required value member | Member type | Required behavior |
| --- | --- | --- | --- |
| `cadastre_record_ref` | `record_id` | `cadastre_id` | References a persisted Cadastre record. `artifact_checksum` must equal the referenced record checksum. |
| `activation_artifact_ref` | `artifact_id` | `external_ref_id` | References an activation-controlled artifact through `030.ActivationControlledArtifactRef`. |
| `lakehouse_artifact_ref` | `artifact_id` | `external_ref_id` | References immutable lakehouse object, manifest, table, snapshot, commit, or canonical metadata bytes declared by `020`. |
| `external_artifact_ref` | `artifact_id` | `external_ref_id` | References external standard, report, repository metadata, provenance, SBOM, compiled schema, or source-native artifact bytes or owner-declared canonical metadata bytes. |

`artifact_id.kind` must be one of the four values in this registry. The one-of object must contain `kind` and exactly one matching value member. Null, omission, unknown kind, multiple value members, mismatched members, null value members, and raw payload bytes fail before `evidence_ref_id` computation.

`artifact_checksum` is mandatory for every `artifact_id.kind`. For `cadastre_record_ref`, `artifact_checksum` must equal the referenced record checksum. For `activation_artifact_ref`, `lakehouse_artifact_ref`, and `external_artifact_ref`, `artifact_checksum` must equal immutable artifact bytes or owner-declared canonical metadata bytes. A mutable label, object prefix, table name, branch name, latest pointer, or payload-inline value must not satisfy `EvidenceRef`.

### EvidenceArtifactClassRegistry

`EvidenceArtifactClassRegistry` is the stable `040` pairing contract between `EvidenceRef.artifact_class` and `EvidenceRef.artifact_id.kind`. Concrete artifact-class row sets may be activated by owner specs, but new `artifact_id.kind` values require a new `040` schema version.

| Registry field | Required behavior |
| --- | --- |
| `artifact_class` | Closed owner-declared artifact class token. |
| `allowed_artifact_id_kind` | One token from `EvidenceArtifactIdKindRegistry`. |
| `owner_spec` | Spec that owns the artifact class and checksum basis. |
| `allowed_authority_class` | Authority class permitted for the evidence citation. |
| `checksum_basis` | `referenced_record_checksum`, `immutable_artifact_bytes`, or `owner_declared_canonical_metadata_bytes`. |
| `payload_inline_policy` | Default `forbidden`; no row may permit raw payload bytes in `EvidenceRef`. |
| `redaction_owner` | Spec that owns caller-visible and audit-visible exposure. |
| `version_manifest_requirement` | Required manifest refs for output-affecting evidence. |
| `validation_refs` | Non-empty refs to `120` evidence-artifact validation rows. |
| `closure_status` | `closed_in_040`, `closed_in_owner_spec`, `activation_controlled_row_set_required`, `blocked_validation`, or `inactive_deferred`. |

| Top-level artifact class family | Allowed `artifact_id.kind` | Owner spec | Required checksum basis |
| --- | --- | --- | --- |
| Cadastre persisted record classes | `cadastre_record_ref` | Owner of the referenced record class | Referenced record checksum. |
| `030.ActivationControlledArtifactRef` classes | `activation_artifact_ref` | `030` plus artifact owner | Immutable activation artifact bytes or owner-declared canonical metadata bytes. |
| `020` lakehouse table, object, manifest, snapshot, and commit artifacts | `lakehouse_artifact_ref` | `020` | Immutable lakehouse bytes or owner-declared canonical metadata bytes. |
| `structured_input_repository_profile` | `activation_artifact_ref` | `030` | Immutable activation artifact bytes or owner-declared canonical profile metadata bytes. |
| `structured_input_repository_snapshot` | `cadastre_record_ref` or `external_artifact_ref` | `030` | Persisted Cadastre snapshot record checksum or immutable external repository metadata bytes. |
| `structured_input_validation_run` | `cadastre_record_ref` or `external_artifact_ref` | `120` | Persisted validation record checksum or immutable validation output bytes for the exact snapshot. |
| `structured_input_materialization_result` | `cadastre_record_ref` | `100` | Persisted materialization result checksum. |
| `structured_input_maintenance_tool_contract` | `activation_artifact_ref` | `030` | Canonical tool contract bytes. |
| `structured_input_maintenance_tool_invocation` | `cadastre_record_ref` or `external_artifact_ref` | `030` | Persisted invocation result checksum or immutable external invocation metadata bytes. |
| `structured_input_repository_template_contract` | `activation_artifact_ref` | `030` | Canonical template contract bytes. |
| `structured_input_repository_ci_contract` | `activation_artifact_ref` | `030` | Canonical CI contract bytes. |
| `structured_input_publication_manifest` | `cadastre_record_ref` or `external_artifact_ref` | `100` | Persisted imported publication manifest checksum or immutable external manifest bytes. |
| `structured_input_candidate_sync_record` | `cadastre_record_ref` | `030` | Persisted sync record checksum. |
| `structured_input_repository_group` | `activation_artifact_ref` | `030` | Canonical group contract bytes. |
| External standards, reports, source-native artifacts, compiled schemas, package repository metadata, provenance, and SBOM artifacts | `external_artifact_ref` | Owner spec that imports the artifact | Immutable external bytes or owner-declared canonical metadata bytes. |

A new artifact class may be activated through the owning spec only when it maps to one existing `artifact_id.kind`, declares checksum basis, declares redaction owner, declares `VersionManifest` requirements, and has passing validation refs. `EvidenceRef.artifact_class` and `EvidenceRef.artifact_id.kind` mismatch fails with `EVIDENCE_ARTIFACT_CLASS_KIND_MISMATCH` before `evidence_ref_id` computation.

Tool logs, CI logs, repository URLs, raw branch names, raw file paths, raw generated artifact bytes, hook payloads, commit messages containing private data, and raw structured-input file bytes must not be inlined in `EvidenceRef`. They may appear only as redacted refs, checksums, byte counts, or owner-declared canonical metadata bytes when `010`, `110`, and the artifact owner permit that exposure.

Branch names, tags, repository URLs, pull request numbers, hook logs, merge event labels, and commit timestamps must not be `artifact_checksum` inputs unless they are serialized as owner-declared canonical metadata bytes and are not used as production authority. Raw repository file bytes and raw structured input payload bytes must not be inlined in `EvidenceRef`.

### CommonRecordHeader

`CommonRecordHeader` applies to every persisted core record in this registry. Per-record field tables define additional fields. When a per-record table includes a record-specific ID field such as `raw_record_id`, that field must equal `CommonRecordHeader.record_id`; a mismatch fails with `CORE_RECORD_ID_MISMATCH`.

| Field | Type | Required | Default/null/omit | Checksum rule |
| --- | --- | ---: | --- | --- |
| `record_type` | `enum_token` | Yes | No default. Must equal the exported record token. | Included. |
| `schema_version` | `string` | Yes | No default. MVP value format is `040.<record>.v1`. | Included. |
| `record_id` | `cadastre_id` | Yes | No default. Must equal the owner ID algorithm output. | Included, but never an ID input. |
| `record_checksum` | `sha256_hex` | Yes | Computed over canonical bytes excluding this field. | Excluded from its own checksum. |
| `version_manifest_id` | `cadastre_id` or null | Yes | Null allowed only for validation fixtures, shadow records, or canary records when `030` permits. Production null fails with `VERSION_MANIFEST_INCOMPLETE`. | Included. |

### CoreRecordFieldRow format

Every field schema table in this registry uses the following columns and meanings.

| Column | Required meaning |
| --- | --- |
| `field_path` | Canonical field path relative to the record body, excluding `CommonRecordHeader` fields. |
| `type` | One `ScalarType` value or a closed union named in this spec. |
| `required` | `yes` means the field must be materialized after default application. |
| `default` | Materialized default. `none` means caller must supply a non-null value unless `null_allowed = yes`. |
| `null_allowed` | `yes` permits JSON null and requires a declared meaning. |
| `omit_allowed` | `yes` permits absence. Omission remains distinct from null. |
| `bounds` | Scalar, byte, length, precision, enum, or policy bound. |
| `canonical_sort_key` | Deterministic ordering rule for arrays or maps; `n/a` for scalars. |
| `id_input` | `ordered:<n>`, `derived`, `no`, or imported owner policy. |
| `checksum_input` | `yes` when included in `record_checksum`; `no` only for `record_checksum` itself or explicitly excluded volatile values. |
| `extension_policy` | `closed` unless the row names an owning extension-map policy. |
| `redaction_owner` | Owner spec that governs exposure and redaction. |
| `missing_error` | Error emitted when a required field is omitted. |
| `invalid_error` | Error emitted when value, bounds, type, ID, checksum, extension, or security validation fails. |

`required = yes` means the field must be materialized. Optional absent fields must be omitted unless the field table declares a materialized default. Null is forbidden unless `null_allowed = yes`.

### RawRecordSchema

| field_path | type | required | default | null_allowed | omit_allowed | bounds | canonical_sort_key | id_input | checksum_input | extension_policy | redaction_owner | missing_error | invalid_error |
| --- | --- | ---: | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `raw_record_id` | `cadastre_id` | yes | none | no | no | scalar default | n/a | derived | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `feed_profile_id` | `cadastre_id` | yes | none | no | no | scalar default | n/a | ordered:1 | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `read_target_kind` | `enum_token` | yes | none | no | no | scalar default | n/a | ordered:2 | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `source_category` | `enum_token` | yes | none | no | no | scalar default | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `source_dataset` | `string` | yes | none | no | no | scalar default | n/a | ordered:3 | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `source_record_type` | `string` | yes | `unknown` only when source type is unavailable | no | no | scalar default | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `scope_key_set` | `canonical_object` | yes | none | no | no | max 65536 canonical bytes; keys lexical | n/a | ordered:4 | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `supplier_batch_or_object_identity` | `canonical_object` | yes | none | no | no | max 65536 canonical bytes | n/a | ordered:5 | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `source_native_record_identity` | `canonical_object` | yes | materialized null sentinel when absent | yes | no | null means no source-native identity was supplied | n/a | ordered:6 | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `canonical_payload_hash` | `sha256_hex` | yes | none | no | no | scalar default | n/a | ordered:7 | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `payload_hash_algorithm` | `enum_token` | yes | `sha256` | no | no | MVP closed enum: `sha256` | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `payload_format` | `enum_token` | yes | none | no | no | scalar default | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `payload_byte_length` | `uint64` | yes | none | no | no | 0 through `020.LakehouseReadPolicy.max_payload_byte_length` | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `payload_ref` | `canonical_object` | yes | none | no | no | bounded lakehouse ref only; credentials and private routes forbidden | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `PRIVATE_BINDING_LEAK` |
| `source_event_metadata` | `canonical_object` | yes | none | no | no | max 65536 canonical bytes; no authority by itself | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `source_schema_ref` | `canonical_object` | yes | null when unavailable | yes | no | null means no schema ref supplied | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `import_profile_version` | `string` | yes | none | no | no | scalar default | n/a | ordered:8 | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `raw_record_import_run_id` | `cadastre_id` | yes | none | no | no | scalar default | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `raw_feed_manifest_id` | `cadastre_id` | yes | none | no | no | scalar default | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `duplicate_status` | `enum_token` | yes | `not_duplicate` | no | no | scalar default | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `quarantine_status` | `enum_token` | yes | `not_quarantined` | no | no | scalar default | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `partition_key` | `canonical_object` | yes | null when not partitioned | yes | no | scalar default | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `evidence_visibility` | `enum_token` | yes | `raw_restricted` | no | no | scalar default | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `lineage_refs` | `array<cadastre_id>` | yes | `[]` | no | no | max 1024 refs | lexical by ref ID | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |

### CadastreSilverObservationSchema

| field_path | type | required | default | null_allowed | omit_allowed | bounds | canonical_sort_key | id_input | checksum_input | extension_policy | redaction_owner | missing_error | invalid_error |
| --- | --- | ---: | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `observation_id` | `cadastre_id` | yes | none | no | no | scalar default | n/a | derived | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `raw_evidence_refs` | `array<cadastre_id>` | yes | none | no | no | 1 through 1024 refs | lexical by ref ID | ordered:1 | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `observation_type` | `enum_token` | yes | none | no | no | scalar default | n/a | ordered:2 | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `mapping_bundle_id` | `cadastre_id` | yes | none | no | no | scalar default | n/a | ordered:3 | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `external_schema_profile_id` | `cadastre_id` | yes | null only when `observation_type` is declared `cadastre_only` | yes | no | nonnull unless `cadastre_only` | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `source_scope` | `canonical_object` | yes | none | no | no | max 65536 canonical bytes; keys lexical | n/a | ordered:4 | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `normalized_payload_checksum` | `sha256_hex` | yes | none | no | no | scalar default | n/a | ordered:5 | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `normalized_fields` | `canonical_object` | yes | `{}` only for `cadastre_only` | no | no | validated by `050.ExternalSchemaProfile` unless `cadastre_only` | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `source_extension_fields` | `canonical_object` | yes | `{}` | no | no | every path declared by `050.SourceExtensionFieldRule` | n/a | no | yes | `050.SourceExtensionFieldRule` | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `field_quality` | `map<string,canonical_object>` | yes | `{}` | no | no | required rows for omission, redaction, parser quality, or source-quality state | keys lexical | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `observed_at` | `timestamp_utc` | yes | null when no authoritative observation time exists | yes | no | null requires `observed_time_quality` explanation | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `observed_time_quality` | `enum_token` | yes | none | no | no | scalar default | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `collected_or_imported_at` | `timestamp_utc` | yes | none | no | no | scalar default | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `lineage` | `canonical_object` | yes | none | no | no | max 65536 canonical bytes | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `confidence_hints` | `array<canonical_object>` | yes | `[]` | no | no | scalar default | canonical JSON bytes | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `identity_inputs` | `array<canonical_object>` | yes | `[]` | no | no | scalar default | canonical JSON bytes | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `flow_role_evidence` | `canonical_object` | yes | null unless observation type can carry flow direction evidence | yes | no | scalar default | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `redaction_summary` | `canonical_object` | yes | `{}` | no | no | scalar default | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |

### CadastreOnlyCoreValidationHandoff

`040` validates only core record shape for `cadastre_only` silver output. It does not decide whether an observation is `cadastre_only`; that decision is owned by `050.ResolveOCSFMapping` and the selected `050.ObservationToOCSFMappingRow`.

| Core validation condition | Required `040` behavior |
| --- | --- |
| `external_schema_profile_id = null` | Valid only when `VersionManifest.included_refs` contains a selected active `050` `cadastre_only` row ref, selected row checksum, row-set ref, row-set checksum, validation refs, package-set refs when package-supplied, and lifecycle transition evidence refs. Missing evidence fails before persistence. |
| `normalized_fields = {}` | Shape-valid only for `cadastre_only` output. Non-empty normalized fields with null profile fail before persistence unless a later `040` amendment defines a Cadastre-owned normalized field schema. |
| `normalized_payload_checksum` | Must be present and must validate against the checksum basis declared by `050.CadastreOnlyMappingOutputPolicy` or the active OCSF mapping policy. |
| Unknown fields | Rejected by `CoreRecordValidationAlgorithm`; `cadastre_only` does not open an extension map. |

A null external profile without selected `050` row evidence must fail with `CORE_FIELD_TYPE_INVALID` or the more specific mapping owner error before persistence. `040` must not infer `cadastre_only` from null profile, empty normalized fields, observation type, source dataset, package label, or validation fixture name.

### SourceExtensionFieldRuleShape

`SourceExtensionFieldRuleShape` is the `040`-owned primitive shape for a source-extension field rule payload. It defines field names, scalar types, canonical serialization, checksum inclusion, and unknown-field rejection. It does not decide whether a rule is active, whether a path is permitted, or whether a mapping bundle may emit the field; those behaviors are owned by `050`.

| field_path | type | required | default | null_allowed | omit_allowed | bounds |
| --- | --- | ---: | --- | --- | --- | --- |
| `rule_id` | `cadastre_id` or `external_ref_id` | yes | none | no | no | stable within `050` |
| `namespace` | `field_path` | yes | none | no | no | max 512 |
| `field_path` | `field_path` | yes | none | no | no | max 512 |
| `type` | `enum_token` | yes | none | no | no | closed by `050` |
| `bounds` | `canonical_object` | yes | none | no | no | max 65536 bytes |
| `redaction` | `enum_token` | yes | none | no | no | closed by `050` |
| `collision_policy` | `enum_token` | yes | none | no | no | closed by `050` |
| `secret_scan_policy` | `enum_token` | yes | none | no | no | closed by `050` |
| `ocsf_reserved_name_policy` | `enum_token` | yes | none | no | no | closed by `050` |
| `permitted_observation_types` | `array<enum_token>` | yes | `[]` | no | no | sorted lexical |
| `permitted_mapping_bundle_refs` | `array<cadastre_id>` | yes | `[]` | no | no | sorted lexical |
| `validation_fixture_refs` | `array<external_ref_id>` | yes | none | no | no | non-empty before activation |
| `activation_scope` | `canonical_object` | yes | none | no | no | must validate as `030.ActivationScope`; canonical bytes are computed by `040.CanonicalJSON` after `030.NormalizeScopeSelector` materializes defaults |
| `lifecycle_status` | `enum_token` | yes | none | no | no | imported from `030` |

Shape validation materializes required defaults before checksum computation. Unknown fields fail with `CORE_UNKNOWN_FIELD`. Non-canonical ordering fails checksum validation. `040` validates shape and canonical bytes only. `050` validates whether the rule permits a `source_extension_fields` path.

`040` defines primitive bytes, scalar bounds, canonical bytes, and unknown-field behavior only for `SourceExtensionFieldRuleShape`. `050` may narrow string, array, map, object, namespace, redaction, collision, and secret-scan bounds, but it must not widen a `040` primitive scalar bound unless this spec is amended first.

### CanonicalEntitySchema

| field_path | type | required | default | null_allowed | omit_allowed | bounds | canonical_sort_key | id_input | checksum_input | extension_policy | redaction_owner | missing_error | invalid_error |
| --- | --- | ---: | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `canonical_entity_id` | `cadastre_id` | yes | none | no | no | scalar default | n/a | derived | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `entity_type` | `enum_token` | yes | none | no | no | scalar default | n/a | ordered:1 | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `creation_identity_decision_id` | `cadastre_id` | yes | none | no | no | scalar default | n/a | ordered:2 | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `resolver_profile_id` | `cadastre_id` | yes | none | no | no | scalar default | n/a | ordered:3 | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `identity_policy_version` | `string` | yes | none | no | no | scalar default | n/a | ordered:4 | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `lifecycle_state` | `enum_token` | yes | none | no | no | scalar default | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `valid_from` | `timestamp_utc` | yes | null when validity is unknown | yes | no | scalar default | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `valid_to` | `timestamp_utc` | yes | null means open interval | yes | no | scalar default | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `known_from` | `timestamp_utc` | yes | none | no | no | scalar default | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `known_to` | `timestamp_utc` | yes | null means open knowledge interval | yes | no | scalar default | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `current_status` | `enum_token` | yes | `active_unknown` | no | no | scalar default | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `source_decision_refs` | `array<cadastre_id>` | yes | `[]` | no | no | scalar default | lexical by ref ID | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `supersedes_entity_ids` | `array<cadastre_id>` | yes | `[]` | no | no | scalar default | lexical by ref ID | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `split_from_entity_id` | `cadastre_id` | yes | null unless split correction created the entity | yes | no | scalar default | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |

`creation_identity_decision_id` is a reference to a `070.IdentityDecision`. Creation permission, creation decision class validity, attachment semantics, merge semantics, and split semantics are owned by `070`; this schema only validates field shape and canonical bytes.

### SourceAssetSchema

| field_path | type | required | default | null_allowed | omit_allowed | bounds | canonical_sort_key | id_input | checksum_input | extension_policy | redaction_owner | missing_error | invalid_error |
| --- | --- | ---: | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `source_asset_id` | `cadastre_id` | yes | none | no | no | scalar default | n/a | derived | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `source_scope` | `canonical_object` | yes | none | no | no | max 65536 canonical bytes; keys lexical | n/a | ordered:1 | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `source_asset_type` | `enum_token` | yes | none | no | no | scalar default | n/a | ordered:2 | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `source_native_identity` | `canonical_object` | yes | none | no | no | scalar default | n/a | ordered:3 | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `asset_generation_key` | `canonical_object` | yes | null only when generation evidence is unavailable | yes | no | scalar default | n/a | ordered:4 | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `valid_from` | `timestamp_utc` | yes | null when validity is unknown | yes | no | scalar default | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `valid_to` | `timestamp_utc` | yes | null means open interval | yes | no | scalar default | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `known_from` | `timestamp_utc` | yes | none | no | no | scalar default | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `known_to` | `timestamp_utc` | yes | null means open knowledge interval | yes | no | scalar default | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `evidence_refs` | `array<cadastre_id>` | yes | `[]` | no | no | scalar default | lexical by ref ID | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `lifecycle_evidence` | `canonical_object` | yes | `{}` | no | no | scalar default | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `current_identity_decision_refs` | `array<cadastre_id>` | yes | `[]` | no | no | scalar default | lexical by ref ID | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |

### IdentifierSchema

| field_path | type | required | default | null_allowed | omit_allowed | bounds | canonical_sort_key | id_input | checksum_input | extension_policy | redaction_owner | missing_error | invalid_error |
| --- | --- | ---: | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `identifier_id` | `cadastre_id` | yes | none | no | no | scalar default | n/a | derived | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `identifier_type` | `enum_token` | yes | none | no | no | scalar default | n/a | ordered:1 | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `normalized_value` | `string` | yes | none | no | no | scalar default | n/a | ordered:2 | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `raw_value` | `string` | yes | null only when unavailable but normalized value is preserved from evidence | yes | no | scalar default | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `identifier_scope` | `canonical_object` | yes | none | no | no | max 65536 canonical bytes; keys lexical | n/a | ordered:3 | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `valid_from` | `timestamp_utc` | yes | null when validity is unknown | yes | no | scalar default | n/a | ordered:4 | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `valid_to` | `timestamp_utc` | yes | null means open interval | yes | no | scalar default | n/a | ordered:5 | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `known_from` | `timestamp_utc` | yes | none | no | no | scalar default | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `known_to` | `timestamp_utc` | yes | null means open knowledge interval | yes | no | scalar default | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `quality` | `canonical_object` | yes | `{}` | no | no | scalar default | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `evidence_refs` | `array<cadastre_id>` | yes | `[]` | no | no | scalar default | lexical by ref ID | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |

### GoldFactSchema

| field_path | type | required | default | null_allowed | omit_allowed | bounds | canonical_sort_key | id_input | checksum_input | extension_policy | redaction_owner | missing_error | invalid_error |
| --- | --- | ---: | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `gold_fact_id` | `cadastre_id` | yes | none | no | no | scalar default | n/a | derived | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `gold_fact_key_id` | `cadastre_id` | yes | none | no | no | scalar default | n/a | derived | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `fact_type` | `enum_token` | yes | none | no | no | scalar default | n/a | ordered:1 for key | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `subject_ref` | `one_of` | yes | none | no | no | `040.GoldFactSubjectRefKindRegistry`; predicate-specific subset owned by `080` | n/a | ordered:2 for key | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_ONE_OF_INVALID` |
| `predicate` | `enum_token` | yes | none | no | no | scalar default | n/a | ordered:3 for key | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `object_kind` | `enum_token` | yes | none | no | no | Must equal `object_value.kind` before ID computation. | n/a | ordered:4 for key | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_ONE_OF_INVALID` |
| `object_value` | `one_of` | yes | none | no | no | `040.GoldFactObjectValueKindRegistry`; predicate-specific subset and null policy owned by `080` | n/a | ordered:5 for key | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_ONE_OF_INVALID` |
| `valid_from` | `timestamp_utc` | yes | null when valid start is unknown and policy permits | yes | no | scalar default | n/a | ordered:6 for key | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `valid_to` | `timestamp_utc` | yes | null means open valid interval | yes | no | scalar default | n/a | ordered:7 for key | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `known_from` | `timestamp_utc` | yes | none | no | no | scalar default | n/a | ordered:2 for immutable ID | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `known_to` | `timestamp_utc` | yes | null means open knowledge interval; effective closure materializes through `080.GoldFactChangeSet` | yes | no | scalar default | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `assertion_state` | `enum_token` | yes | none | no | no | scalar default | n/a | ordered:3 for immutable ID | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `confidence` | `decimal_string` | yes | none | no | no | `040.DecimalPrecisionPolicy.confidence_0_1`: canonical six fractional digits, no rounding, range `0.000000` through `1.000000` inclusive | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_DECIMAL_PRECISION_INVALID` |
| `authority_profile_row_id` | `cadastre_id` | yes | none | no | no | scalar default | n/a | ordered:4 for immutable ID | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `temporal_resolution_id` | `cadastre_id` | yes | none | no | no | scalar default | n/a | ordered:5 for immutable ID | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `coverage_assertion_refs` | `array<cadastre_id>` | yes | `[]`; non-empty when coverage-sensitive | no | no | scalar default | lexical by ref ID | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `evidence_refs` | `array<cadastre_id>` | yes | none | no | no | 1 through 1024 refs unless derived no-op | lexical by ref ID | ordered:6 for immutable ID | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `correction_policy_id` | `cadastre_id` | yes | none | no | no | scalar default | n/a | ordered:7 for immutable ID | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `correction_refs` | `array<cadastre_id>` | yes | `[]` | no | no | scalar default | lexical by ref ID | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `absence_outcome` | `enum_token` | yes | null unless produced by `060.DeriveAbsenceOrUnknown` | yes | no | scalar default | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |

### EvidenceRefSchema

| field_path | type | required | default | null_allowed | omit_allowed | bounds | canonical_sort_key | id_input | checksum_input | extension_policy | redaction_owner | missing_error | invalid_error |
| --- | --- | ---: | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `evidence_ref_id` | `cadastre_id` | yes | none | no | no | scalar default | n/a | derived | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `artifact_class` | `enum_token` | yes | none | no | no | scalar default | n/a | ordered:1 | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `artifact_id` | `one_of` | yes | none | no | no | `040.EvidenceArtifactIdKindRegistry`; class/kind pair must match `040.EvidenceArtifactClassRegistry` | n/a | ordered:2 | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_ONE_OF_INVALID` |
| `artifact_checksum` | `sha256_hex` | yes | none | no | no | scalar default | n/a | ordered:3 | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `evidence_role` | `enum_token` | yes | none | no | no | scalar default | n/a | ordered:4 | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `field_paths` | `array<field_path>` | yes | `[]` | no | no | scalar default | lexical by field path | ordered:5 | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `visibility_state` | `enum_token` | yes | `restricted_metadata` | no | no | scalar default | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `redaction_state` | `enum_token` | yes | `not_redacted` | no | no | scalar default | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `authority_class` | `enum_token` | yes | `supporting_evidence` | no | no | scalar default | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `source_scope` | `canonical_object` | yes | null when not source-scoped | yes | no | scalar default | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |

### GraphNodeDeltaShapeSchema

| field_path | type | required | default | null_allowed | omit_allowed | bounds | canonical_sort_key | id_input | checksum_input | extension_policy | redaction_owner | missing_error | invalid_error |
| --- | --- | ---: | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `graph_node_delta_id` | `cadastre_id` | yes | none | no | no | scalar default | n/a | imported `090.GraphNodeDeltaIdPolicy` | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `delta_operation` | `enum_token` | yes | none | no | no | scalar default | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `graph_node_id` | `cadastre_id` | yes | none | no | no | scalar default | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `GRAPH_BACKEND_ID_FORBIDDEN` |
| `node_type` | `enum_token` | yes | none | no | no | scalar default | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `source_object_ref` | `one_of` | yes | none | no | no | scalar default | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `properties` | `canonical_object` | yes | `{}`; every property declared by `090.GraphPropertyEvidencePolicy` | no | no | scalar default | n/a | no | yes | `090.GraphPropertyEvidencePolicy` | `110` | `CORE_REQUIRED_FIELD_MISSING` | `EVIDENCE_REF_RAW_PAYLOAD_FORBIDDEN` |
| `evidence_refs` | `array<cadastre_id>` | yes | `[]` only when `090` permits structural synthetic output | no | no | scalar default | lexical by ref ID | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `assertion_state` | `enum_token` | yes | none | no | no | scalar default | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `valid_from` | `timestamp_utc` | yes | null when validity is unknown | yes | no | scalar default | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `valid_to` | `timestamp_utc` | yes | null means open interval | yes | no | scalar default | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `known_from` | `timestamp_utc` | yes | null when unknown and `090` permits | yes | no | scalar default | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `known_to` | `timestamp_utc` | yes | null means open knowledge interval | yes | no | scalar default | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `projection_profile_id` | `cadastre_id` | yes | none | no | no | scalar default | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `graph_delta_set_id` | `cadastre_id` | yes | none | no | no | scalar default | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |

### GraphEdgeDeltaShapeSchema

| field_path | type | required | default | null_allowed | omit_allowed | bounds | canonical_sort_key | id_input | checksum_input | extension_policy | redaction_owner | missing_error | invalid_error |
| --- | --- | ---: | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `graph_edge_delta_id` | `cadastre_id` | yes | none | no | no | scalar default | n/a | imported `090.GraphEdgeDeltaIdPolicy` | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `delta_operation` | `enum_token` | yes | none | no | no | scalar default | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `graph_edge_id` | `cadastre_id` | yes | none | no | no | scalar default | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `GRAPH_BACKEND_ID_FORBIDDEN` |
| `edge_type` | `enum_token` | yes | none | no | no | scalar default | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `from_graph_node_id` | `cadastre_id` | yes | none | no | no | scalar default | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `GRAPH_BACKEND_ID_FORBIDDEN` |
| `to_graph_node_id` | `cadastre_id` | yes | none | no | no | scalar default | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `GRAPH_BACKEND_ID_FORBIDDEN` |
| `direction_rule_ref` | `cadastre_id` | yes | none | no | no | scalar default | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `properties` | `canonical_object` | yes | `{}`; every property declared by `090.GraphPropertyEvidencePolicy` | no | no | scalar default | n/a | no | yes | `090.GraphPropertyEvidencePolicy` | `110` | `CORE_REQUIRED_FIELD_MISSING` | `EVIDENCE_REF_RAW_PAYLOAD_FORBIDDEN` |
| `evidence_refs` | `array<cadastre_id>` | yes | `[]` only when `090` permits structural synthetic output | no | no | scalar default | lexical by ref ID | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `assertion_state` | `enum_token` | yes | none | no | no | scalar default | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `confidence` | `decimal_string` | yes | null when edge type has no confidence policy | yes | no | `040.DecimalPrecisionPolicy.confidence_0_1`; canonical six fractional digits when present | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_DECIMAL_PRECISION_INVALID` |
| `valid_from` | `timestamp_utc` | yes | null when validity is unknown | yes | no | scalar default | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `valid_to` | `timestamp_utc` | yes | null means open interval | yes | no | scalar default | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `known_from` | `timestamp_utc` | yes | null when unknown and `090` permits | yes | no | scalar default | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `known_to` | `timestamp_utc` | yes | null means open knowledge interval | yes | no | scalar default | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `projection_profile_id` | `cadastre_id` | yes | none | no | no | scalar default | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `graph_delta_set_id` | `cadastre_id` | yes | none | no | no | scalar default | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |

### CoreRecordIdPolicy

ID algorithms must serialize ordered ID inputs as a JSON array with `CanonicalJSON`, hash the UTF-8 canonical bytes with SHA-256, and return `<prefix>_<lowercase_hex_digest>`. Digest truncation is forbidden. If different canonical input bytes produce the same ID, validation must emit the record-specific collision error and commit no record.

The complete canonical `one_of` object, including `kind` and the matching value member, is serialized into ID inputs exactly as materialized by `CanonicalJSON`. Implementations must not hash only the value member or replace the `one_of` object with an owner-local string.

Any pre-closure record that used inferred `one_of` kinds, inferred value members, inferred null behavior, or inferred artifact-class pairing is not promotion-eligible. It must be replayed and rekeyed under the closed `schema_version`.

| Record or algorithm | Prefix | Ordered ID inputs | Missing-input behavior | Null sentinel behavior | Collision error | Schema-version behavior |
| --- | --- | --- | --- | --- | --- | --- |
| `ComputeRawRecordId` | `raw` | `feed_profile_id`, `read_target_kind`, `source_dataset`, `scope_key_set`, `supplier_batch_or_object_identity`, `source_native_record_identity`, `canonical_payload_hash`, `import_profile_version` | Required inputs reject with `CORE_REQUIRED_FIELD_MISSING`. | `source_native_record_identity` materializes JSON null when absent. | `RAW_RECORD_ID_COLLISION` | Input order change requires new `schema_version`. |
| `CadastreSilverObservation` | `obs` | `raw_evidence_refs`, `observation_type`, `mapping_bundle_id`, `source_scope`, `normalized_payload_checksum` | Reject. | None. | `SILVER_OBSERVATION_ID_COLLISION` | Input order change requires new `schema_version`. |
| `CanonicalEntity` | `cent` | `entity_type`, `creation_identity_decision_id`, `resolver_profile_id`, `identity_policy_version` | Reject. | None. | `CANONICAL_ENTITY_ID_COLLISION` | Input order change requires new `schema_version`. |
| `SourceAsset` | `sasset` | `source_scope`, `source_asset_type`, `source_native_identity`, `asset_generation_key` | Reject except nullable generation key. | `asset_generation_key` materializes JSON null when unavailable. | `SOURCE_ASSET_ID_COLLISION` | Input order change requires new `schema_version`. |
| `Identifier` | `ident` | `identifier_type`, `normalized_value`, `identifier_scope`, `valid_from`, `valid_to` | Reject except nullable interval endpoints. | Nullable interval endpoints materialize JSON null. | `IDENTIFIER_ID_COLLISION` | Input order change requires new `schema_version`. |
| `ComputeGoldFactKeyId` | `gfkey` | `fact_type`, `subject_ref`, `predicate`, `object_kind`, `object_value`, `valid_from`, `valid_to` | Reject except nullable valid endpoints. | Nullable valid endpoints materialize JSON null. | `GOLD_FACT_ID_COLLISION` | Key input order change requires new `schema_version`. |
| `ComputeGoldFactId` | `gfact` | `gold_fact_key_id`, `known_from`, `assertion_state`, `authority_profile_row_id`, `temporal_resolution_id`, `evidence_refs`, `correction_policy_id` | Reject. | None. | `GOLD_FACT_ID_COLLISION` | Immutable ID input order change requires new `schema_version`. |
| `ComputeEvidenceRefId` | `eref` | `artifact_class`, `artifact_id`, `artifact_checksum`, `evidence_role`, `field_paths` | Reject. | Empty `field_paths` is `[]`, not null. | `EVIDENCE_REF_ID_COLLISION` | Input order change requires new `schema_version`. |
| `GraphNodeDeltaShape` | `gnd` | Imported from `090.GraphNodeDeltaIdPolicy`. | Reject before graph apply. | As `090` defines. | `GRAPH_BACKEND_ID_FORBIDDEN` when backend ID is used; graph ID collision owner remains `090`. | Runtime delta ID input order change requires `090` profile/schema update. |
| `GraphEdgeDeltaShape` | `ged` | Imported from `090.GraphEdgeDeltaIdPolicy`. | Reject before graph apply. | As `090` defines. | `GRAPH_BACKEND_ID_FORBIDDEN` when backend ID is used; graph ID collision owner remains `090`. | Runtime delta ID input order change requires `090` profile/schema update. |

### CoreRecordChecksumPolicy

`record_checksum` is SHA-256 over `CanonicalJSON` bytes for the full materialized record, including `CommonRecordHeader` and body fields, excluding only `record_checksum`. Raw payload bytes must not be inlined into any core record checksum input; payload hashes and bounded refs are checksum inputs. Unknown fields outside declared extension maps fail before checksum computation.

The complete canonical `one_of` object, including `kind` and the matching value member, is included in record checksums exactly as materialized by `CanonicalJSON`. Implementations must not substitute owner-local display strings, backend IDs, raw source strings, or artifact labels for a closed `one_of` object in checksum input.

| Record | Included fields | Excluded fields | Extension-map checksum rule | Replay rule |
| --- | --- | --- | --- | --- |
| All core records | `CommonRecordHeader` except `record_checksum`, plus every body field whose schema row has `checksum_input = yes`. | `record_checksum`; raw payload bytes; diagnostics not present in the schema. | Declared extension maps serialize in lexical key order after owner validation. | Same materialized record bytes must produce byte-identical checksum across replay and independent implementations. |

### CoreRecordValidationAlgorithm

```text
ValidateCoreRecord(record, schema, environment):
1. Reject when record_type is absent or not an exported record token.
2. Resolve CoreRecordSchema by record_type and schema_version.
3. Reject unknown fields outside declared extension maps.
4. Materialize required defaults exactly as declared.
5. Reject omitted required fields.
6. Reject explicit null where null_allowed = no.
7. Validate scalar type, container type, bounds, enum membership, and one_of tag/value pairing.
7a. Apply validation precedence from `CoreRecordValidationPrecedence` when more than one failure is present.
8. Canonically sort arrays and maps that declare sort keys.
9. Validate extension-map ownership and redaction ownership.
10. Compute expected record_id from CoreRecordIdPolicy.
11. Reject record_id mismatch.
12. Compute expected record_checksum excluding record_checksum.
13. Reject checksum mismatch.
14. Reject production records with null version_manifest_id.
15. Return canonical bytes, record_id, record_checksum, and deterministic validation result.
```

`CoreRecordValidationAlgorithm` is exported as `ValidateCoreRecord` for import by downstream specs.

### CoreRecordValidationPrecedence

When a record has multiple validation defects, `ValidateCoreRecord` must emit the first applicable class in this total order. The validation result may include non-primary diagnostics only after the primary error is selected.

| Order | Failure class | Primary error |
| ---: | --- | --- |
| 1 | unknown field outside declared extension map | `CORE_UNKNOWN_FIELD` |
| 2 | required field omitted | `CORE_REQUIRED_FIELD_MISSING` |
| 3 | explicit null where forbidden | `CORE_NULL_FORBIDDEN` |
| 4 | scalar or container type invalid | `CORE_FIELD_TYPE_INVALID` |
| 5 | scalar bound, byte bound, length, or decimal precision invalid | `CORE_FIELD_BOUNDS_INVALID` or `CORE_DECIMAL_PRECISION_INVALID` |
| 6 | enum token not in closed owner enum | `CORE_ENUM_INVALID` |
| 7 | `one_of` tag/value invalid | `CORE_ONE_OF_INVALID` |
| 8 | computed ID mismatch | `CORE_RECORD_ID_MISMATCH` |
| 9 | checksum mismatch | `CORE_RECORD_CHECKSUM_MISMATCH` |
| 10 | required production version manifest missing | `VERSION_MANIFEST_INCOMPLETE` |

### ComputeRawRecordId

```text
ComputeRawRecordId(inputs):
1. Read the ordered inputs from `CoreRecordIdPolicy.ComputeRawRecordId`.
2. Normalize strings to NFC, enum tokens to lowercase snake case, hashes to lowercase hex, and objects through `CanonicalJSON`.
3. Materialize `source_native_record_identity` as JSON null when absent.
4. Serialize the ordered input array with `CanonicalJSON`.
5. Return `raw_` plus SHA-256 lowercase hex over the UTF-8 canonical bytes.
6. If a prior different input array has the same ID, emit `RAW_RECORD_ID_COLLISION` and commit no record.
```

### ComputeGoldFactKeyId

```text
ComputeGoldFactKeyId(fact):
1. Require fact_type, subject_ref, predicate, object_kind, object_value, valid_from, and valid_to after temporal resolution.
2. Materialize nullable valid interval endpoints as JSON null.
3. Serialize the ordered array from `CoreRecordIdPolicy.ComputeGoldFactKeyId` with `CanonicalJSON`.
4. Return `gfkey_` plus SHA-256 lowercase hex over the UTF-8 canonical bytes.


`ComputeGoldFactKeyId` must validate `subject_ref` through `GoldFactSubjectRefKindRegistry`, validate `object_value` through `GoldFactObjectValueKindRegistry`, and require `object_kind == object_value.kind` before serialization. The ID input contains the complete canonical `subject_ref` and `object_value` objects, not only their value members.
```

### ComputeGoldFactId

```text
ComputeGoldFactId(fact):
1. Require gold_fact_key_id, known_from, assertion_state, authority_profile_row_id, temporal_resolution_id, evidence_refs, and correction_policy_id.
2. Sort evidence_refs lexically by ref ID.
3. Serialize the ordered array from `CoreRecordIdPolicy.ComputeGoldFactId` with `CanonicalJSON`.
4. Return `gfact_` plus SHA-256 lowercase hex over the UTF-8 canonical bytes.
5. Changing confidence, known_to, correction_refs, or absence_outcome affects record_checksum; it does not affect gold_fact_id unless an accepted future schema version changes this policy.
```

### ComputeEvidenceRefId

```text
ComputeEvidenceRefId(evidence_ref):
1. Require artifact_class, artifact_id, artifact_checksum, evidence_role, and field_paths.
2. Sort field_paths lexically.
3. Reject inline payload bytes with `EVIDENCE_REF_RAW_PAYLOAD_FORBIDDEN`.
4. Serialize the ordered array from `CoreRecordIdPolicy.ComputeEvidenceRefId` with `CanonicalJSON`.
5. Return `eref_` plus SHA-256 lowercase hex over the UTF-8 canonical bytes.


`ComputeEvidenceRefId` must validate `artifact_id` through `EvidenceArtifactIdKindRegistry` and validate the `artifact_class`/`artifact_id.kind` pair through `EvidenceArtifactClassRegistry` before serialization. The ID input contains the complete canonical `artifact_id` object, not only its value member.
```

### CoreRecordErrorCodeSet

| Error code | Emitted when |
| --- | --- |
| `CORE_UNKNOWN_FIELD` | Unknown field appears outside a declared extension map. |
| `CORE_REQUIRED_FIELD_MISSING` | Required field is omitted. |
| `CORE_NULL_FORBIDDEN` | Field is explicit null where null is forbidden. |
| `CORE_FIELD_TYPE_INVALID` | Field has wrong scalar or container type. |
| `CORE_FIELD_BOUNDS_INVALID` | Field exceeds declared bound. |
| `CORE_DECIMAL_PRECISION_INVALID` | Decimal string is outside range, non-canonical, or has more fractional digits than the declared scale. |
| `CORE_ENUM_INVALID` | `enum_token` value is not in the closed owner enum. |
| `CORE_ONE_OF_INVALID` | `one_of` value has unknown kind, missing matching value member, multiple value members, or forbidden null. |
| `CORE_RECORD_ID_MISMATCH` | Supplied ID does not equal computed ID. |
| `CORE_RECORD_CHECKSUM_MISMATCH` | Supplied checksum does not equal computed checksum. |
| `CORE_SCHEMA_VERSION_UNSUPPORTED` | Record schema version is unknown or inactive. |
| `RAW_RECORD_ID_COLLISION` | Different raw ID inputs produce the same raw ID. |
| `SILVER_OBSERVATION_ID_COLLISION` | Different silver ID inputs produce the same observation ID. |
| `CANONICAL_ENTITY_ID_COLLISION` | Different entity ID inputs produce the same canonical entity ID. |
| `SOURCE_ASSET_ID_COLLISION` | Different source asset ID inputs produce the same source asset ID. |
| `IDENTIFIER_ID_COLLISION` | Different identifier ID inputs produce the same identifier ID. |
| `GOLD_FACT_ID_COLLISION` | Different gold fact ID inputs produce the same gold fact ID. |
| `EVIDENCE_REF_ID_COLLISION` | Different evidence-ref ID inputs produce the same evidence-ref ID. |
| `EVIDENCE_REF_RAW_PAYLOAD_FORBIDDEN` | `EvidenceRef` embeds raw payload bytes. |
| `GRAPH_BACKEND_ID_FORBIDDEN` | Backend-generated ID appears in graph delta shape, graph response, selector, evidence ref, replay key, drillback key, or pagination identity. |
| `EVIDENCE_ARTIFACT_CLASS_KIND_MISMATCH` | `EvidenceRef.artifact_class` does not permit the observed `artifact_id.kind` under `EvidenceArtifactClassRegistry`. |
| `CORE_SCHEMA_RUNTIME_OVERRIDE_FORBIDDEN` | Activation-controlled artifact attempts to modify stable core schema behavior. |

### CoreRecordErrorRegistryFragment

This owner fragment feeds `110.GenerateErrorCodeRegistry`. `110` owns the generated caller-visible registry. This table must not render API output by itself. Every row below is complete enough for `110.GenerateErrorCodeRegistry`. A future accepted owner specification update may narrow presentation through `110` overrides, but it must not remove caller-visible code, severity, retryability, owner, affected record type, field path, redaction state, or correlation fields.

| error_code | owner_spec | severity | retry_class | caller_visible_fields | audit_visible_fields | redaction_rule | owner_context_schema_ref | fixture_ref |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `CORE_UNKNOWN_FIELD` | `040` | `error` | `caller_correctable` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `040.CoreRecordErrorContext` | `error-registry-040-core-unknown-field` |
| `CORE_REQUIRED_FIELD_MISSING` | `040` | `error` | `caller_correctable` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `040.CoreRecordErrorContext` | `error-registry-040-core-required-field-missing` |
| `CORE_NULL_FORBIDDEN` | `040` | `error` | `caller_correctable` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `040.CoreRecordErrorContext` | `error-registry-040-core-null-forbidden` |
| `CORE_FIELD_TYPE_INVALID` | `040` | `error` | `caller_correctable` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `040.CoreRecordErrorContext` | `error-registry-040-core-field-type-invalid` |
| `CORE_FIELD_BOUNDS_INVALID` | `040` | `error` | `caller_correctable` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `040.CoreRecordErrorContext` | `error-registry-040-core-field-bounds-invalid` |
| `CORE_DECIMAL_PRECISION_INVALID` | `040` | `error` | `caller_correctable` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `040.CoreRecordErrorContext` | `error-registry-040-core-decimal-precision-invalid` |
| `CORE_ENUM_INVALID` | `040` | `error` | `caller_correctable` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `040.CoreRecordErrorContext` | `error-registry-040-core-enum-invalid` |
| `CORE_ONE_OF_INVALID` | `040` | `error` | `caller_correctable` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `040.CoreRecordErrorContext` | `error-registry-040-core-one-of-invalid` |
| `CORE_RECORD_ID_MISMATCH` | `040` | `error` | `caller_correctable` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `040.CoreRecordErrorContext` | `error-registry-040-core-record-id-mismatch` |
| `CORE_RECORD_CHECKSUM_MISMATCH` | `040` | `error` | `caller_correctable` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `040.CoreRecordErrorContext` | `error-registry-040-core-record-checksum-mismatch` |
| `CORE_SCHEMA_VERSION_UNSUPPORTED` | `040` | `blocked` | `policy_change_required` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `040.CoreRecordErrorContext` | `error-registry-040-core-schema-version-unsupported` |
| `RAW_RECORD_ID_COLLISION` | `040` | `error` | `retry_after_owner_repair` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `040.CoreRecordErrorContext` | `error-registry-040-raw-record-id-collision` |
| `SILVER_OBSERVATION_ID_COLLISION` | `040` | `error` | `retry_after_owner_repair` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `040.CoreRecordErrorContext` | `error-registry-040-silver-observation-id-collision` |
| `CANONICAL_ENTITY_ID_COLLISION` | `040` | `error` | `retry_after_owner_repair` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `040.CoreRecordErrorContext` | `error-registry-040-canonical-entity-id-collision` |
| `SOURCE_ASSET_ID_COLLISION` | `040` | `error` | `retry_after_owner_repair` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `040.CoreRecordErrorContext` | `error-registry-040-source-asset-id-collision` |
| `IDENTIFIER_ID_COLLISION` | `040` | `error` | `retry_after_owner_repair` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `040.CoreRecordErrorContext` | `error-registry-040-identifier-id-collision` |
| `GOLD_FACT_ID_COLLISION` | `040` | `error` | `retry_after_owner_repair` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `040.CoreRecordErrorContext` | `error-registry-040-gold-fact-id-collision` |
| `EVIDENCE_REF_ID_COLLISION` | `040` | `error` | `retry_after_owner_repair` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `040.CoreRecordErrorContext` | `error-registry-040-evidence-ref-id-collision` |
| `EVIDENCE_REF_RAW_PAYLOAD_FORBIDDEN` | `040` | `security_error` | `none` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.always_forbidden_sensitive_values` | `040.CoreRecordErrorContext` | `error-registry-040-evidence-ref-raw-payload-forbidden` |
| `GRAPH_BACKEND_ID_FORBIDDEN` | `040` | `security_error` | `none` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.always_forbidden_sensitive_values` | `040.CoreRecordErrorContext` | `error-registry-040-graph-backend-id-forbidden` |
| `EVIDENCE_ARTIFACT_CLASS_KIND_MISMATCH` | `040` | `error` | `caller_correctable` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.owner_context` | `040.CoreRecordErrorContext` | `error-registry-040-evidence-artifact-class-kind-mismatch` |
| `CORE_SCHEMA_RUNTIME_OVERRIDE_FORBIDDEN` | `040` | `security_error` | `none` | `110.StandardErrorCallerFields` | `110.StandardErrorAuditFields` | `110.StandardErrorRedactionRule.security_boundary` | `040.CoreRecordErrorContext` | `error-registry-040-core-schema-runtime-override-forbidden` |

`CORE_ONE_OF_INVALID` fixture coverage must include subject-ref, object-value, and evidence artifact failures. It must distinguish unknown kind, missing member, extra member, forbidden null, omitted field, object-kind mismatch, raw payload, backend ID, and artifact-class/kind mismatch.

### CoreRecordErrorContext

`CoreRecordErrorContext` is the owner context schema for `040` core record validation, canonical serialization, ID, checksum, one-of, evidence-ref, and graph-delta-shape registry rows. It satisfies `110.OwnerErrorContextMinimumSchema` and must not include raw payload bytes, private bindings, credentials, backend-generated IDs, source-native identity values, or artifact payload bytes in caller-visible context.

| Field | Required | Rule |
| --- | ---: | --- |
| `context_schema_version` | Yes | Immutable `040` context schema version. |
| `owner_spec` | Yes | Must be `040`. |
| `error_code` | Yes | Must match the generated registry row. |
| `failure_class` | Yes | Closed token: `core_schema`, `canonical_json`, `scalar`, `enum`, `one_of`, `record_id`, `record_checksum`, `evidence_ref`, `graph_delta_shape`, `raw_payload_boundary`, `backend_id_boundary`, or `runtime_override`. |
| `operation` | Yes | Core record validation, canonical serialization, ID computation, checksum computation, one-of validation, evidence-ref validation, or graph-delta shape validation. |
| `affected_record_type` | Yes | Core record type, evidence ref, graph node delta shape, graph edge delta shape, scalar field, enum field, one-of member, or checksum artifact. |
| `field_path` | Yes | Exact field path when applicable; null for artifact-wide failures. |
| `artifact_refs` | Yes | Canonically sorted refs to core schema registry rows, scalar policy rows, one-of registry rows, evidence artifact registry rows, validation fixtures, input record refs, or version manifest refs consulted by the error; empty only when no artifact was consulted. |
| `validation_refs` | Yes | Exact `120` core record validation fixture refs. |
| `redaction_classes` | Yes | Map every nested owner-context field to one `110.ErrorRedactionClassMatrix` class. Raw payload bytes, private bindings, credentials, backend IDs, source-native identity values, raw artifact payloads, and raw graph backend values must map to `always_forbidden`. |
| `blocking_reason` | Yes when generated row severity is `blocked` | Bounded reason; otherwise null or omitted. |
| `record_id` | No | Required when a persisted or candidate record ID was evaluated and the caller is authorized for the ref. |
| `schema_version` | No | Required for schema-version failures. |
| `expected_checksum_ref` | No | Required when a checksum mismatch was evaluated and the checksum ref is authorized. |
| `observed_checksum_ref` | No | Required when a checksum mismatch was evaluated and the checksum ref is authorized. |
| `one_of_kind` | No | Required for one-of validation failures. |

### CorePrivateBindingLeakHandoff

`040` no longer owns a generated `PRIVATE_BINDING_LEAK` registry row. Core validation may detect concrete private vendor/source bindings, credentials, route names, tenant host lists, or environment-specific inventories in core-shaped public artifacts, but the emitted owner error must be imported `010.PRIVATE_BINDING_LEAK` with `010.BoundaryErrorContext`.

Core shape errors remain `040` errors. `EVIDENCE_REF_RAW_PAYLOAD_FORBIDDEN` and `GRAPH_BACKEND_ID_FORBIDDEN` remain `040` security errors because they reject core evidence and graph-delta shape violations rather than public/private source-boundary violations.

### Core schema security and extensibility constraints

| Constraint | Required behavior |
| --- | --- |
| Private binding leakage | `RawRecord` may store vendor-neutral source category and redacted access-ref hashes only. Concrete private routes, credentials, tenant host lists, and private source inventories must fail with `PRIVATE_BINDING_LEAK`. |
| Raw payload boundary | Raw payload bytes must remain in raw/bronze lakehouse storage. `RawRecord` stores hashes, byte lengths, and bounded refs. `EvidenceRef` and graph delta shapes must not inline payload bytes. |
| External schema authority | OCSF may validate `CadastreSilverObservation.normalized_fields`; it must not define Cadastre identity, authority, omission, temporal, graph, or gold semantics. |
| Forward compatibility | Unknown fields fail closed. New fields require a new `schema_version`, migration row, and validation fixtures. |
| Extensibility | Extension maps must be owner-declared. `040` validates `SourceExtensionFieldRuleShape` bytes and unknown-field behavior; `050` validates activation and path permission. Undeclared extension fields fail before production output. |
| Minimal coupling | `040` defines record bytes and validation behavior only. It must not define parser mechanisms, resolver scoring, graph backend products, package runtimes, or source-specific mappings. |
| Closed one-of boundary | `GoldFact.subject_ref`, `GoldFact.object_value`, and `EvidenceRef.artifact_id` must use the closed registries in this spec. Downstream specs may restrict allowed subsets but must not add `kind` values, change value members, alter null behavior, or replace canonical one-of bytes with local strings. |

### Core schema volatility boundary

`CanonicalJSON`, `OmissionState`, `ScalarType`, `CoreRecordFieldRow`, core record schemas, ID policies, checksum policies, validation precedence, and core error codes are `stable_core_contract` material owned by `040`.

Activation-controlled artifacts must not add, remove, rename, reinterpret, or override core fields. They must not change ID inputs, checksum inputs, nullability, omission semantics, redaction ownership, scalar bounds, canonical sort keys, or extension-map policy.

Source-specific values may appear only in declared extension maps whose activation is owned by `050`. `EvidenceRef` may reference activation-controlled artifacts only by artifact class, artifact ID, checksum, evidence role, and field paths; it must not inline artifact payload bytes.

If a record carries an activation-artifact value outside a declared extension map or allowed ref field, validation must fail before persistence using `CORE_UNKNOWN_FIELD`, `CORE_FIELD_TYPE_INVALID`, the owner-specific extension error, or `CORE_SCHEMA_RUNTIME_OVERRIDE_FORBIDDEN` when the attempted override itself is observable.

| Error code | Emitted when |
| --- | --- |
| `CORE_SCHEMA_RUNTIME_OVERRIDE_FORBIDDEN` | An activation-controlled artifact attempts to change core fields, omission states, ID inputs, checksum inputs, scalar bounds, nullability, canonical sort keys, or extension-map policy. |

### Core schema closure rows

`040` closes decimal precision, validation precedence, and the core `one_of` kind universe for `GoldFact.subject_ref`, `GoldFact.object_value`, and `EvidenceRef.artifact_id`. Owner-specific enum value sets and predicate-specific or artifact-class-specific subsets remain owner-routed and must be represented in `120` validation rows; downstream implementations must not invent missing enum tokens or union members.

| Closure row | Required behavior | Closure state |
| --- | --- | --- |
| decimal precision | Use `DecimalPrecisionPolicy.confidence_0_1`; no rounding; persisted value has six fractional digits. | closed_local |
| one-of members | Use `CoreOneOfRegistry`, `GoldFactSubjectRefKindRegistry`, `GoldFactObjectValueKindRegistry`, `EvidenceArtifactIdKindRegistry`, and `EvidenceArtifactClassRegistry`. | closed_local |
| enum registries | Use `CoreEnumRegistryOwnership`; any field lacking a closed owner enum blocks authoritative promotion for affected outputs. | blocked_validation |
| identity enum closure | `070.Identity closed enum tables` must pass `120` validation before authoritative promotion when identity output is in scope. | blocked_validation |
| private binding error | Core validators may detect private binding leaks, but the generated owner row is imported `010.PRIVATE_BINDING_LEAK`; `040` must not define a duplicate registry row. | imported_from_010 |
| core schema volatility | Core schemas, omission states, ID/checksum policies, scalar bounds, validation precedence, and core one-of registries are stable core; activation artifacts cannot alter them. | closed_local |

## Assertion States

`GoldFact.assertion_state` must be a closed enum. Default MVP states are:

| State | Meaning |
| --- | --- |
| `active` | Fact is asserted current for the query interval. |
| `stale` | Fact is retained but stale under an active staleness policy. |
| `retracted` | Fact has been explicitly retracted by authorized evidence. |
| `superseded` | Fact was replaced by correction or stronger evidence. |
| `conflicted` | Concurrent evidence prevents a single resolved assertion. |
| `unknown` | Evidence exists but does not authorize presence or absence. |
| `no_op` | Candidate derivation intentionally emitted no fact. |

`040` owns the assertion-state token vocabulary only. `080` owns all correction transitions and no-op emission behavior. `no_op` must not be treated as active presence, authorized absence, graph eligibility, compliance pass/fail, or source completeness. Default no-op behavior is represented by `080.GoldFactChangeSet.operation = no_op_duplicate`; a persisted `GoldFact` with `assertion_state = no_op` is forbidden unless a future active owner row defines output eligibility and validation rows for that exact use.

Graph, API, export, and analysis paths must treat no-op correction evidence as no output change unless their owner specs import an active `080` no-op behavior row that permits a narrower diagnostic rendering. `090` must not project no-op facts or no-op changesets into graph output by default.

## Graph Delta Primitive Shapes

This spec owns only the primitive record shape of `GraphNodeDeltaShape` and `GraphEdgeDeltaShape`. Projection rules, graph apply, backend mapping, traversal, runtime delta ID inputs, and query behavior are owned by `090`. A graph delta primitive shape that contains a backend-generated ID must fail with `GRAPH_BACKEND_ID_FORBIDDEN` before graph apply, query response materialization, evidence ref generation, replay, drillback, or pagination identity is emitted.

### GraphDeltaOperationEnum

`GraphNodeDeltaShape.delta_operation` and `GraphEdgeDeltaShape.delta_operation` use the closed graph delta operation token set below. Projection semantics remain owned by `090`; this table owns only primitive token validity.

| Operation token | Primitive validity | Additional owner rule |
| --- | --- | --- |
| `upsert` | Allowed for node and edge deltas. | `090` must define projection and apply ordering. |
| `update_visibility` | Allowed for node and edge deltas. | `090.GraphObjectOutputEligibilityRow` must permit the visibility transition. |
| `expire` | Allowed for node and edge deltas. | Requires `060` authorization before projection/apply when absence-sensitive. |
| `cleanup` | Allowed for node and edge deltas. | Requires `060` authorization before projection/apply. |
| `no_op` | Allowed only as a persisted diagnostic delta when `090` declares no backend mutation. | Must not call the graph backend. |
| all other tokens | Rejected. | Emit `CORE_FIELD_TYPE_INVALID` before graph apply. |

Graph delta operation values are canonically serialized as lowercase snake-case enum tokens. Null, omission, unknown operation, or backend-native operation aliases fail before graph delta ID or checksum computation.

## Canonical Model Contract Details

### CanonicalChecksumPolicy

| Field | Rule |
| --- | --- |
| Hash algorithm | `sha256` for MVP. |
| Digest encoding | lowercase hexadecimal. |
| Byte input | UTF-8 bytes of `CanonicalJSON` unless the record declares a binary payload hash. |
| Field inclusion | Include every field listed by `CoreRecordChecksumPolicy`; ID inputs and checksum inputs are not the same contract. |
| Volatile-field exclusion | Exclude processing timestamps, diagnostics ordering artifacts, and correlation IDs unless the owner declares them output-affecting. |
| Extension-field handling | Include declared extension maps in lexical key order; reject undeclared unknown fields. |
| Versioning | Prefix checksum policy version in records whose checksum can affect replay or activation. |

### CanonicalIdPolicy

| Field | Rule |
| --- | --- |
| ID namespace | Each record type declares a stable prefix. |
| Input order | Each owner table declares ordered inputs. |
| Serialization | Inputs serialize as an ordered array using `CanonicalJSON`. |
| Digest truncation | Forbidden for MVP unless the owner table explicitly permits a longer collision check. |
| Collision behavior | Different input bytes with same ID emit owner-specific collision error and commit no record. |
| Version prefix | Include identity policy version when changing input order or normalization. |

### CanonicalSortKey

Arrays whose order affects IDs, checksums, replay equivalence, graph deltas, or validation output must declare a canonical sort key. Arrays without a declared sort key preserve input order and the input order becomes output-affecting.

### UnknownFieldPolicy

Unknown fields are rejected unless the owning record declares an extension map. `050` owns runtime activation rules for source extension fields; this spec owns the primitive unknown-field rule.

### Record identity policy table

`CoreRecordIdPolicy` in the `Core Record Schema Registry` is the owning ID policy. The table below is retained only as an import map and must not be used to restate or override the registry.

| Record | Owning ID contract | Runtime contributor | Additional constraint owner |
| --- | --- | --- | --- |
| `RawRecord` | `040.ComputeRawRecordId` | `020` supplies feed, target, manifest, source dataset, scope, supplier identity, payload hash, and import profile inputs. | `020` |
| `CadastreSilverObservation` | `040.CoreRecordIdPolicy` | `050` supplies mapping and normalized payload inputs. | `050` |
| `CanonicalEntity` | `040.CoreRecordIdPolicy` | `070` supplies creation decision, resolver, policy, attachment, merge, and split decision refs as resolver-owned inputs. | `070` |
| `SourceAsset` | `040.CoreRecordIdPolicy` | `070` supplies source scope, source-native identity, attachment decision refs, and split decision refs as resolver-owned inputs. | `070` |
| `Identifier` | `040.CoreRecordIdPolicy` | `070` supplies typed identifier scope, validity inputs, attachment decision refs, merge decision refs, and split decision refs as resolver-owned inputs. | `070` |
| `GoldFact` | `040.ComputeGoldFactKeyId` and `040.ComputeGoldFactId` | `060` and `080` supply authority, temporal, evidence, and correction inputs. | `060`, `080` |
| `EvidenceRef` | `040.ComputeEvidenceRefId` | Owner specs supply referenced artifact metadata only; raw payload bytes are forbidden. | `040`, `110` |
| `GraphNodeDeltaShape` and `GraphEdgeDeltaShape` | Shape validation in `040`; runtime delta ID policies imported from `090`. | `090` supplies projection/apply semantics. | `090` |

### Omission-state crosswalk

| Omission state | Allowed owners | API label import | Source authority effect | Graph projection effect | Compliance export effect |
| --- | --- | --- | --- | --- | --- |
| `observed_present` | `040`, owner facts | `110` | may support authority if `060` permits | may project if `090` permits | may export if owner permits |
| `observed_empty` | `040`, owner facts | `110` | no absence by itself | owner-defined | owner-defined |
| `not_provided` | `040`, `060` | `110` | unknown by default | no negative edge | unknown/not checked |
| `not_applicable` | `040`, `060`, `110` | `110` | no absence unless policy says | no edge by default | not applicable |
| `permission_limited` | `040`, `060`, `110` | `110` | blocks absence | no negative edge | permission limited |
| `source_unavailable` | `040`, `060`, `110` | `110` | blocks absence | no negative edge | error/unknown |
| `unsupported` | `040`, owner spec | `110` | no authority | no edge unless explicit | unsupported/unknown |
| `malformed` | `040`, `050` | `110` | no authority | no edge | error |
| `redacted` | `040`, `110` | `110` | hidden from caller only | redaction policy | redacted |

### Acceptance Criteria

| ID | Criterion |
| --- | --- |
| `040-API-HANDOFF-AC-001` | API handoff fixtures distinguish `conflicted`, `ambiguous`, assertion-state `unknown`, absence-outcome `unknown`, and `no_op`; none is treated as pass, fail, authorized absence, graph cleanup, or source deletion by default. |
| `040-CLEANUP-AC-001` | No banned reference class remains. |
| `040-CLEANUP-AC-002` | Canonical JSON remains deterministic for IDs, checksums, replay equivalence, package activation, validation output, graph delta identity, and audit evidence. |
| `040-CLEANUP-AC-003` | Omission states remain explicit and cannot be inferred from optionality, nullability, OCSF absence, CIM absence, or source row absence alone. |
| `040-SCHEMA-PATCH-AC-001` | Every exported core record has a field schema using the required `CoreRecordFieldRow` columns. |
| `040-SCHEMA-PATCH-AC-002` | Every exported core record has ID input order, prefix, canonicalization, missing behavior, null sentinel behavior, checksum inclusion, checksum exclusion, and collision behavior. |
| `040-SCHEMA-PATCH-AC-003` | Every nullable field declares the meaning of null, and omission remains distinct from null. |
| `040-SCHEMA-PATCH-AC-004` | Every array or map that affects IDs, checksums, replay, graph deltas, validation output, or audit output declares deterministic ordering. |
| `040-SCHEMA-PATCH-AC-005` | `GoldFact` separates `gold_fact_key_id` from immutable `gold_fact_id`. |
| `040-SCHEMA-PATCH-AC-006` | `EvidenceRef` and graph delta shapes reject raw payload bytes. |
| `040-IDENTITY-ENUM-AC-001` | Identity enum closure is validated by `120` and inconsistent or absent `070` identity enum closure blocks authoritative promotion. |
| `040-SCHEMA-PATCH-AC-007` | Backend-generated IDs fail in graph delta shapes with `GRAPH_BACKEND_ID_FORBIDDEN`. |
| `040-SCHEMA-PATCH-AC-008` | `ValidateCoreRecord` produces byte-identical canonical bytes and checksums for the same inputs across two independent implementations. |
| `040-VOLATILITY-AC-001` | Activation artifacts cannot alter core record fields or ID/checksum behavior. |
| `040-VOLATILITY-AC-002` | Source-extension fields remain owner-declared through `050`. |
| `040-VOLATILITY-AC-003` | Core schemas remain byte-stable for the same input after changing only an activation-controlled mapping/profile artifact. |
| `040-FACT-ABSENCE-AC-001` | Every `GoldFact.absence_outcome` token is either null or one closed `FactAbsenceOutcome` value. |
| `040-FACT-ABSENCE-AC-002` | Parser-generated or mapping-generated absence outcomes fail before persistence. |
| `040-FACT-ABSENCE-AC-003` | Null absence outcome is not rendered as `unknown` without `110` owner mapping. |
| `040-FACT-ABSENCE-CLOSURE-AC-001` | A `GoldFact` with non-null `absence_outcome` fails validation or export-readiness when the producing `VersionManifest` lacks the matching `060.AbsenceDerivationResult` ref. |
| `040-FACT-ABSENCE-CLOSURE-AC-002` | `authorized_absent` and `authorized_not_observed` are not treated as authorized negative output unless the referenced `060.AbsenceDerivationResult` authorizes the exact predicate, scope, and requested effect. |
| `040-CONFIDENCE-CANONICAL-AC-001` | Confidence-only correction fixtures compare canonical six-fractional-digit confidence bytes and reject non-canonical persisted spellings. |
| `040-NOOP-ASSERTION-AC-001` | A `no_op` assertion state or `080.no_op_duplicate` changeset is not rendered as active presence, authorized absence, graph eligibility, compliance pass/fail, or source completeness. |
| `040-SOURCE-EXT-SHAPE-AC-001` | A minimal valid `SourceExtensionFieldRuleShape` fixture validates with byte-stable canonical ordering and checksum. |
| `040-SOURCE-EXT-SHAPE-AC-002` | Missing required, unknown-field, non-canonical ordering, invalid bounds, and checksum-replay fixtures fail or pass exactly as declared by `SourceExtensionFieldRuleShape`. |
| `040-SOURCE-EXT-SHAPE-AC-003` | Invalid source-extension rule shape is rejected before `050` activation behavior evaluates the rule. |
| `040-GRAPH-SOURCE-REF-AC-001` | MVP graph node deltas accept `canonical_entity_ref` source refs and reject null, generic external graph refs, and inactive structural refs. |
| `040-GRAPH-SOURCE-REF-AC-002` | MVP observed-connection edge deltas accept `gold_fact_ref` source refs and reject null, generic external graph refs, and structural refs. |
| `040-GRAPH-CONFIDENCE-AC-001` | Graph edge confidence uses `DecimalPrecisionPolicy.confidence_0_1` and rejects non-canonical persisted spellings or more than six fractional digits. |
| `040-GRAPH-DELTA-OPERATION-AC-001` | Graph delta operations are limited to `upsert`, `update_visibility`, `expire`, `cleanup`, and `no_op`; all other tokens fail before apply. |
| `040-CORE-ONEOF-SUBJECT-AC-001` | `GoldFact.subject_ref` accepts only `canonical_entity_ref`, `source_asset_ref`, and `identifier_ref`, each with exactly one matching value member, and rejects raw records, silver observations, evidence refs, graph/backend IDs, package IDs, telemetry IDs, source-native strings, unknown kinds, missing members, extra members, null, and omission. |
| `040-CORE-ONEOF-OBJECT-AC-001` | `GoldFact.object_value` accepts only the closed object-value kind set, requires `object_kind == object_value.kind`, and rejects identity-like strings when predicate contracts require reference kinds. |
| `040-CORE-ONEOF-NULL-AC-001` | `null_value` is distinct from omission and is valid only when `080.GoldFactPredicateContractRow.null_object_policy = allowed`; forbidden null fails before `gold_fact_key_id` computation. |
| `040-CORE-ONEOF-EVIDENCE-AC-001` | `EvidenceRef.artifact_id` accepts only `cadastre_record_ref`, `activation_artifact_ref`, `lakehouse_artifact_ref`, and `external_artifact_ref`, requires exactly one matching value member, requires `artifact_checksum`, and rejects raw payload bytes before `evidence_ref_id` computation. |
| `040-EVIDENCE-ARTIFACT-CLASS-AC-001` | `EvidenceRef.artifact_class` and `artifact_id.kind` pairing is total for declared artifact-class families, and mismatches fail with `EVIDENCE_ARTIFACT_CLASS_KIND_MISMATCH`. |
| `040-CORE-ONEOF-ID-CHECKSUM-AC-001` | Complete canonical `one_of` objects, including `kind` and matching value member, are included in ID and checksum inputs; hashing only value members or owner-local strings fails validation. |
| `040-CORE-ONEOF-MIGRATION-AC-001` | Pre-closure records using inferred kinds, inferred members, inferred null behavior, or inferred artifact-class pairing are not promotion-eligible and must be replayed and rekeyed under the closed schema version. |
| `040-ERROR-FRAGMENT-CORE-AC-001` | `040.CoreRecordErrorRegistryFragment` has no `TODO:` cells and generates deterministic `110.ErrorCodeRegistryRow` values for every `040.CoreRecordErrorCodeSet` code. |
| `040-EXPORT-ALIAS-AC-001` | Every downstream import of a `040.*Schema`, one-of registry, evidence artifact registry, or `ComputeEvidenceRefId` resolves to exactly one exported name in this file. |
| `040-EXPORT-ALIAS-AC-002` | Schema aliases validate as aliases to `CoreRecordSchema` row families and do not create duplicate field definitions. |
| `040-COMPUTE-EVIDENCE-REF-ID-AC-001` | `ComputeEvidenceRefId` is callable by exact contract name and rejects raw payload bytes before ID computation. |

### Structured input evidence acceptance criteria

| ID | Criterion |
| --- | --- |
| `040-STRUCTURED-INPUT-EVIDENCE-AC-001` | Structured-input evidence uses only existing `EvidenceRef.artifact_id.kind` values, rejects raw payload bytes, rejects mutable labels as checksums, validates class/kind pairing, and requires `VersionManifest` inclusion for output-affecting evidence. |
| `040-STRUCTURED-INPUT-EVIDENCE-AC-002` | Maintenance tool contracts, tool invocations, template contracts, CI contracts, publication manifests, sync records, and repository groups validate only through allowed artifact-class/kind pairs and checksums. |
| `040-STRUCTURED-INPUT-EVIDENCE-AC-003` | Tool logs, CI logs, repository URLs, branch names, raw paths, hook payloads, generated artifact bytes, and raw structured-input bytes are rejected when inlined in `EvidenceRef`. |

## Definition of Done

| ID | Criterion |
| --- | --- |
| `040-SCOPE-SELECTOR-SERIALIZATION-AC-001` | Selector input order does not affect normalized selector bytes or selector checksum after `030.NormalizeScopeSelector` and `040.CanonicalJSON` are applied. |
| `040-SCOPE-SELECTOR-SERIALIZATION-AC-002` | A `040` schema row containing `activation_scope` rejects selector bytes that fail `030.ActivationScope` validation before core record checksum computation. |
| `040-AC-001` | Two implementations serialize, hash, compare, and validate every core record shape identically. |
| `040-AC-002` | Omitted, null, empty, redacted, permission-limited, unknown, and unsupported values are distinguishable in serialized output. |
| `040-AC-003` | Graph delta primitive records cannot be created from backend internal IDs. |
| `040-AC-004` | Every `GoldFact` carries valid-time, known-time, assertion-state, evidence, confidence, and authority references. |
| `040-AC-005` | Unknown fields are rejected unless declared through a namespaced extension field rule owned by `050`. |
| `040-AC-007` | `SourceExtensionFieldRuleShape` validates primitive row bytes while `050` remains the sole owner of runtime source-extension permission. |
| `040-AC-006` | The spec cannot become authoritative while any `TODO:` row remains in the core schema registry, ID policy, enum registry, one-of registry, decimal precision policy, or validation fixture set. |
| `040-AC-008` | `GoldFact.subject_ref`, `GoldFact.object_value`, and `EvidenceRef.artifact_id` use closed one-of registries with deterministic value members, null behavior, ID inputs, checksum inputs, and validation fixtures. |
| `040-AC-009` | Every `EvidenceRef.artifact_class` in declared scope maps to exactly one allowed `EvidenceRef.artifact_id.kind` or fails before evidence-ref ID computation. |

## Open Questions

Open questions marked `TODO:` block authoritative status for the affected contract. A downstream implementation must not resolve a `TODO:` by inference.
