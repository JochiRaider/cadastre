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
| `subject_ref` | `one_of` | yes | none | no | no | closed union declared by fact type | n/a | ordered:2 for key | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `predicate` | `enum_token` | yes | none | no | no | scalar default | n/a | ordered:3 for key | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `object_kind` | `enum_token` | yes | none | no | no | scalar default | n/a | ordered:4 for key | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `object_value` | `one_of` | yes | none | no | no | closed union declared by object_kind | n/a | ordered:5 for key | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `valid_from` | `timestamp_utc` | yes | null when valid start is unknown and policy permits | yes | no | scalar default | n/a | ordered:6 for key | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `valid_to` | `timestamp_utc` | yes | null means open valid interval | yes | no | scalar default | n/a | ordered:7 for key | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `known_from` | `timestamp_utc` | yes | none | no | no | scalar default | n/a | ordered:2 for immutable ID | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `known_to` | `timestamp_utc` | yes | null means open knowledge interval; effective closure materializes through `080.GoldFactChangeSet` | yes | no | scalar default | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `assertion_state` | `enum_token` | yes | none | no | no | scalar default | n/a | ordered:3 for immutable ID | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `confidence` | `decimal_string` | yes | none | no | no | 0 through 1 inclusive; TODO: precision scale must be closed before authoritative promotion | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
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
| `artifact_id` | `one_of` | yes | none | no | no | `cadastre_id` or `external_ref_id` | n/a | ordered:2 | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
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
| `confidence` | `decimal_string` | yes | null when edge type has no confidence policy | yes | no | 0 through 1 inclusive; TODO: precision scale must be closed before authoritative promotion | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `valid_from` | `timestamp_utc` | yes | null when validity is unknown | yes | no | scalar default | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `valid_to` | `timestamp_utc` | yes | null means open interval | yes | no | scalar default | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `known_from` | `timestamp_utc` | yes | null when unknown and `090` permits | yes | no | scalar default | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `known_to` | `timestamp_utc` | yes | null means open knowledge interval | yes | no | scalar default | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `projection_profile_id` | `cadastre_id` | yes | none | no | no | scalar default | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |
| `graph_delta_set_id` | `cadastre_id` | yes | none | no | no | scalar default | n/a | no | yes | closed | `110` | `CORE_REQUIRED_FIELD_MISSING` | `CORE_FIELD_TYPE_INVALID` |

### CoreRecordIdPolicy

