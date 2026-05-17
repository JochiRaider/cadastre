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

## Core Records

| Record | Required identity | Required fields or field groups |
| --- | --- | --- |
| `RawRecord` | Deterministic raw record ID from feed scope, source dataset, supplier/object identity, source-native ID when present, payload hash, and import profile version. | Source identity, source dataset/type, native ID, event metadata, payload format, payload hash, collector/import versions, run lineage, duplicate/quarantine status, partition key. |
| `CadastreSilverObservation` | Observation ID from raw evidence refs, observation type, mapping bundle, source scope, and normalized payload checksum. | Raw refs, observation type, observed time, collected/imported time, normalized fields, source extension fields, field quality, lineage, confidence hints, flow role evidence when applicable. |
| `CanonicalEntity` | Product-owned canonical entity ID. | Entity type, lifecycle state, valid/known bounds, current status, source decision refs. |
| `SourceAsset` | Source-scoped asset ID. | Source/feed scope, source-native identity fields, source asset type, evidence refs, lifecycle evidence. |
| `Identifier` | Identifier ID from identifier type, normalized value, source scope, and validity interval. | Type, normalized value, scope, raw value refs, validity/known time, quality. |
| `GoldFact` | Fact ID from canonical subject, predicate, object/value, valid interval, assertion state, authority profile, and evidence set. | Subject, predicate, object/value, valid time, known time, assertion state, confidence, authority ref, evidence refs, correction refs. |
| `EvidenceRef` | Evidence ref ID from referenced artifact class, artifact ID, role, and checksum. | Artifact class, artifact ID, evidence role, raw/silver/gold link, checksum, visibility state. |

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

This spec owns only the primitive record shape of `GraphNodeDelta` and `GraphEdgeDelta`. Projection rules, graph apply, backend mapping, traversal, and query behavior are owned by `090`.

## Canonical Model Contract Details

### CanonicalChecksumPolicy

| Field | Rule |
| --- | --- |
| Hash algorithm | `sha256` for MVP. |
| Digest encoding | lowercase hexadecimal. |
| Byte input | UTF-8 bytes of `CanonicalJSON` unless the record declares a binary payload hash. |
| Field inclusion | Include every output-affecting field listed by the record identity policy. |
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

| Record | Owning ID inputs | Required or optional | Canonicalization | Missing behavior | Additional constraint owner |
| --- | --- | --- | --- | --- | --- |
| `RawRecord` | feed profile, target kind, dataset, scope, supplier identity, native ID, payload hash, import profile | mixed | `CanonicalJSON` ordered array | optional native ID materializes null | `020` |
| `CadastreSilverObservation` | raw refs, observation type, mapping bundle, source scope, normalized payload checksum | required | lexical refs and canonical checksum | reject | `050` |
| `CanonicalEntity` | entity type, resolver decision, policy version | required | canonical string and refs | reject | `070` |
| `SourceAsset` | source scope, source-native identity, validity interval | mixed | scope-key canonical JSON | reject if scope absent | `070` |
| `Identifier` | identifier type, normalized value, source scope, validity interval | required | canonical string and interval | reject | `070` |
| `GoldFact` | subject, predicate, object/value, valid interval, assertion state, authority profile, evidence set | required | canonical ordered tuple | reject | `080` |
| `EvidenceRef` | artifact class, artifact ID, role, checksum | required | canonical ordered tuple | reject | `040` |
| `GraphNodeDeltaShape` and `GraphEdgeDeltaShape` | owner-provided graph delta inputs | required | shape only; runtime ID owner imports policy | reject | `090` |

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

## Definition of Done

| ID | Criterion |
| --- | --- |
| `040-AC-001` | Two implementations serialize, hash, compare, and validate every core record shape identically. |
| `040-AC-002` | Omitted, null, empty, redacted, permission-limited, unknown, and unsupported values are distinguishable in serialized output. |
| `040-AC-003` | Graph delta primitive records cannot be created from backend internal IDs. |
| `040-AC-004` | Every `GoldFact` carries valid-time, known-time, assertion-state, evidence, confidence, and authority references. |
| `040-AC-005` | Unknown fields are rejected unless declared through a namespaced extension field rule owned by `050`. |


## Open Questions

Open questions marked `TODO:` block authoritative status for the affected contract. A downstream implementation must not resolve a `TODO:` by inference.