ID algorithms must serialize ordered ID inputs as a JSON array with `CanonicalJSON`, hash the UTF-8 canonical bytes with SHA-256, and return `<prefix>_<lowercase_hex_digest>`. Digest truncation is forbidden. If different canonical input bytes produce the same ID, validation must emit the record-specific collision error and commit no record.

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
```

### CoreRecordErrorCodeSet

| Error code | Emitted when |
| --- | --- |
| `CORE_UNKNOWN_FIELD` | Unknown field appears outside a declared extension map. |
| `CORE_REQUIRED_FIELD_MISSING` | Required field is omitted. |
| `CORE_NULL_FORBIDDEN` | Field is explicit null where null is forbidden. |
| `CORE_FIELD_TYPE_INVALID` | Field has wrong scalar or container type. |
| `CORE_FIELD_BOUNDS_INVALID` | Field exceeds declared bound. |
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

### Core schema security and extensibility constraints

| Constraint | Required behavior |
| --- | --- |
| Private binding leakage | `RawRecord` may store vendor-neutral source category and redacted access-ref hashes only. Concrete private routes, credentials, tenant host lists, and private source inventories must fail with `PRIVATE_BINDING_LEAK`. |
| Raw payload boundary | Raw payload bytes must remain in raw/bronze lakehouse storage. `RawRecord` stores hashes, byte lengths, and bounded refs. `EvidenceRef` and graph delta shapes must not inline payload bytes. |
| External schema authority | OCSF may validate `CadastreSilverObservation.normalized_fields`; it must not define Cadastre identity, authority, omission, temporal, graph, or gold semantics. |
| Forward compatibility | Unknown fields fail closed. New fields require a new `schema_version`, migration row, and validation fixtures. |
| Extensibility | Extension maps must be owner-declared. Undeclared extension fields fail before production output. |
| Minimal coupling | `040` defines record bytes and validation behavior only. It must not define parser mechanisms, resolver scoring, graph backend products, package runtimes, or source-specific mappings. |

### Core schema unresolved rows

The following rows block authoritative promotion until closed by the owning spec or explicit product governance.

| Blocker | Required closure |
| --- | --- |
| `TODO:decimal-precision` | `GoldFact.confidence` and graph edge `confidence` require an accepted decimal precision and rounding policy before production activation. |
| `TODO:one-of-members` | `GoldFact.subject_ref`, `GoldFact.object_value`, graph `source_object_ref`, and `EvidenceRef.artifact_id` require owner-specific closed union member tables before authoritative promotion. |
| `TODO:enum-registries` | Every `enum_token` field requires a closed owner enum before authoritative promotion. |

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

## Graph Delta Primitive Shapes

This spec owns only the primitive record shape of `GraphNodeDeltaShape` and `GraphEdgeDeltaShape`. Projection rules, graph apply, backend mapping, traversal, runtime delta ID inputs, and query behavior are owned by `090`. A graph delta primitive shape that contains a backend-generated ID must fail with `GRAPH_BACKEND_ID_FORBIDDEN` before graph apply, query response materialization, evidence ref generation, replay, drillback, or pagination identity is emitted.

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
| `CanonicalEntity` | `040.CoreRecordIdPolicy` | `070` supplies identity decision, resolver, and policy inputs. | `070` |
| `SourceAsset` | `040.CoreRecordIdPolicy` | `070` supplies source scope and source-native identity inputs. | `070` |
| `Identifier` | `040.CoreRecordIdPolicy` | `070` supplies typed identifier scope and validity inputs. | `070` |
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
| `040-CLEANUP-AC-001` | No banned reference class remains. |
| `040-CLEANUP-AC-002` | Canonical JSON remains deterministic for IDs, checksums, replay equivalence, package activation, validation output, graph delta identity, and audit evidence. |
| `040-CLEANUP-AC-003` | Omission states remain explicit and cannot be inferred from optionality, nullability, OCSF absence, CIM absence, or source row absence alone. |
| `040-SCHEMA-PATCH-AC-001` | Every exported core record has a field schema using the required `CoreRecordFieldRow` columns. |
| `040-SCHEMA-PATCH-AC-002` | Every exported core record has ID input order, prefix, canonicalization, missing behavior, null sentinel behavior, checksum inclusion, checksum exclusion, and collision behavior. |
| `040-SCHEMA-PATCH-AC-003` | Every nullable field declares the meaning of null, and omission remains distinct from null. |
| `040-SCHEMA-PATCH-AC-004` | Every array or map that affects IDs, checksums, replay, graph deltas, validation output, or audit output declares deterministic ordering. |
| `040-SCHEMA-PATCH-AC-005` | `GoldFact` separates `gold_fact_key_id` from immutable `gold_fact_id`. |
| `040-SCHEMA-PATCH-AC-006` | `EvidenceRef` and graph delta shapes reject raw payload bytes. |
| `040-SCHEMA-PATCH-AC-007` | Backend-generated IDs fail in graph delta shapes with `GRAPH_BACKEND_ID_FORBIDDEN`. |
| `040-SCHEMA-PATCH-AC-008` | `ValidateCoreRecord` produces byte-identical canonical bytes and checksums for the same inputs across two independent implementations. |

## Definition of Done

| ID | Criterion |
| --- | --- |
| `040-AC-001` | Two implementations serialize, hash, compare, and validate every core record shape identically. |
| `040-AC-002` | Omitted, null, empty, redacted, permission-limited, unknown, and unsupported values are distinguishable in serialized output. |
| `040-AC-003` | Graph delta primitive records cannot be created from backend internal IDs. |
| `040-AC-004` | Every `GoldFact` carries valid-time, known-time, assertion-state, evidence, confidence, and authority references. |
| `040-AC-005` | Unknown fields are rejected unless declared through a namespaced extension field rule owned by `050`. |
| `040-AC-006` | The spec cannot become authoritative while any `TODO:` row remains in the core schema registry, ID policy, enum registry, one-of registry, decimal precision policy, or validation fixture set. |


## Open Questions

Open questions marked `TODO:` block authoritative status for the affected contract. A downstream implementation must not resolve a `TODO:` by inference.
